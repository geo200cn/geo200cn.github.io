---
title: "Regression Trees"
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




This lab guide goes through the use of regression trees to make spatial predictions. It follows closely the material presented in ISLR Ch. 8. The objectives of this lab guide are as follows..

1. Learn how to run classification trees
2. Learn how to prune classification trees
3. Learn how to run regression and classification random forest models
4. Learn how to make predictions using a random forest model

To achieve these objectives, you will be applying regression tree methods in [species distribution modelling](https://rspatial.org/sdm) or ecological niche modeling. The objective in species distribution modelling is to predict the entire range of a species based on a set of locations where it has been observed. In this lab guide, we use the hominid species Imaginus magnapedum (also known under the vernacular names of “bigfoot” and “sasquatch”). This species is so hard to find (at least by scientists) that its very existence is commonly denied by the mainstream media! For more information about this controversy, see the article by Lozier, Aniello and Hickerson: [Predicting the distribution of Sasquatch in western North America: anything goes with ecological niche modelling](http://onlinelibrary.wiley.com/doi/10.1111/j.1365-2699.2009.02152.x/abstract). Note that much of this lab guide has been adapted from rspatial.org.


***

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.



Website created and maintained by [Noli Brazil](https://nbrazil.faculty.ucdavis.edu/)
