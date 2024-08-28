---
bibliography: book.bib
link-citations: true
biblio-style: apalike
csl: chicago-fullnote-bibliography.csl
editor_options: 
  markdown: 
    wrap: sentence
urlcolor: blue
---



# Landslides susceptibility mapping using Random Forest

## Introduction

In this application, we explore the capabilities of a stochastic approach based on a machine learning (ML) algorithm to elaborate landslides susceptibility mapping in Canton Vaud, Switzerland.
Generally speaking, ML includes a class of algorithms for the analysis, modelling, and visualization of environmental data and it performs particularly well to model environmental hazards, which naturally have a complex and non-linear behavior.
Here we use Random Forest, an ensemble ML algorithm based on decision trees.

### The main objective

Landslides are one of the major hazard occurring around the world.
In Switzerland, landslides cause damages to infrastructures and sometimes threaten human lives.
Shallow landslides are triggered by intense rainfalls.
Such slope movements are generally very rapid and hardly predictable.
Different quantitative approaches have been developed to assess the most susceptible areas.

This project applies a data-driven methodology based on Random Forest (RF) (@breiman_random_2001) to elaborate the landslides susceptibility map of canton of Vaud, in Switzerland.
RF is applied to a set of independent variables (i.e., the predictors) and dependent variables (the inventoried landslides and an equal number of locations for absences).
The overall methodology is described in the following graphic (\autoref{Methodology}).

### The overall methodology

The methodological overview is described in the following graphic: \autoref{Methodology}.

<div class="figure" style="text-align: center">
<img src="images/Methodology.png" alt="Basic elements of the generic methodology \label{Methodology}" width="80%" height="80%" />
<p class="caption">(\#fig:gen-met)Basic elements of the generic methodology \label{Methodology}</p>
</div>

### Load libraries

To perform the analysis, you have first to install the following libraries:

-   **library(terra)**: Methods for spatial data analysis with vector (points, lines, polygons) and raster (grid) data.

-   **library(sp)**: Classes and methods for spatial data.

-   **library(readr)**: The goal of 'readr' is to provide a fast and friendly way to read rectangular data (like 'csv', 'tsv', and 'fwf').

-   **library(randomForest)**: Classification and regression based on a forest of trees using random inputs, based on Breiman (2001) <doi:10.1023/A:1010933404324>.

-   **library(dplyr)**: It is the next iteration of plyr, focused on tools for working with data frames (hence the d in the name).

-   **library(pROC)**: Allowing to compute, analyze ROC curves, and

    -   **library(plotROC)** to display ROC curve

-   **(ggplot2)**: Is a system for declaratively creating graphics.

**List of the loaded libraries**


```
##  [1] "plotROC"      "ggplot2"      "pROC"         "dplyr"        "randomForest"
##  [6] "readr"        "terra"        "distill"      "stats"        "graphics"    
## [11] "grDevices"    "utils"        "datasets"     "methods"      "base"
```

## Import and process geodata

Import *landslides punctual dataset* presences and absences (LS_pa) and *predictors* (in raster format).
This help the exploratory data analyses step and to understand the input data structure.

### Landslides dataset

The **landslide inventory** has been provided by the environmental office of the canton of Vaud.
Only *shallow landslides* are used for susceptibility modelling.
One pixel per landslide-area (namely the one located at the highest elevation) has been extracted.
Since the landslide scarp is located in the upper part of the polygon, it makes sense to consider the highest pixel to characterize each single event.

Our model includes the implementation of the landslide **pseudo-absences**, which are the areas where the hazardous events did not took place (i.e. landslide location is known and the mapped footprint areas are available, but the non-landslide areas have to be defined).
Indeed, to assure a good generalization of the model and to avoid the overestimation of the absence, pseudo-absences need to be generated in all the cases where they are not explicitly expressed.
In this case study, an equal number of point as for presences has been randomly generated in the study area, except within landslides polygons, lakes and glaciers (that is what is called "validity domain", where events could potentially occur).

**The landslides dataset**


``` r
# Import the boundary of Canton Vaud 
Vaud <- vect("data/RF/Vaud_CH.shp")
Lake <- vect("data/RF/Lakes_VD.shp")


# Import the landslides dataset (dependent variable)
LS_pa <- read.csv("data/RF/LS_pa.csv") 

# Convert the numeric values (0/1) as factor 
##(i.e. categorical value)
LS_pa$LS<-as.factor(LS_pa$LS)

LS_vect<-vect(LS_pa, geom=c("X", "Y"),crs=crs(Vaud))

# Display the structure (str) and result summaries (summary)
str(LS_vect)
summary(LS_vect)

# Plot the events
plot(Vaud)
plot(Lake, col="lightblue", add=TRUE)
plot(LS_vect, col=LS_pa$LS, pch=20, cex=0.5, add=TRUE)
```

<img src="05-RF_Cl_files/figure-html/import-data-1.png" width="672" style="display: block; margin: auto;" />

### Predictor variables

Selecting predictive variables is a key stage of landslide susceptibility modelling when using a data-driven approach.
There is no consensus about the number of variables and which variables should be used.
In the present exercice e will use the following:

-   **DEM (digital elevation model)**: provided by the Swiss Federal Offce of Topography.
    The elevation is not a direct conditioning factor for landslide; however, it can reflect differences in vegetation characteristics and soil

-   **Slope**: is one of the most explicating factor in landslide susceptibility modelling.

$$Slope = arctan(\sqrt{(dz/dx)^2 + (dz/dy)^2)} * (\pi/2)$$

-   **Curvature**: curvature is widely used in landslide susceptibility modelling.
    It allows assessing the water flow acceleration and sediment transport process (*profile curvature*) and the water flow propensity to converge and diverge (*plan curvature*).
    They were derived from DEM using the curvature tool in ArcGIS.

-   **TWI (topographical water index)**: topography plays a key role in the spatial distribution of soil hydrological conditions.
    Defining $\alpha$ as the upslope contributing area describing the propensity of a cell to receive water, and $\beta$ as the slope angle, TWI (compute by the formula below), reflects the propensity of a cell to evacuate water:

$$TWI=ln(\alpha/tan(\beta))$$

-   **Distance to roads**: roads build in mountainous areas often cut the slope, weakening the cohesion of the soil.
    Moreover, roads surfaces are highly impermeable.
    This raster has been elaborated using the euclidean distance tool in ArcGIS, from the swissTLMRegio map where roads are represented by lines.

-   **Land Cover**: developed by the Swiss administration and based on aerial photographs and control points.
    It includes 27 categories distributed in the following 6 domains: human modified surfaces, herbaceous vegetation, shrubs vegetation, tree vegetation, surfaces without vegetation, water surfaces (glaciers included).

-   **Geology**: The use of the lithology increase the performance of the susceptibility landslide models.
    We use here the map elaborated by the Canton Vaud, defining the geotypes and reclassified in 10 classes in order to differentiate sedimentary rocks.

Than the predictor variables have to be aggregated into single object, storing multiple raster.
We use here the generic function **c** to combine the single raster into multiple raster objet.


``` r
## Import raster (independent variables) 25 meter resolution

landCover<-as.factor(rast("data/RF/landCover.tif"))
geology<-as.factor(rast("data/RF/Geological_classes.tif"))

planCurv<-rast("data/RF/plan_curvature.tif")/100
profCurv<-rast("data/RF/profil_curvature.tif")/100

# this because ArcGIS multiply curvature values by 100

TWI <- rast("data/RF/TWI.tif")
Slope <- rast("data/RF/Slope.tif")
dem <- rast("data/RF/DEM.tif")
dist <- rast("data/RF/dist_roads.tif")

# Combine raster

features<-c(dist, dem, landCover, TWI, planCurv, profCurv, Slope, geology)

# renames features as in LS
names(features)<-c("distRoad", "DEM", "landCover", "TWI", "planCurv", "profCurv", "slope", "geology")

# mask to DEM extension
features <- terra::mask(features, dem)

plot(features)
```

<img src="05-RF_Cl_files/figure-html/import-raster-1.png" width="672" style="display: block; margin: auto;" />

### The use of categorical variables in Machine Learning

The majority of ML algorithms (e.g., support vector machines, artificial neural network, deep learning) makes predictions on the base of the proximity between the values of the predictors, computed in terms of euclidean distance.
This means that these algorithms can not handle directly categorical values (i.e., qualitative descriptors).
Thus, in most of the cases, categorical variables need to be transformed into a numerical format.
One of the advantage of using Random Forest (as implemented in R) is that it can handle directly categorical variables, since the algorithm operate by constructing a multitude of decision trees at training time and the best split is chosen just by counting the proportion of each class observation.

To understand the characteristics of the categorical variables, you can plot the tow raster **Land Cover** and **Geology** by using their original classes and look at the attribute table to analyse the corresponding definitions.


``` r
plot(geology)
```

<img src="05-RF_Cl_files/figure-html/geology-map-1.png" width="672" style="display: block; margin: auto;" />

``` r
plot(landCover)
```

<img src="05-RF_Cl_files/figure-html/geology-map-2.png" width="672" style="display: block; margin: auto;" />

<div class="figure" style="text-align: center">
<img src="data/RF/Cat_classes.png" alt="Categorical variables \label{Cat_class}" width="80%" height="80%" />
<p class="caption">(\#fig:cat_class)Categorical variables \label{Cat_class}</p>
</div>

\newpage

## Extract velues

In this step, you will extract the values of the predictors at each location in the landslides (presences and absences) dataset.
The final output represents the input dataset with dependent (LS = landslides) and independent (raster features) variables.

**The final input dataset**


``` r
# Extract values from the raster dataset (features)
LS_input <-extract(features, LS_vect, method="simple",  xy=TRUE)

LS_input$LS <- as.factor(LS_vect$LS) # add LS
str(LS_input) # explore the dataset

# remove extra column (ID)
LS_input <- LS_input[,2:ncol(LS_input)]
LS_input<-na.omit(LS_input)

# Explore the newly created input dataset
head(LS_input)
str(LS_input)
```

### Split the input dataset into training (80%) and testing (20%)

A well-established procedure in ML is to split the input dataset into training, validation, and testing.

-   The **training dataset** is needed to calibrate the parameters of the model, which will be used to get predictions on new data.

-   The purpose of the **validation dataset** is to optimize the hyperparameter of the model (*training phase*).
    **NB**: in Random Forest this subset is represented by the Out-Of-Bag (**OOB**)!

-   To provide an unbiased evaluation of the final model and to assess its performance, results are then predicted over unused observations (*prediction phase*), defined as the **testing dataset**.


``` r
# Shuffle the rows
set.seed(123) # to ensure reproducibility 
LS_input_sh<-LS_input [sample(nrow(LS_input), nrow(LS_input)), ] 

# Split the input dataset into training (80%) and testing (20%)
n <- nrow (LS_input_sh)
set.seed(123)
n_train <- round(0.80 * n) 
train_indices <- sample(1:n, n_train)

# Create indices
LS_train <- LS_input_sh[train_indices, ]  
LS_test <- LS_input_sh[-train_indices, ]

# Count the number of elements in the two subset: training and testing
count(LS_train)
count(LS_test)
```

## Run Random Forest

Computationally, a subset of the training dataset is generated by bootstrapping (i.e. random sampling with replacement).
For each subset a decision tree is grown and, at each split, the algorithm randomly selects a number of variables (*mtry*) and it computes the Gini index to identify the best one.
The process stops when each node contains less than a fixed number of data points.
The fundamental hyperparameters that needs to be defined in RF are *mtry* and the total number of trees (*ntrees*).

The prediction error on the training dataset is finally assessed by evaluating predictions on those observations that were not used in the subset, defined as "out-of-bag" (**OOB**).
This values is used the optimize the values of the hyperparameters, by a trial and error process (that is, trying to minimize the OOB estimate of error rate).


``` r
# Set the seed of R‘s random number generator, 
## this is useful for creating simulations that can be reproduced.
set.seed(123) 

# Run RF model
RF_LS<-randomForest(y=LS_train$LS, x=LS_train[1:8],data=LS_train, ntree=500, mtry=3,importance=TRUE)
```

### RF main outputs

Printing the results of RF allows you to gain insight into the outputs of the implemented model, namely the following: - a summary of the model hyperparameters - the OOB estimate of error rate - the confusion matrix; in this case a 2x2 matrix used for evaluating the performance of the classification model (1-presence vs 0-absence).

The plotting the of the error rate is useful to estimate the decreasing values on the OOB and on the predictions (presence (1) / absence (0)) over increasing number of trees.


``` r
# Print the model setting
print(RF_LS) 
```

```
## 
## Call:
##  randomForest(x = LS_train[1:8], y = LS_train$LS, ntree = 500,      mtry = 3, importance = TRUE, data = LS_train) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 3
## 
##         OOB estimate of  error rate: 16%
## Confusion matrix:
##      0    1 class.error
## 0 1691  397   0.1901341
## 1  267 1795   0.1294859
```

``` r
# Show the predicted probability values
RF.predict <- predict(RF_LS,type="prob")
head(RF.predict) # 0 = absence ; 1 = presence
```

```
##               0          1
## 2063 0.28795812 0.71204188
## 1750 0.02259887 0.97740113
## 688  0.76506024 0.23493976
## 3301 1.00000000 0.00000000
## 4401 0.98514851 0.01485149
## 1166 0.05113636 0.94886364
```

``` r
# Plot the OOB error rate
plot(RF_LS)  
legend(x="topright", legend=c("perd 0", "pred 1", "OOB error"), 
 col=c("red", "green", "black"), lty=1:2, cex=0.8)
```

<img src="05-RF_Cl_files/figure-html/RF-outputs-1.png" width="672" style="display: block; margin: auto;" />

### Model evaluation

The prediction capability of the implemented RF model can be evaluated by predicting the results over previously unseen data, that is the testing dataset.
The *Area Under the "Receiver Operating Characteristic (ROC)" Curve* (**AUC**) represents the evaluation score used here as indicator of the goodness of the model in classifying areas more susceptible to landslides.
ROC curve is a graphical technique based on the plot of the percentage of correct classification (the true positives rate) against the false positives rate (occurring when an outcome is incorrectly predicted as belonging to the class "1" when it actually belongs to the class "0"), evaluates for many thresholds.
The AUC value lies between 0.5, denoting a bad classifier, and 1, denoting an excellent classifier, which, on the other hand, can in this case overfit.


``` r
# Make predictions on the testing dataset
RFpred_test <- predict(object = RF_LS, newdata = LS_test, type="prob")

# Make predictions on the validation dataset (taining using the Out-of-bag)
RFpred_oob <- predict(object = RF_LS, newdata = LS_train, type="prob", OOB=TRUE)

roc_test <- roc(LS_test$LS, RFpred_test[,2])
roc_oob <- roc(LS_train$LS, RFpred_oob[,2])

plot.new()
plot(1-roc_test$specificities, roc_test$sensitivities, type = 'l', col = 'blue',  xlab = "False positive rate", ylab = "True positive rate")
lines(1-roc_oob$specificities, roc_oob$sensitivities, type = 'l', col = 'red')
```

<img src="05-RF_Cl_files/figure-html/RF-val-1.png" width="672" style="display: block; margin: auto;" />

``` r
# Print AUC values
roc_test
roc_oob
```

## Susceptibility map

You have now all the elements to elaborate the final landslide susceptibility map.
This will be achieved by making predictions (of presence only) based on the values of the predictors, which are stored into multiple raster named \*features\*, created above.


``` r
# Convert the input multiple raster to data frame 
features_df<-as.data.frame(features, xy=TRUE, na.rm=TRUE) 

# Predict results of RF (probability of fires presence: index = 2) to the feature space 
## this operation can take several minuts to run!
scp_map<-predict(object = RF_LS, newdata = features_df, type="prob", index=2) 

scp_df=as.data.frame(scp_map) # convert to data frame
# get coordinates 
scp_df$X=features_df$x
scp_df$Y=features_df$y

# Convert to raster the probability to get a landslide
## 3=X, 4=Y, 2=probability of presence (1)
scp_rast=rast(scp_df[,c(3,4,2)],type="xyz")
summary(scp_rast)
plot(scp_rast)
```

<img src="05-RF_Cl_files/figure-html/Scp-map-1.png" width="672" style="display: block; margin: auto;" />

``` r
# Save all outputs
## this operation can take several minuts to run!
save.image(file="LSM_RF.RData")

# Export susceptibility map as raster
writeRaster(scp_rast,"Susceptibility_LSmap.tif",overwrite=T) 
```

### Class intervals for decision maker

What can you say by looking at the map?
Actually a risk heat map is a data visualization tool for communicating the level for a specific risk to occur.
These maps helps authorities to identify and prioritize the risks associated with a given hazard.

Normally an authority (i.e., a decision maker) prioritize its efforts based on the available resources it has.
So, it can be more useful to detect the areas with the highest probability of burning based given intervals (i.e., breaks).
The authority can thus concentrate its resources for preventive actions on a given percentage (such as 5%, 10%, or 20%) of the area with the highest probability of burning, instead of concentrate on the areas with a "stochastic" output probability value of 0.8 (for example).

-   Susceptibility maps based on equal intervals, five classes (each 20%)


``` r
library("RColorBrewer")
plot(scp_rast, xlab = "East [m]", ylab = "North [m]", main = "Landslides susceptibility map", col = rev(c('#a50026','#d73027','#f46d43','#fdae61','#fee08b','#d9ef8b')))
```

<img src="05-RF_Cl_files/figure-html/equal-interval-map-1.png" width="672" style="display: block; margin: auto;" />

-   Breaks chosen based on the **summary statics**: these values corresponds to 0-25%-50%-75%-100% of the p-value distribution.


``` r
brk<-c(0, 0.03, 0.14, 0.43, 1) 

plot(scp_rast, xlab = "East [m]", ylab = "North [m]", 
     main = "Landslides susceptibility map", 
     col = rev(c("brown", "orange", "yellow", "grey")), breaks=brk) 
```

<img src="05-RF_Cl_files/figure-html/quantile-map-1.png" width="672" style="display: block; margin: auto;" />

-   Breaks chosen based on well-established **percentile classes**


``` r
# Output predicted values are transformed to a vector
pred.vect <- as.vector(scp_map[, 2])

# The function "quantile" is used to fix classes
qtl.pred <- quantile(pred.vect,probs=c(0.25,0.5,0.75,0.85,0.95), na.rm = TRUE)
qtl.pred
```

```
##   25%   50%   75%   85%   95% 
## 0.026 0.138 0.424 0.606 0.824
```

``` r
# and then extract the corresponding values
qtl.int<- c(0,0.03,0.14,0.42,0.6,0.82,1)
plot(scp_rast, xlab = "East [m]", ylab = "North [m]", 
     main = "Landslides susceptibility map", 
     col = rev(c("brown", "red", "orange","yellow", "green", "grey")), breaks=qtl.int)
```

<img src="05-RF_Cl_files/figure-html/percentile-map-1.png" width="672" style="display: block; margin: auto;" />

## Conclusions and further analyses

This exercise allowed you to familiarize with Random Forest, by the proposed application about landslides susceptibility mapping and variables importance assessment.

To be sure that everything is perfectly clear for you, we propose you to **answer the following questions** and to discuss your answers with the other participants to the course or directly with the teacher.

1)  Why is it important to implement the pseudo-absence, other that the presences (i.e., the observations) in a data-driven modelization?

2)  What is the difference between a numerical and a categorical variable?
    Give some examples of both types.
    Why RF can handle directly native categorical variables?
    Is it the same for other algorithms (like logistic regression or neural network)?

3)  Which is the values of the OOB estimate error rate of your model?
    Which parameters you can change to try to reduce it?
    Be brave and do it (i.e., change the values for *ntree* and *mtry,* than analyse which values for the AUC you obtain and which model perform better.

    \newpage

## References

The research framework that inspired this computational lab refers to a pioneering study in susceptibility mapping for wildfire events by [@tonini_machine_2020] and further developed for the assessment of variable importance by [@trucchia_machine-learning_2022].