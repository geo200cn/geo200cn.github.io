---
title: "Model Resampling and Selection"
subtitle: <h4 style="font-style:normal">GEO 200CN - Quantitative Geography</h4>
author: <h4 style="font-style:normal">Professor Noli Brazil</h4>
output: 
  html_document:
    toc: true
    toc_depth: 3
    toc_float: true
    theme: yeti
    code_folding: show
---


<style>
p.comment {
background-color: #DBDBDB;
padding: 10px;
border: 1px solid black;
margin-left: 25px;
border-radius: 5px;
font-style: italic;
}

.figure {
   margin-top: 20px;
   margin-bottom: 20px;
}

h1.title {
  font-weight: bold;
  font-family: Arial;  
}

h2.title {
  font-family: Arial;  
}

</style>


<style type="text/css">
#TOC {
  font-size: 13px;
  font-family: Arial;
}
</style>


\




In the last lab guide, we learned how to partition our data into training and testing sets to estimate the predictive quality of a regression model.  In this lab, we extend this knowledge by going through the resampling techniques described in ISRL Ch. 5. We further expand our statistical toolkit by exploring methods that will allow us to choose the best set of predictors based on predictive quality.  These methods are discussed in ISLR Ch 6.  Although discussed in separate chapters, these methods are [intimately tied](https://www.youtube.com/watch?v=FA5jsa1lR9c). You run a Ch. 6 method to give a set of possible models and then you run a Ch. 5 method to give you a metric for determining which model gives the best predictive fit.

The objectives of this lab are as follows

1. Learn how to run cross validation to calculate test error rates
2. Learn how to run forward and backward stepwise selection
3. Learn how to run ridge and lasso regression
4. Learn how to select from models in (2) and (3) using the methods learned in (1)

To help us accomplish objective 1, we will use temperature data for California weather stations.  To help us accomplish objectives 2-4, we will use a data set from the previous lab - 2017 [Behavioral Risk Factor Surveillance System](https://www.cdc.gov/brfss/index.html) (BRFSS).  Download the data for this lab guide from Canvas in the Labs and Assignments Week 8 folder.  All the data are zipped into the file *resampling.zip*.




***

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.


Website created and maintained by [Noli Brazil](https://nbrazil.faculty.ucdavis.edu/)
