---
title: "Advanced Geospatial Data Analysis in R - Environmental applications"
author: "Dr. Marj Tonini, Haokun Liu"
date: "05/09/2024"
output:
  pdf_document: default
  html_document: default
  toc: true 
editor_options: 
  chunk_output_type: inline
  markdown: 
    wrap: sentence
urlcolor: blue
bibliography: book.bib
csl: apa.csl
---



# Preface {.unnumbered}

Earth surface environmental processes exhibit distinctive characteristics, encompassing both spatial and temporal dimensions, along with various attributes and predictive variables.
Coupled with uncertainty and complexity issues, all this contribute to make this field of research highly challenging.
Furthermore, in the era of Data Science, the wealth of available data and the rapid development of analytical models have emerged as distinctive aspects in the realm of **Advanced Geospatial Data Analysis (AGDA)**.
The domain of AGDA encompasses data exploration, manipulation, and modelling, from the acquisition phase up to the visualization and interpretation of the results.
Mapping where events are located and how they relate to each other provides a better understanding of the process being studied.
Finally, as an increasing volume of geo-environmental data becomes more and more accessible, the demand for spatial data scientists is growing rapidly, both in public research institutions as well as in private companies.

The term "**Data Science**" was defined for the first time by Peter Naur in 1974 as "*the science of dealing with data, once they have been established, while the relation of the data to what they represent is delegated to other fields and sciences*" [@naur_peter_concise_1974].
In essence, this definition frames Data Science as the technical and methodological handling of data, while the initial creation or gathering of data, as well as the interpretation of its meaning in the broader context, is not the primary focus of this discipline.
In 1977, the International Association for Statistical Computing was estabilished and further refined the concept of data science, emphasizing its role in the process of transforming raw data into valuable information and actionable knowledge ([IASC](https://iasc-isi.org/)).
The IASC articulated that data science involves not only the collection and processing of data but also the application of statistical methods and computational techniques to extract meaningful insights, which can then be used to support decision-making and drive innovation across various domains​.

In disciplines like Environmental and Earth Sciences, Physical Geography, Humanities and Social Sciences, the use of Data Science procedures is emerging only recently, proving to be extremely efficient to deal with the complexity of the investigated process and the heterogeneity of the underlying data sources [@amato_spatiotemporal_2022].
This leads to a cultural shift, moving scientists away from individual working within their own research domain.
Indeed, disciplinary boundaries are more and more permeable, pushing scientists to be more open to collaborate among them and with decision makers on the investigation and understanding of real-world problems.
Modern earth and environmental scientists need to interact with other disciplines, apparently far from their domain.
This openness is increasingly important as society struggles to respond to the implication of anthropogenic pressures on different issues, such as natural hazards and climate change, or the harmful impacts of human activities on biodiversity, water and air quality, human health.

## Book overview {.unnumbered}

The primary focus of this eBook is the application of Data Science methodologies to analyze and understand Earth's surface environmental processes.
The scientific approaches in this emerging field (spanning statistics, mathematics, geomatics, and computer science) are often challenging to master.
Thus, while maintaining a rigorous emphasis on mathematical and statistical formalism, this eBook prioritizes practical applications in the domain of Geosciences.

Emphasis will be placed on the application of advanced geospatial tools to natural and anthropogenic hazards, and to land use processes.
Theoretical concepts will be supported by case studies achieved using spatial clustering techniques and supervised and unsupervised machine learning algorithms (e.g., stochastic point process models for spatio-temporal cluster analysis; application of machine learning based approaches for geodemographic segmentation, susceptibility assessment and risk mapping).

### Computing lab {.unnumbered}

Computations are conducted using the R free software environment, which is specifically designed for data analysis and manipulation.
To complete the applied computing labs presented in each chapter, readers can download the datasets from here: [AGDA Book Gihub].

The computing labs will cover the following geocomputational techniques:

-   **Exploratory Data Analysis and Visualization**: Examining geographical variations in statistical data distributions, including Geographically Weighted Summary Statistics.
-   **Cluster Detection and Mapping**: Techniques such as Ripley's K-function, kernel density estimation, and DBSCAN for identifying and visualizing clusters.
-   **Self-Organizing Maps**: As example of unsupervised machine learning method for data clustering/segmentation and visualization.
-   **Random Forest**: As example of supervised machine learning algorithm. We present both the global and the local version, and we use Random Forest to introduce interpretability & explainability in machine learning.

### Target audience {.unnumbered}

The target audience includes master's and PhD students in Earth and Environmental Sciences, Biology and Spatial Ecology, Physical Geography, and related disciplines.
Our goal is to empower students by guiding them through both theoretical knowledge and hands-on practical applications, helping them develop strong problem-solving skills.

This eBook aims to equip the audience with:

-   A solid understanding of key practical concepts and applied aspects in AGDA.
-   Advanced tools to effectively navigate and analyze spatial datasets in geosciences.

This book is designed for intermediate to advanced R users with experience in geospatial data and a keen interest in geocomputing.
If you have only a basic knowledge of these areas, we strongly encourage you to explore the essential references provided in each chapter.
These resources offer valuable documentation and additional materials to help deepen your understanding.

The main prerequisites are the following.

-   Knowledge of basic statistics: methods of descriptive statistics (measures of central tendency and dispersion); how to assess relationships between variables; concepts of correlation and regression.
-   Basic knowledge in geomatics (GIS): basic operations with raster and vector datasets.
-   R programming basics and RStudio.

## Authors information {.unnumbered}

**Marj Tonini** is a spatial data scientist with a profound expertise in geospatial modeling for risk assessment, particularly in relation to wildfires and landslides.
She earned her Ph.D. in 2002 from the Sant'Anna School of Advanced Studies in Pisa, Italy, where she developed an agro-environmental modeling thesis that laid the foundation for her career.
In 2004, she joined the University of Lausanne as a postdoctoral researcher in geospatial data analysis, and by 2008, she was appointed as Senior Research Manager at the Institute of Earth Surface Dynamics, where she continues to serve in her current role.
Marj's research is centered on the development of innovative methodologies that facilitate the efficient extraction of knowledge from complex environmental datasets.
Her work is driven by the goal of creating a robust methodological framework to understand the spatio-temporal dynamics of environmental processes and to evaluate the influence of various predictor variables.
Her current research focuses on the analysis of land use and land cover changes, alongside the development of predictive scenarios and the assessment of susceptibility and risks associated with natural hazards.
Through her work, she seeks to advance the field of geospatial science by translating data-driven insights into practical solutions for managing and mitigating environmental risks.

Marj has written the vast majority of this book.
Her contributions include the development of the conceptual framework, writing the theoretical background chapters, developing case studies, designing methodologies, and conducting data collection and analysis, software/tool development.
the Sections "My Journey of Learning" refers to her personal experience.

**Haokun Liu** is a Ph.D. student at the Group of Cities and Dynamics of Networks (CITADYNE Group) at the University of Lausanne.
In addition to his doctoral studies, he serves as a student assistant at the Swiss Geocomputing Center (SGC).
Benefiting from rigorous and comprehensive training in both China and Switzerland, Haokun has developed a diverse research portfolio.
His primary research interests and experience encompass a wide range of interdisciplinary fields, including urban analytics, health geography, and spatial data science.
His work focuses on leveraging advanced computational techniques to address complex urban and environmental challenges, bridging the gap between data-driven insights and real-world applications.

Haokun's contributions were instrumental in establishing and managing the GitHub repository used to host and maintain this eBook.
He also played a key role in data visualization and mapping, and provided valuable revisions to the applied computing labs.

## My Journey of Learning: Integrating Geospatial Data Analysis with Data Science {.unnumbered}

In the early 2000s, as I (Marj) embarked on my PhD journey, a new frontier was beginning to unfold in the world of environmental research.
Geographical information systems (GIS) were just starting to make waves, offering a fresh perspective on how we could investigate and understand the environment.
It was through my thesis on the effects of spreading olive vegetation water on agricultural land that I first delved into the world of GIS.
Each tool and technique I learned opened new doors, revealing the vast potential of spatial data in environmental analysis and management.
This early exposure was more than just a technical skill; it was a revelation.
I began to see the world through the lens of spatial data, understanding how it could illuminate patterns and connections that were otherwise hidden.
This experience not only shaped my research but ignited a passion that would drive me to explore the depths of spatial analysis for years to come.

After completing my PhD, my journey led me to a PostDoc position where I had the extraordinary opportunity to work alongside Professor Mikhail Kanevski.
Mikhail was a true pioneer, a visionary in the fields of environmental data mining, geostatistics, and the burgeoning field of applying machine learning to spatial environmental data.
Being part of his research group was like stepping into a realm where the boundaries of what was possible were constantly being pushed and redefined.
In this research team, I found myself diving headfirst into the world of spatial statistics, driven by the desire to uncover valuable insights from the complex data that described our environment.
The work was challenging, yet it was also exhilarating, as every new technique I learned brought me closer to understanding the intricate tapestry of environmental phenomena.
This experience was pivotal, setting the course for my future research and solidifying my belief in the power of combining advanced techniques with a deep understanding of spatial data.

As the years passed, the tools and resources available to researchers like me grew exponentially.
The development and widespread adoption of free programming languages like Python and R revolutionized the way we approached statistical analysis and machine learning.
What was once the domain of a select few became accessible to many, thanks to the wealth of pre-developed scripts and resources these languages offered.
For me, and for countless others, this accessibility was a game-changer.
It meant that advanced techniques were no longer out of reach, and it fostered a culture of greater reproducibility in research.

Using standardized, widely adopted tools like Python and R, researchers could now more easily share their code and methodologies, allowing others to replicate and validate findings.
This collaborative spirit not only strengthened the reliability and credibility of scientific research but also created a community where knowledge could be shared and built upon more effectively.

Reflecting on this journey, I realize that each step---from my early days with GIS to my time in Professor Kanevski's group, to my current role as an independent lead researcher in environmental spatial data analysis in the era of data science---has been a chapter in a larger story of discovery and innovation.
It's a story of how spatial data and advanced analytics have become indispensable tools in our quest to understand and protect the environment, and it's a story that continues to unfold with each new challenge and breakthrough.

## Acknowledgements {.unnumbered}

The case studies presented in each chapter came from different projects carried out in collaboration with several colleagues, including master and PhD students.
All the produced scientific papers are duly cited in the bibliography.

I would like to express my gratitude to Professor Mário Gonzalez Pereira and Dr. Joana Parente, for their extensive and fruitful collaboration in investigating the spatio-temporal distribution of wildfires in Portugal.
One of our studies, which explores the evolution of forest fires from spatio-temporal point events to smoothed density maps, forms an integral part of Chapters 3 and 4.

I also thank Professor Stuart Lane and Dr. Natan Micheletti for introducing me to the fascinating world of rock glacier research.
Notably, the 3D point cloud dataset analyzed in Chapter 8 was acquired and processed by Natan during his PhD studies.

My thanks also go to Axelle Bersier for her meticulous work in acquiring and pre-processing the Swiss national population census dataset, which is used for the exercise on unsupervised learning in Chapter 5.
The same for Julien Riese, who produced the input dataset and collaborated with me in developing the code that assesses landslide susceptibility in Canton Vaud, the main focus of Chapter 6.
Both Julien and Axelle exemplify the high caliber of master's students I have had the pleasure of supervising.

For transparency, I acknowledge the use of ChatGPT in assisting with the reformulation of certain sentences in this book and DALL-E to generate the image icon.
