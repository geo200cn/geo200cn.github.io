---
title: "Introduction to Spatial Data"
subtitle: <h4 style="font-style:normal">GEO 200CN - Quantitative Geography</h4>
author: <h4 style="font-style:normal">Professor Noli Brazil</h4>
date: <h4 style="font-style:normal">April 18, 2022</h4>
output: 
  html_document:
    toc: true
    toc_depth: 4
    toc_float: true
    theme: cosmo
    code_folding: show
    self_contained: false
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




This week you will learn how to process and map spatial data in R. The three major packages for processing spatial data in R are **sp**, **sf** and **raster**. This guide will provide an introduction to these packages, with a heavier focus on the **sf** package. To motivate the use of these packages, the bulk of the guide will focus on how to map data, which will help you answer the mapping questions in this week's assignment.


<div style="margin-bottom:25px;">
</div>
## **Installing and loading packages**
\


Install **sp**, **sf**, **raster** if you have not already done so. 


```r
install.packages(c("sf", "raster", "sp"))
```

We will also be using the packages **tmap**, **RColorBrewer**, **rgdal**, and **cartogram** in this lab.


```r
install.packages(c("tmap", "RColorBrewer", "rgdal", "cartogram"))
```

Load all of these packages using `library()`.  We'll also be using the package **tidyverse**, so load it up as well.  Remember, you always need to use `library()` at the top of your R Markdown files.


```r
library(sf)
library(sp)
library(raster)
library(tmap)
library(tidyverse)
library(RColorBrewer)
library(cartogram)
```


<div style="margin-bottom:25px;">
</div>
## **Spatial Data**
\


Spatial phenomena can generally be thought of as either discrete objects with clear boundaries or as a continuous phenomenon that can be observed everywhere, but does not have natural boundaries. Discrete spatial objects may refer to a river, road, country, town, or a research site. Examples of continuous phenomena, or “spatial fields”, include elevation, temperature, and air quality.

Spatial objects are usually represented by vector data. Such data consists of a description of the “geometry” or “shape” of the objects, and normally also includes additional variables. For example, a vector data set may describe the borders of the countries of the world (geometry), and also store their names and the size of their population; or the geometry of the roads in an area, as well as their type and names. These additional variables are often referred to as “attributes”. Continuous spatial data (fields) are usually represented with a raster data structure. We discuss these two data types in turn.


<div style="margin-bottom:25px;">
</div>
## **Vector data**
\

The main vector data types are points, lines and polygons. In all cases, the geometry of these data structures consists of sets of coordinate pairs (x, y). Points are the simplest case. Each point has one coordinate pair, and *n* associated variables. For example, a point might represent a place where a rat was trapped, and the attributes could include the date it was captured, the person who captured it, the species size and sex, and information about the habitat. It is also possible to combine several points into a multi-point structure, with a single attribute record. For example, all the coffee shops in a town could be considered as a single geometry.

The geometry of lines is a just a little bit more complex. First note that in this context, the term ‘line’ refers to a set of one or more polylines (connected series of line segments). For example, in spatial analysis, a river and all its tributaries could be considered as a single ‘line’ (but they could also also be several lines, perhaps one for each tributary river). Lines are represented as ordered sets of coordinates (nodes). The actual line segments can be computed (and drawn on a map) by connecting the points. Thus, the representation of a line is very similar to that of a multi-point structure. The main difference is that the ordering of the points is important, because we need to know which points should be connected. A network (e.g. a road or river network), or spatial graph, is a special type of lines geometry where there is additional information about things like flow, connectivity, direction, and distance.

A polygon refers to a set of closed polylines. The geometry is very similar to that of lines, but to close a polygon the last coordinate pair coincides with the first pair. Multiple polygons can be considered as a single geometry. For example, the Philippines consists of many islands. Each island can be represented by a single polygon, but together they can be represented as a single (multi-) polygon representing the entire country.

We will be primarily focusing on vector data in the next several weeks.  We will return to raster data when we learn about spatial interpolation later in the quarter.


The traditional way of handling vector data in R is to use the **sp** package. **sp** has been around since 2005, and thus has a rich ecosystem of tools built on top of it. **sf** is newer (first released in 2016) so it doesn’t have such a rich ecosystem. However, it’s much easier to use and fits in very naturally with the **tidyverse**.    Which means we can use most of the functions we went through when we learned about the [tidyverse](https://geo200cn.github.io/eda) package to clean and process **sf** spatial objects. The **sp** package uses a rather complex data structure, which can make it challenging to use. 

The trend is gradually shifting towards the use of **sf** as the primary spatial package.  For this reason and because it adheres to tidy principles, I prefer **sf** over **sp**. But because it is relatively new, **sf** is not wholly compatible with all of R's spatial functions, particularly those that perform spatial data analysis, including raster analysis (although this is [changing](https://keen-swartz-3146c4.netlify.com/raster.html#raster)). In contrast, **sp** is compatible with most spatial functions. 

In this class, we will lean heavily on the **sf** package when dealing with vector spatial data, but use **sp** when convenient or needed.


<div style="margin-bottom:25px;">
</div>
### **sp package**
\


The **sp** package defines a set of classes to represent spatial data. A class defines a particular data type. The `data.frame` is an example of a class. Any particular `data.frame` you create is an object (instantiation) of that class.

The main reason for defining classes is to create a standard representation of a particular data type to make it easier to write functions (also known as ‘methods’) for them. In fact, the **sp** package does not provide many functions to modify or analyze spatial data; but the classes it defines are used in more than 100 other R packages that provide specific functionality. 

**sp** introduces a number of classes with names that start with *Spatial*. For vector data, the basic types are the *SpatialPoints*, *SpatialLines*, and *SpatialPolygons*. These classes only represent geometries. To also store variables, classes are available with these names plus *DataFrame*, for example, *SpatialPolygonsDataFrame* and *SpatialPointsDataFrame*. When referring to any object with a name that starts with *Spatial*, it is common to write Spatial\*. When referring to a *SpatialPolygons* or *SpatialPolygonsDataFrame* object it is common to write SpatialPolygons\*. Don't worry too much about these details.  If you are interested, the Spatial classes (and their use) are described in detail by 
[Bivand, Pebesma and Gómez-Rubio](https://link.springer.com/book/10.1007/978-1-4614-7618-4).


To demonstrate how vector data are generally depicted in **sp**, let's bring in the polygon shapefile *lux*, which is a shapefile of Luxembourg and is a part of the **raster** package.  The shapefile is the most commonly used file format for vector data. Use the following code to to get the full path name of the file’s location. We need to do this as the location of this file depends on where the **raster** package is installed. You should not use the `system.file()` function for your own files. In other words, don't dwell too much on what this function is doing. It only serves for creating examples with data that ships with R.


```r
filename <- system.file("external/lux.shp", package="raster")
filename
```

```
## [1] "/Library/Frameworks/R.framework/Versions/4.1/Resources/library/raster/external/lux.shp"
```


Now we have the filename we need to use for the `shapefile()` function. This function comes with the **raster** package. For it to work you must also have the **rgdal** package, which we loaded in earlier.


```r
lux <- shapefile(filename)
```

The `shapefile()` function returns Spatial\*DataFrame objects. In this case a *SpatialPolygonsDataFrame*. We can use the basic `plot()` function to make a map.


```r
plot(lux)
```

![](introspatial_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

We mapped the polygons above, but we can also map one of the attributes.  We can do this by using the function `spplot()`.  Here we show a choropleth map of the area of each region *AREA*.


```r
spplot(lux, 'AREA')
```

![](introspatial_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

We can manipulate the *SpatialPolygonsDatarFrame* like we would a regular data frame. Here we'll use base R to manipulate. To extract the attributes (*data.frame*) from a Spatial object, use:


```r
d <- as.data.frame(lux)
head(d)
```

```
##   ID_1       NAME_1 ID_2     NAME_2 AREA
## 0    1     Diekirch    1   Clervaux  312
## 1    1     Diekirch    2   Diekirch  218
## 2    1     Diekirch    3    Redange  259
## 3    1     Diekirch    4    Vianden   76
## 4    1     Diekirch    5      Wiltz  263
## 5    2 Grevenmacher    6 Echternach  188
```

You can extract a variable


```r
lux$NAME_2
```

```
##  [1] "Clervaux"         "Diekirch"         "Redange"          "Vianden"         
##  [5] "Wiltz"            "Echternach"       "Remich"           "Grevenmacher"    
##  [9] "Capellen"         "Esch-sur-Alzette" "Luxembourg"       "Mersch"
```

Create a new variable, for example a new variable containing *AREA* multiplied by 100


```r
lux$AREA_100 <- lux$AREA*100
```

Now, sub-setting by variable. 


```r
lux[, 'NAME_2']
```

```
## class       : SpatialPolygonsDataFrame 
## features    : 12 
## extent      : 5.74414, 6.528252, 49.44781, 50.18162  (xmin, xmax, ymin, ymax)
## crs         : +proj=longlat +datum=WGS84 +no_defs 
## variables   : 1
## names       :   NAME_2 
## min values  : Capellen 
## max values  :    Wiltz
```

Note how this code is different from the code above it. Above `lux$NAME_2` returns a vector of values is returned. With the approach directly above you get a new *SpatialPolygonsDataFrame* with only one variable.

Selecting rows (records).


```r
g <- lux[lux$NAME_1 == 'Grevenmacher',]
plot(g)
```

![](introspatial_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

There are other **sp** spatial data functions you can use to manipulate the polygons.  Since we won't be focusing on **sp** in this class, you can learn more about these functions on your own [here](https://rspatial.org/raster/spatial/7-vectmanip.html#).  


<div style="margin-bottom:25px;">
</div>
### **sf package**
\

The **sf** package is the other major package for handling vector data in R. It conceives of spatial objects as **s**imple **f**eatures. What is a feature? A feature is thought of as a thing, or an object in the real world, such as a building or a tree. A county can be a feature. As can a city and a neighborhood. Features have a geometry describing where on Earth the features are located, and they have attributes, which describe other properties.  The main application of simple feature geometries is to describe two-dimensional geometries by points, lines, or polygons. 


<div style="margin-bottom:25px;">
</div>
#### **Reading spatial data**
\

The function for reading in spatial data into R as an **sf** object is `st_read()`.  I zipped up and uploaded onto GitHub a folder containing files for this lab.  Set your working directory to an appropriate folder and use the following code to download and unzip the file.


```r
#insert the pathway to the folder you want your data stored into
setwd("insert your pathway here")
#downloads file into your working directory 
download.file(url = "https://raw.githubusercontent.com/geo200cn/data/master/spatialsflab.zip", destfile = "spatialsflab.zip")
#unzips the zipped file
unzip(zipfile = "spatialsflab.zip")
```



The data are also located on Canvas in the Labs and Assignments Week 4 folder. 

You should see the shapefiles *saccityinc.shp*, *Rivers.shp*, *Parks.shp*, and *EV_Chargers.shp* in the folder you specified in `setwd()`. These files contain Sacramento city [Census tract](https://www2.census.gov/geo/pdfs/education/CensusTracts.pdf) polygons, Sacramento river polylines, Sacramento [parks](https://data.cityofsacramento.org/datasets/b3047674f3f04a759c484fe5208faf6c_0/about) polygons, and [Sacramento Electric Vehicle Charging](https://data.cityofsacramento.org/datasets/93efdbcdbc744210848dd7f601e622e3_0/about) Locations points, respectively. You should also see a csv file *sacrace.csv*, which contains Sacramento city Census tract race/ethnic population counts. First, bring in the shapefiles using the function `st_read()`.


```r
saccitytracts <- st_read("saccityinc.shp", stringsAsFactors = FALSE)
parks <- st_read("Parks.shp", stringsAsFactors = FALSE)
rivers <- st_read("Rivers.shp", stringsAsFactors = FALSE)
evcharge <- st_read("EV_Chargers.shp", stringsAsFactors = FALSE)
```


The argument `stringsAsFactors = FALSE` tells R to keep any variables that look like a character as a character and not a factor.  Look at *saccitytracts* 


```r
saccitytracts
```

```
## Simple feature collection with 122 features and 2 fields
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -121.5601 ymin: 38.43789 xmax: -121.3627 ymax: 38.6856
## Geodetic CRS:  NAD83
## First 10 features:
##         GEOID medinc                       geometry
## 1  6067007104   High MULTIPOLYGON (((-121.5076 3...
## 2  6067000400   High MULTIPOLYGON (((-121.4748 3...
## 3  6067006900    Low MULTIPOLYGON (((-121.4729 3...
## 4  6067003102 Medium MULTIPOLYGON (((-121.4457 3...
## 5  6067006400    Low MULTIPOLYGON (((-121.4291 3...
## 6  6067001101    Low MULTIPOLYGON (((-121.4989 3...
## 7  6067009633 Medium MULTIPOLYGON (((-121.4476 3...
## 8  6067001900 Medium MULTIPOLYGON (((-121.4824 3...
## 9  6067009614   High MULTIPOLYGON (((-121.4447 3...
## 10 6067004906 Medium MULTIPOLYGON (((-121.4531 3...
```

The output gives us pertinent information regarding the spatial file, including the type (MULTIPOLYGON), the extent or bounding box, the [coordinate reference system](https://www.nceas.ucsb.edu/sites/default/files/2020-04/OverviewCoordinateReferenceSystems.pdf),
and the 4 variables.  Click on *saccitytracts* in your environment window.  You'll find that a window pops up in the top left portion of your interface.  Unlike with **sp** objects, an **sf** object looks and feels like a regular data frame.  The only difference is there is a *geometry* column.  This column "spatializes" the dataset, or lets R know where each feature is geographically located on the Earth's surface.

You can bring in other types of spatial data files other than shapefiles.  See a list of these file types [here](https://cran.r-project.org/web/packages/sf/vignettes/sf2.html).

Next bring in *sacrace.csv* which contains Hispanic (*hisp*), non-Hispanic Asian (*nhasn*), non-Hispanic Black (*nhblk*), non-Hispanic white (*nhwhite*) and total population (*tpopr*) for tracts in Sacramento city. Use the function `read_csv()`.


```r
sacrace <- read_csv("sacrace.csv")
```





<div style="margin-bottom:25px;">
</div>
#### **Data manipulation**
\

**sf** objects are data frames.  This means that they are tidy friendly.  You can use many of the functions we learned [previously](https://geo200cn.github.io/eda.html) to manipulate **sf** objects, and this includes our new best friend the pipe `%>%` operator.  For example, let's do the following tasks on *saccitytracts*.

1. Merge in the race/ethnicity and total population variables from *sacrace* into *saccitytracts* using `left_join()`. The common ID is *GEOID*
2. Create the variable *pwh* which is the percent of the tract population that is non-Hispanic white using `mutate()`
3. Keep just the variables *GEOID*, *medinc*, and *pwh* using the function `select()`

We do this in one line of continuous code using the pipe operator `%>%`


```r
saccitytracts <- saccitytracts %>%
            left_join(sacrace, by = "GEOID") %>%
            mutate(pwh = nhwhite/tpopr) %>%
            select(GEOID, medinc, pwh)
```


You can also keep certain polygons (i.e. subset rows).  For example, keep the tracts that are majority non-white


```r
mwhite <- saccitytracts %>%
          filter(pwh < 0.5)
#number of neighborhoods in Sacramento that are majority non white
nrow(mwhite)
```

```
## [1] 94
```


The main takeaway: **sf** objects are tidy friendly, so you can use many of the tidy functions on these objects to manipulate your data set.

While tidyverse offers a set of functions for data manipulation, the **sf** package offers a suite of functions unique to manipulating spatial data. Spatial data manipulation involves cleaning or altering your data set based on the geographic location of features.  Most of these functions start out with the prefix `st_`. To see all of the functions, type in


```r
methods(class = "sf")
```

We won’t go through these functions as the list is quite extensive. You can learn  about them in Lovelace et al's excellent [guide](https://geocompr.robinlovelace.net/spatial-operations.html).  The focus of this lab is to learn how to map **sf** objects, which we'll get to later in this guide.


<div style="margin-bottom:25px;">
</div>
## **Raster Data**
\

Raster data is commonly used to represent spatially continuous phenomena such as elevation. A raster divides the world into a grid of equally sized rectangles (referred to as cells or, in the context of satellite remote sensing, pixels) that all have one or more values (or missing values) for the variables of interest. A raster cell value should normally represent the average (or majority) value for the area it covers. However, in some cases the values are actually estimates for the center of the cell (in essence becoming a regular set of points with an attribute).

In contrast to vector data, in raster data the geometry is not explicitly stored as coordinates. It is implicitly set by knowing the spatial extent and the number of rows and columns in which the area is divided. From the extent and number of rows and columns, the size of the raster cells (spatial resolution) can be computed. While raster cells can be thought of as a set of regular polygons, it would be very inefficient to represent the data that way as coordinates for each cell would have to be stored explicitly. It would also dramatically increase processing speed in most cases.

The main package for handling raster data in R is **raster**.  The **raster** package has functions for creating, reading, manipulating, and writing raster data.  It is built around a number of classes of which the *RasterLayer*, *RasterBrick*, and *RasterStack* classes are the most important. When discussing methods that can operate on all three of these objects, they are referred to as Raster* objects.

The package provides, among other things, general raster data manipulation functions that can easily be used to develop more specific functions. For example, there are functions to read a chunk of raster values from a file or to convert cell numbers to coordinates and back. The package also implements raster algebra and many other functions for raster data manipulation.

A *RasterLayer* object represents single-layer (variable) raster data. A *RasterLayer* object always stores a number of fundamental parameters that describe it. These include the number of columns and rows, the spatial extent, and the Coordinate Reference System (CRS). In addition, a *RasterLayer* can store information about the file in which the raster cell values are stored (if there is such a file). A *RasterLayer* can also hold the raster cell values in memory.

Sometimes you will bring in a raster dataset directly from a file. To do that, you use the the `raster()` function. Here is an example using the ‘Meuse’ dataset (taken from the **raster** package), using a file in the native ‘raster-file’ format. Use the following code to bring in the data.


```r
filename <- system.file("external/test.grd", package="raster")
r1 <- raster(filename)
r1
```

```
## class      : RasterLayer 
## dimensions : 115, 80, 9200  (nrow, ncol, ncell)
## resolution : 40, 40  (x, y)
## extent     : 178400, 181600, 329400, 334000  (xmin, xmax, ymin, ymax)
## crs        : +proj=sterea +lat_0=52.1561605555556 +lon_0=5.38763888888889 +k=0.9999079 +x_0=155000 +y_0=463000 +datum=WGS84 +units=m +no_defs 
## source     : test.grd 
## names      : test 
## values     : 138.7071, 1736.058  (min, max)
```

We see that this data is a *RasterLayer*.  Remember that raster data are basically grid data, and the grid we have contains 115 rows, 80 cells, and 9200 (115 x 80) cells. Let's plot what we got (it rhymes!)


```r
plot(r1)
```

![](introspatial_files/figure-html/unnamed-chunk-22-1.png)<!-- -->


Rather than bringing in a raster dataset, you often have to convert a vector object, typically polygons or points, into a raster. Polygon to raster conversion is typically done to create a *RasterLayer* that can act as a mask, i.e. to set to NA a set of cells of a raster object, or to summarize values on a raster by zone. For example a country polygon is transferred to a raster that is then used to set all the cells outside that country to NA; whereas polygons representing administrative regions such as states can be transferred to a raster to summarize raster values by region.  

Let's turn the Luxembourg polygon *lux* we used earlier into a raster by using the function `raster()`. The size of each pixel defines the resolution or `res` of the new raster.


```r
r <- raster(lux, res=0.01 )
r
```

```
## class      : RasterLayer 
## dimensions : 73, 78, 5694  (nrow, ncol, ncell)
## resolution : 0.01, 0.01  (x, y)
## extent     : 5.74414, 6.52414, 49.45162, 50.18162  (xmin, xmax, ymin, ymax)
## crs        : +proj=longlat +datum=WGS84 +no_defs
```

If you try to plot the raster you get an error.


```r
plot(r)
```

```
## Error in .plotraster2(x, col = col, maxpixels = maxpixels, add = add, : no values associated with this RasterLayer
```

Object *r* only has the skeleton of a raster data set. That is, it knows about its location, resolution, etc., but there are no values associated with it. The function `rasterize()`transfers values associated with 'object' type spatial data (points, lines, polygons) to raster cells. Let's transfer the area *AREA* from `lux` to their associated raster cell.


```r
vr <- rasterize(lux, r, 'AREA')
plot(vr)
```

![](introspatial_files/figure-html/unnamed-chunk-25-1.png)<!-- -->

Notice the difference between how this map "looks" compared to the vector data mapped earlier. Why is there a difference?

Similar to vector data, you can manipulate raster data, such assign values to cells, add two rasters, and so on.  We won't go into too much of this until later in the class when we go through spatial interpolation, but you can learn more about these functions [here](https://rspatial.org/raster/spatial/8-rastermanip.html#).

It is quite common to analyze raster data using single-layer objects. However, in many cases multi-variable raster data sets are used. The **raster** package has two classes for multi-layer data the *RasterStack* and the *RasterBrick*. The principal difference between these two classes is that a *RasterBrick* can only be linked to a single (multi-layer) file. In contrast, a *RasterStack* can be formed from separate files and/or from a few layers (‘bands’) from a single file.  We won't be using *RasterStack* and *RasterBrick* too much in this class.  You can learn more about them on your own [here](https://rspatial.org/raster/spatial/4-rasterdata.html#rasterstack-and-rasterbrick).  

We will be revisiting raster data later in the class when we get into spatial prediction.

<div style="margin-bottom:25px;">
</div>
## **Mapping**
\

Now that you've learned the basics of bringing in and manipulating spatial data, let's go through how we can map it.  We already did some mapping above, but let's go through the set of map types discussed in OSU Ch. 3 and lecture. We'll focus entirely on mapping **sf** spatial objects. There are two ways of mapping **sf** objects: `ggplot()` from the **ggplot2** package and `tm_shape()` from the **tmap** package. 

<div style="margin-bottom:25px;">
</div>
### **ggplot**
\

Because **sf** is tidy friendly, it is no surprise we can use the tidyverse plotting function `ggplot()` to make maps. We already received an introduction to `ggplot()` a [few labs ago](https://geo200cn.github.io/eda.html).  Recall its basic structure:


<br>

````
ggplot(data = <DATA>) +
      <GEOM_FUNCTION>(mapping = aes(x, y)) +
      <OPTIONS>()
````
<br>

In mapping, `geom_sf()` is `<GEOM_FUNCTION>()`. Unlike with functions like `geom_histogram()` and `geom_boxplot()`, we don’t specify an x and y axis. Instead you use fill if you want to map a variable or color to just map boundaries.

Let’s use `ggplot()` to make a choropleth map. Choropleth maps are discussed on page 73-76 in OSU. We need to specify a numeric variable in the `fill =` argument within `geom_sf()`. Here we map percent non-Hispanic white *pwh*.


```r
ggplot(data = saccitytracts) +
  geom_sf(aes(fill = pwh))
```

![](introspatial_files/figure-html/unnamed-chunk-26-1.png)<!-- -->

We can also specify a title (as well as subtitles and captions) using the `labs()` function.


```r
ggplot(data = saccitytracts) +
  geom_sf(aes(fill = pwh)) +
    labs(title = "Percent white Sacramento City Tracts")  
```

![](introspatial_files/figure-html/unnamed-chunk-27-1.png)<!-- -->

We can make further layout adjustments to the map. Don't like a blue scale on the legend? You can change it using the `scale_file_gradient()` function. Let's change it to a white to red gradient.  We can also eliminate the gray tract border colors to make the fill color distinction clearer. We do this by specifying `color = NA` inside `geom_sf()`. We can also get rid of the gray background by specifying a basic black and white theme using `theme_bw()`.


```r
ggplot(data = saccitytracts) +
  geom_sf(aes(fill = pwh), color = NA) +
  scale_fill_gradient(low= "white", high = "red", na.value ="gray") +  
  labs(title = "Percent white Sacramento City Tracts",
       caption = "Source: American Community Survey") +  
  theme_bw()
```

![](introspatial_files/figure-html/unnamed-chunk-28-1.png)<!-- -->

I also added a caption indicating the source of the data using the `captions =` parameter  within `labs()`.

You also use `geom_sf()` for mapping points (an example of a pin map as described in OSU page 66).  Color them black using `fill = "black"`. Let's map the location of EV chargers in Sacramento.


```r
ggplot(data = evcharge) +
  geom_sf(fill = "black") +
  labs(title = "EV Charge Stations Sacramento City Tracts",
       caption = "Source: City of Sacramento") +  
  theme_bw()
```

![](introspatial_files/figure-html/unnamed-chunk-29-1.png)<!-- -->

You can overlay the points over Sacramento tracts to give the locations some perspective.  Here, you add two `geom_sf()` for the tracts and the EV charge stations.


```r
ggplot() +
  geom_sf(data = saccitytracts) +
  geom_sf(data = evcharge, fill = "black") +
  labs(title = "EV Charge Stations Sacramento City Tracts",
       caption = "Source: City of Sacramento") +  
  theme_bw()
```

![](introspatial_files/figure-html/unnamed-chunk-30-1.png)<!-- -->


Note that `data =` moves out of `ggplot()` and into `geom_sf()` because we are mapping more than one spatial feature. 

What about a map of polylines? Here is a map of rivers intersecting with Sacramento City.



```r
ggplot() +
  geom_sf(data = rivers, col = "blue") +
  labs(title = "Sacramento Rivers",
       caption = "Source: Sacramento County") +  
  theme_bw()
```

![](introspatial_files/figure-html/unnamed-chunk-31-1.png)<!-- -->


<div style="margin-bottom:25px;">
</div>
### **tmap**
\


Whether one uses the **tmap** or **ggplot** is a matter of taste, but I find that **tmap** has some benefits, so let's focus on its mapping functions next.

**tmap** uses the same layered logic as **ggplot**. The initial command is `tm_shape()`, which specifies the geography to which the mapping is applied. This is followed by a number of tm_* options that select the type of map and several optional customizations. Check the full list of `tm_` elements [here](https://www.rdocumentation.org/packages/tmap/versions/2.0/topics/tmap-element).
<div style="margin-bottom:25px;">
</div>
#### **Choropleth map**
\

Let's make a choropleth map of percent white in Sacramento.  


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile")
```

![](introspatial_files/figure-html/unnamed-chunk-32-1.png)<!-- -->

You first put the dataset *saccitytracts* inside `tm_shape()`. Because you are plotting polygons, you use `tm_polygons()` next. If you are plotting points, you will use `tm_dots()`. If you are plotting lines, you will use `tm_lines()`. The argument `col = "pwh"` tells R to shade the tracts by the variable *pwh*.  The argument `style = "quantile"` tells R to break up the shading into quintiles, or equal groups of 5.  I find that this is where **tmap** offers a distinct advantage over **ggplot** in that users have greater control over the legend and bin breaks.  **tmap** allows users to specify algorithms to automatically create breaks with the `style` argument.  OSU discusses the importance of breaks and classifications on page 75.  You can also change the number of breaks by setting `n=`. The default is `n=5`. Rather than quintiles, you can show quartiles using `n=4`


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile", n=4)
```

![](introspatial_files/figure-html/unnamed-chunk-33-1.png)<!-- -->

Check out [this breakdown](https://geocompr.robinlovelace.net/adv-map.html#color-settings) of the available classification styles in **tmap**.  

You can overlay multiple features on one map. For example, we can add park polygons on top of city tracts, providing a visual association between parks and percent white.  You will need to add two `tm_shape()` functions each for *saccitytracts* and *parks*.


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile", n=4) +
  tm_shape(parks) +
    tm_polygons(col = "green")
```

![](introspatial_files/figure-html/unnamed-chunk-34-1.png)<!-- -->


The `tm_polygons()` command is a wrapper around two other functions, `tm_fill()` and `tm_borders()`. `tm_fill()` controls the contents of the polygons (color, classification, etc.), while `tm_borders()` does the same for the polygon outlines.

For example, using the same shape (but no variable), we obtain the outlines of the neighborhoods from the `tm_borders()` command.


```r
tm_shape(saccitytracts) +
  tm_borders()
```

![](introspatial_files/figure-html/unnamed-chunk-35-1.png)<!-- -->

Similarly, we obtain a choropleth map without the polygon outlines when we just use the `tm_fill()` command.


```r
tm_shape(saccitytracts) + 
  tm_fill("pwh")
```

![](introspatial_files/figure-html/unnamed-chunk-36-1.png)<!-- -->

When we combine the two commands, we obtain the same map as with `tm_polygons()` (this illustrates how in R one can often obtain the same result in a number of different ways).  Try this on your own.

<div style="margin-bottom:25px;">
</div>
#### **Color Scheme**
\

The argument `palette =` defines the color ranges associated with the bins and determined by the `style` arguments.   Several built-in palettes are contained in **tmap**. For example, using `palette = "Reds"` would yield the following map for our example.


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds") 
```

![](introspatial_files/figure-html/unnamed-chunk-37-1.png)<!-- -->

Under the hood, `“Reds”` refers to one of the color schemes supported by the **RColorBrewer** package (see below).

In addition to the built-in palettes, customized color ranges can be created by specifying a vector with the desired colors as anchors. This will create a spectrum of colors in the map that range between the colors specified in the vector. For instance, if we used `c(“red”, “blue”)`, the color spectrum would move from red to purple, then to blue, with in between shades. In our example:


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette = c("red","blue")) 
```

![](introspatial_files/figure-html/unnamed-chunk-38-1.png)<!-- -->

Not exactly a pretty picture. In order to capture a diverging scale, we insert “white” in between red and blue.


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette = c("red","white", "blue")) 
```

![](introspatial_files/figure-html/unnamed-chunk-39-1.png)<!-- -->

A preferred approach to select a color palette is to chose one of the schemes contained in the **RColorBrewer** package. These are based on the research of cartographer Cynthia Brewer (see the colorbrewer2 [web site](https://colorbrewer2.org/#type=sequential&scheme=BuGn&n=3) for details). ColorBrewer makes a distinction between sequential scales (for a scale that goes from low to high), diverging scales (to highlight how values differ from a central tendency), and qualitative scales (for categorical variables). For each scale, a series of single hue and multi-hue scales are suggested. In the **RColorBrewer** package, these are referred to by a name (e.g., the “Reds” palette we used above is an example). The full list is contained in the **RColorBrewer** [documentation](https://www.rdocumentation.org/packages/RColorBrewer/versions/1.1-2/topics/RColorBrewer).

There are two very useful commands in this package. One sets a color palette by specifying its name and the number of desired categories. The result is a character vector with the hex codes of the corresponding colors.

For example, we select a sequential color scheme going from blue to green, as *BuGn*, by means of the command `brewer.pal`, with the number of categories (6) and the scheme as arguments. The resulting vector contains the HEX codes for the colors.


```r
brewer.pal(6,"BuGn")
```

```
## [1] "#EDF8FB" "#CCECE6" "#99D8C9" "#66C2A4" "#2CA25F" "#006D2C"
```

Using this palette in our map yields the following result.


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette="BuGn") 
```

![](introspatial_files/figure-html/unnamed-chunk-41-1.png)<!-- -->

The command `display.brewer.pal()` allows us to explore different color schemes before applying them to a map. For example:


```r
display.brewer.pal(6,"BuGn")
```

![](introspatial_files/figure-html/unnamed-chunk-42-1.png)<!-- -->


<div style="margin-bottom:25px;">
</div>
#### **Legend**
\

There are many options to change the formatting of the legend.  The automatic title for the legend is not that attractive, since it is simply the variable name. This can be customized by setting the `title` argument.


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds",
              title = "Percent white") 
```

![](introspatial_files/figure-html/unnamed-chunk-43-1.png)<!-- -->

Another important aspect of the legend is its positioning. This is handled through the `tm_layout()` function. This function has a vast number of options, as detailed in the [documentation](https://www.rdocumentation.org/packages/tmap/versions/2.1-1/topics/tm_layout). There are also specialized subsets of layout functions, focused on specific aspects of the map, such as `tm_legend()`, `tm_style()` and `tm_format()`. We illustrate the positioning of the legend.

The default is to position the legend inside the map. Often, this default solution is appropriate, but sometimes further control is needed. The `legend.position` argument to the `tm_layout` function moves the legend around the map, and it takes a vector of two string variables that determine both the horizontal position (“left”, “right”, or “center”) and the vertical position (“top”, “bottom”, or “center”).

For example, if we would want to move the legend to the upper-right position, we would use the following set of commands.


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds",
              title = "Percent white") +
  tm_layout(legend.position = c("right", "top"))
```

![](introspatial_files/figure-html/unnamed-chunk-44-1.png)<!-- -->

There is also the option to position the legend outside the frame of the map. This is accomplished by setting `legend.outside` to TRUE (the default is FALSE), and optionally also specify its position by means of `legend.outside.position()`. The latter can take the values “top”, “bottom”, “right”, and “left”.

For example, to position the legend outside and on the right, would be accomplished by the following commands.


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds",
              title = "Percent white") +
  tm_layout(legend.outside = TRUE, legend.outside.position = "right")
```

![](introspatial_files/figure-html/unnamed-chunk-45-1.png)<!-- -->

We can also customize the size of the legend, its alignment, font, etc. We refer to the documentation for specifics.

<div style="margin-bottom:25px;">
</div>
#### **Title**
\

Another functionality of the `tm_layout()` function is to set a title for the map, and specify its position, size, etc. For example, we can set the title, the `title.size` and the `title.position` as in the example below. We made the font size a bit smaller (0.8) in order not to overwhelm the map, and positioned it in the bottom right-hand corner.


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds",
              title = "Percent white") +
  tm_layout(title = "Percent white Sacramento tracts", title.size = 0.8, 
            title.position = c("right","bottom"),
            legend.outside = TRUE, legend.outside.position = "right")
```

![](introspatial_files/figure-html/unnamed-chunk-46-1.png)<!-- -->

To have a title appear on top (or on the bottom) of the map, we need to set the `main.title` argument of the `tm_layout()` function, with the associated `main.title.position`, as illustrated below (with `title.size` set to 1.25 to have a larger font).


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds",
              title = "Percent white") +
  tm_layout(main.title = "Percent white Sacramento tracts", 
            main.title.size = 1.25, main.title.position="center",
            legend.outside = TRUE, legend.outside.position = "right")
```

![](introspatial_files/figure-html/unnamed-chunk-47-1.png)<!-- -->

<div style="margin-bottom:25px;">
</div>
#### **Other features**
\


We need to add other key elements to the map. First, the scale bar, which you can add using the function `tm_scale_bar()`. Scale bars provide a visual indication of the size of features, and distance between features, on the map.


```r
tm_shape(saccitytracts, unit = "mi") +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds",
              title = "Percent white") +
    tm_scale_bar(breaks = c(0, 1, 2), text.size = 1, position = c("left", "bottom")) +
  tm_layout(main.title = "Percent white Sacramento tracts", 
            main.title.size = 1.25, main.title.position="center",
            legend.outside = TRUE, legend.outside.position = "right")
```

![](introspatial_files/figure-html/unnamed-chunk-48-1.png)<!-- -->

The argument `breaks` tells R the distances to break up and end the bar.  Make sure you use reasonable break points. Sacramento is not, for example, 500 miles wide, so you should not use `c(0,10,500)` (try it and see what happens. You won't like it). Note that the scale is in miles (were in America!).  The default is in kilometers (the rest of the world!), but you can specify the units within `tm_shape()` using the argument `unit`.  Here, we used `unit = "mi"`.  The `position =` argument locates the scale bar on the bottom left of the map.

<br>

We can also add a north arrow, which we can add using the function `tm_compass()`. You can control for the type, size and location of the arrow within this function. I place a 4-star arrow on the bottom right of the map.


```r
tm_shape(saccitytracts, unit = "mi") +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds",
              title = "Percent white") +
    tm_scale_bar(breaks = c(0, 1, 2), text.size = 1, position = c("left", "bottom")) +
    tm_compass(type = "4star", position = c("right", "bottom"))  +
  tm_layout(main.title = "Percent white Sacramento tracts", 
            main.title.size = 1.25, main.title.position="center",
            legend.outside = TRUE, legend.outside.position = "right")
```

![](introspatial_files/figure-html/unnamed-chunk-49-1.png)<!-- -->

We can also eliminate the frame around the map using the argument `frame = FALSE`.


```r
tm_shape(saccitytracts, unit = "mi") +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds",
              title = "Percent white") +
    tm_scale_bar(breaks = c(0, 1, 2), text.size = 1, position = c("left", "bottom")) +
      tm_compass(type = "4star", position = c("right", "bottom"))  +
  tm_layout(main.title = "Percent white Sacramento tracts", 
            main.title.size = 1.25, main.title.position="center",
            legend.outside = TRUE, legend.outside.position = "right",
            frame = FALSE)
```

![](introspatial_files/figure-html/unnamed-chunk-50-1.png)<!-- -->


So far we've created static maps. That is, maps that don't "move".  But, we're all likely used to Google or Bing maps - maps that we can move around and zoom into.  You can make interactive maps in R using the package **tmap**.  Here is another benefit of using **tmap** over **ggplot** - the latter does not provide interactivity.

To make your **tmap** object interactive, use the function `tmap_mode()` and specify "view".


```r
tmap_mode("view")
```

Now that the interactive mode has been ‘turned on’, all maps produced with `tm_shape()` will launch.


```r
tm_shape(saccitytracts) +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds", 
              border.alpha = 0, title = "Percent white")
```

```{=html}
<div id="htmlwidget-f5c43e90a94d7823ef6d" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-f5c43e90a94d7823ef6d">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"createMapPane","args":["tmap401",401]},{"method":"addProviderTiles","args":["Esri.WorldGrayCanvas",null,"Esri.WorldGrayCanvas",{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"pane":"tilePane"}]},{"method":"addProviderTiles","args":["OpenStreetMap",null,"OpenStreetMap",{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"pane":"tilePane"}]},{"method":"addProviderTiles","args":["Esri.WorldTopoMap",null,"Esri.WorldTopoMap",{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"pane":"tilePane"}]},{"method":"addPolygons","args":[[[[{"lng":[-121.507582,-121.507543,-121.507514,-121.507774,-121.507737,-121.507976,-121.509925,-121.512704,-121.516841,-121.521182,-121.523996,-121.52466,-121.524462,-121.525731,-121.529841,-121.531185,-121.531414,-121.531309,-121.529871,-121.527606,-121.524423,-121.519455,-121.519751,-121.51947,-121.512404,-121.510475,-121.507582],"lat":[38.676903,38.673418,38.670816,38.670816,38.66654,38.665335,38.665646,38.665648,38.666305,38.665996,38.66645,38.666446,38.671625,38.673443,38.674456,38.677031,38.677607,38.680081,38.680005,38.680004,38.679874,38.677921,38.676905,38.675594,38.677036,38.676892,38.676903]}]],[[{"lng":[-121.474761,-121.468138,-121.465208,-121.461522,-121.461426,-121.462834,-121.464247,-121.470933,-121.473609,-121.477613,-121.476192,-121.474761],"lat":[38.585069,38.583443,38.583056,38.58197,38.581471,38.578234,38.574981,38.576737,38.577452,38.57852,38.581802,38.585069]}]],[[{"lng":[-121.472905,-121.471668,-121.470249,-121.466821,-121.466355,-121.461208,-121.45936,-121.457538,-121.448495,-121.443877,-121.438118,-121.435403,-121.440636,-121.446882,-121.448161,-121.449431,-121.450046,-121.45019,-121.45028,-121.455336,-121.461248,-121.462489,-121.463762,-121.466335,-121.470373,-121.473469,-121.476403,-121.475558,-121.472905],"lat":[38.597893,38.598439,38.600312,38.608395,38.611163,38.6111,38.611125,38.611136,38.611123,38.611121,38.611064,38.610821,38.605988,38.600211,38.598732,38.596446,38.593839,38.590291,38.59032,38.590591,38.589159,38.58877,38.588618,38.588867,38.590683,38.592956,38.596121,38.596654,38.597893]}]],[[{"lng":[-121.445746,-121.442113,-121.435292,-121.434458,-121.432758,-121.427541,-121.427763,-121.427515,-121.432848,-121.433813,-121.438691,-121.44259,-121.445559,-121.445746],"lat":[38.531788,38.532146,38.532262,38.532331,38.532347,38.532303,38.530768,38.525077,38.525041,38.525034,38.525001,38.524974,38.531392,38.531788]}]],[[{"lng":[-121.429138,-121.421638,-121.421638,-121.423412,-121.419896,-121.417591,-121.417602,-121.410683,-121.410573,-121.405735,-121.405313,-121.406244,-121.411775,-121.412988,-121.419931,-121.430594,-121.429759,-121.429168,-121.429168,-121.429156,-121.429138],"lat":[38.654607,38.654524,38.653909,38.651008,38.650971,38.649954,38.647201,38.647132,38.639894,38.639816,38.63862,38.637756,38.63264,38.632476,38.632739,38.632925,38.636479,38.638993,38.640116,38.642927,38.654607]}]],[[{"lng":[-121.498892,-121.49651,-121.489851,-121.484264,-121.477613,-121.480001,-121.485321,-121.486653,-121.488474,-121.492716,-121.493672,-121.494629,-121.501271,-121.498892],"lat":[38.578047,38.58356,38.581785,38.580295,38.57852,38.57301,38.57443,38.574785,38.574026,38.575156,38.572962,38.570769,38.572539,38.578047]}]],[[{"lng":[-121.447553,-121.445968,-121.442399,-121.436125,-121.435315,-121.433327,-121.431261,-121.429241,-121.428525,-121.446674,-121.446708,-121.446736,-121.446811,-121.447553],"lat":[38.467031,38.467142,38.466434,38.465082,38.464581,38.463588,38.463781,38.460007,38.458623,38.463235,38.463681,38.464065,38.464752,38.467031]}]],[[{"lng":[-121.482384,-121.478373,-121.477033,-121.475715,-121.470369,-121.469039,-121.471151,-121.472436,-121.474412,-121.475844,-121.480354,-121.485698,-121.482384],"lat":[38.567506,38.566436,38.566079,38.565727,38.564301,38.563946,38.559082,38.556125,38.556821,38.557203,38.558404,38.559828,38.567506]}]],[[{"lng":[-121.444652,-121.441507,-121.435436,-121.434696,-121.429857,-121.425127,-121.423741,-121.432836,-121.436154,-121.436194,-121.444991,-121.444652],"lat":[38.447177,38.443855,38.441825,38.441258,38.441208,38.440486,38.438001,38.437938,38.437935,38.441462,38.441479,38.447177]}]],[[{"lng":[-121.453086,-121.450311,-121.454642,-121.46165,-121.462307,-121.464188,-121.466354,-121.464487,-121.46301,-121.45922,-121.453086],"lat":[38.482303,38.474295,38.474316,38.474516,38.474471,38.474487,38.481526,38.481495,38.480937,38.481682,38.482303]}]],[[{"lng":[-121.507582,-121.50751,-121.507423,-121.493603,-121.493401,-121.495643,-121.504235,-121.507514,-121.507543,-121.507582],"lat":[38.676903,38.68193,38.685217,38.685217,38.670807,38.670808,38.670816,38.670816,38.673418,38.676903]}]],[[{"lng":[-121.427541,-121.421147,-121.421153,-121.415146,-121.412811,-121.410133,-121.409149,-121.409124,-121.411799,-121.414432,-121.419937,-121.420091,-121.427515,-121.427763,-121.427541],"lat":[38.532303,38.532365,38.531984,38.53197,38.53207,38.532081,38.52927,38.525063,38.525065,38.525066,38.525071,38.525071,38.525077,38.530768,38.532303]}]],[[{"lng":[-121.517356,-121.512483,-121.511117,-121.508451,-121.505744,-121.505722,-121.505434,-121.50363,-121.503986,-121.497854,-121.49788,-121.506254,-121.508085,-121.51256,-121.517356],"lat":[38.625027,38.627592,38.627435,38.627449,38.62744,38.621306,38.617175,38.613761,38.612789,38.61274,38.608065,38.606919,38.606233,38.614858,38.625027]}]],[[{"lng":[-121.491264,-121.482736,-121.48035,-121.480722,-121.480345,-121.480341,-121.480336,-121.486859,-121.49335,-121.49239,-121.491963,-121.491876,-121.491264],"lat":[38.495727,38.49576,38.49577,38.492209,38.490448,38.486942,38.481431,38.481422,38.481412,38.483828,38.489044,38.493927,38.495727]}]],[[{"lng":[-121.49788,-121.491807,-121.490827,-121.48127,-121.483114,-121.484197,-121.484191,-121.484272,-121.478881,-121.475527,-121.475537,-121.475536,-121.475561,-121.47555,-121.47451,-121.472905,-121.475558,-121.476403,-121.476653,-121.478961,-121.481284,-121.48553,-121.488586,-121.49294,-121.496637,-121.500517,-121.505114,-121.508085,-121.506254,-121.49788],"lat":[38.608065,38.608542,38.608516,38.607288,38.612291,38.615341,38.622421,38.627519,38.629607,38.629652,38.617601,38.617264,38.611243,38.604366,38.59968,38.597893,38.596654,38.596121,38.59636,38.598938,38.600445,38.601977,38.602401,38.602618,38.602501,38.601902,38.600503,38.606233,38.606919,38.608065]}]],[[{"lng":[-121.505073,-121.496695,-121.494648,-121.49335,-121.486859,-121.480336,-121.466354,-121.464188,-121.475655,-121.479375,-121.482819,-121.487518,-121.487318,-121.494188,-121.497895,-121.503123,-121.503221,-121.50423,-121.504467,-121.505073],"lat":[38.481269,38.481371,38.481396,38.481412,38.481422,38.481431,38.481526,38.474487,38.474287,38.473379,38.472451,38.472514,38.470961,38.468872,38.468173,38.467189,38.469744,38.475109,38.476831,38.481269]}]],[[{"lng":[-121.447756,-121.447745,-121.447739,-121.447344,-121.443513,-121.440926,-121.438415,-121.429138,-121.429156,-121.429168,-121.429168,-121.429759,-121.430594,-121.438394,-121.441658,-121.44666,-121.447346,-121.447364,-121.447311,-121.447787,-121.447756],"lat":[38.641433,38.647577,38.654827,38.654957,38.656463,38.654735,38.654708,38.654607,38.642927,38.640116,38.638993,38.636479,38.632925,38.633002,38.633022,38.633052,38.632864,38.636693,38.640309,38.64032,38.641433]}]],[[{"lng":[-121.450475,-121.44787,-121.446743,-121.447136,-121.444836,-121.439215,-121.436682,-121.434867,-121.433537,-121.432869,-121.431961,-121.433328,-121.440678,-121.441705,-121.448571,-121.449873,-121.450475],"lat":[38.571276,38.576162,38.57842,38.57869,38.581846,38.579655,38.578432,38.576673,38.5736,38.570934,38.56731,38.567203,38.569133,38.569405,38.571211,38.571123,38.571276]}]],[[{"lng":[-121.44259,-121.438691,-121.433813,-121.432848,-121.427515,-121.427481,-121.427459,-121.427454,-121.427447,-121.435862,-121.438352,-121.439229,-121.439757,-121.440148,-121.44197,-121.44259],"lat":[38.524974,38.525001,38.525034,38.525041,38.525077,38.519732,38.515993,38.513894,38.510522,38.510507,38.515855,38.517704,38.518818,38.51967,38.523625,38.524974]}]],[[{"lng":[-121.544963,-121.538928,-121.537785,-121.535595,-121.53495,-121.534727,-121.53473,-121.540593,-121.542106,-121.543404,-121.544121,-121.534985,-121.534872,-121.533674,-121.530902,-121.528691,-121.52355,-121.523768,-121.522562,-121.521822,-121.52282,-121.525758,-121.53092,-121.532176,-121.533058,-121.533623,-121.534336,-121.535153,-121.536971,-121.540055,-121.544963],"lat":[38.483903,38.486741,38.487636,38.489932,38.490927,38.491532,38.493559,38.493342,38.494158,38.496322,38.497596,38.498198,38.498697,38.498005,38.497407,38.496955,38.495906,38.493322,38.492052,38.49172,38.490551,38.489356,38.487716,38.48695,38.485959,38.48465,38.483155,38.482466,38.481283,38.479973,38.483903]}]],[[{"lng":[-121.446674,-121.428525,-121.428415,-121.42768,-121.427137,-121.425872,-121.427937,-121.431675,-121.434118,-121.437826,-121.43979,-121.444649,-121.444863,-121.445585,-121.445909,-121.446238,-121.446481,-121.446674],"lat":[38.463235,38.458623,38.458311,38.454836,38.452378,38.449722,38.449814,38.451293,38.451199,38.450595,38.448882,38.448801,38.452108,38.455679,38.457307,38.458975,38.46069,38.463235]}]],[[{"lng":[-121.470798,-121.460444,-121.459275,-121.456899,-121.454981,-121.454097,-121.453086,-121.45922,-121.46301,-121.464487,-121.466354,-121.469127,-121.470798],"lat":[38.495868,38.496047,38.496013,38.491536,38.487543,38.485173,38.482303,38.481682,38.480937,38.481495,38.481526,38.490507,38.495868]}]],[[{"lng":[-121.497892,-121.4963,-121.495434,-121.493202,-121.491342,-121.489875,-121.488412,-121.487986,-121.487326,-121.484191,-121.484197,-121.483114,-121.48127,-121.490827,-121.491807,-121.49788,-121.497854,-121.497892],"lat":[38.621255,38.621411,38.621851,38.622711,38.622681,38.622886,38.622764,38.622717,38.622501,38.622421,38.615341,38.612291,38.607288,38.608516,38.608542,38.608065,38.61274,38.621255]}]],[[{"lng":[-121.474322,-121.467087,-121.46615,-121.465178,-121.456912,-121.447739,-121.447745,-121.447756,-121.449837,-121.45692,-121.468121,-121.471691,-121.474322],"lat":[38.655347,38.655334,38.655169,38.655024,38.654932,38.654827,38.647577,38.641433,38.641325,38.641524,38.641828,38.641512,38.655347]}]],[[{"lng":[-121.502176,-121.500329,-121.499371,-121.498472,-121.491047,-121.48971,-121.485698,-121.487232,-121.487469,-121.488236,-121.488267,-121.48867,-121.496039,-121.497139,-121.500395,-121.502176],"lat":[38.556434,38.559533,38.561142,38.563222,38.561249,38.560893,38.559828,38.555859,38.555094,38.55264,38.552542,38.551278,38.5526,38.552801,38.554838,38.556434]}]],[[{"lng":[-121.454449,-121.44443,-121.436626,-121.423299,-121.421645,-121.416577,-121.427857,-121.42787,-121.432953,-121.434173,-121.438211,-121.439162,-121.444012,-121.45384,-121.456384,-121.459803,-121.460532,-121.465064,-121.460645,-121.454449],"lat":[38.559917,38.557161,38.555008,38.551357,38.55102,38.544644,38.544634,38.546858,38.546796,38.546781,38.546732,38.546722,38.546671,38.546757,38.553198,38.556701,38.557484,38.562835,38.561618,38.559917]}]],[[{"lng":[-121.505239,-121.500932,-121.498601,-121.49842,-121.498017,-121.487431,-121.483144,-121.483036,-121.48291,-121.482839,-121.482759,-121.482736,-121.491264,-121.503261,-121.506064,-121.505964,-121.505864,-121.505239],"lat":[38.504641,38.513314,38.520429,38.520982,38.520716,38.52097,38.521114,38.510222,38.506599,38.504563,38.499405,38.49576,38.495727,38.495679,38.495632,38.499403,38.502747,38.504641]}]],[[{"lng":[-121.513413,-121.512474,-121.505773,-121.498601,-121.500932,-121.505239,-121.505883,-121.506339,-121.508407,-121.511781,-121.515557,-121.516569,-121.516269,-121.513413],"lat":[38.521419,38.521249,38.521612,38.520429,38.513314,38.504641,38.504593,38.502775,38.503308,38.506147,38.507768,38.50779,38.516723,38.521419]}]],[[{"lng":[-121.506275,-121.501314,-121.498304,-121.49651,-121.498892,-121.501537,-121.508266,-121.506275],"lat":[38.586288,38.58362,38.582818,38.58356,38.578047,38.578749,38.580505,38.586288]}]],[[{"lng":[-121.436855,-121.435862,-121.434576,-121.436204,-121.436812,-121.436855],"lat":[38.510495,38.510507,38.507712,38.507556,38.507355,38.510495]}]],[[{"lng":[-121.493768,-121.493612,-121.492829,-121.487087,-121.484323,-121.481879,-121.480952,-121.48625,-121.49431,-121.495899,-121.49536,-121.493768],"lat":[38.535148,38.535636,38.538135,38.539138,38.539613,38.531701,38.528677,38.528304,38.528351,38.528629,38.530214,38.535148]}]],[[{"lng":[-121.427459,-121.419919,-121.414297,-121.409108,-121.409081,-121.415222,-121.415863,-121.422496,-121.427447,-121.427454,-121.427459],"lat":[38.515993,38.516061,38.51611,38.516156,38.510551,38.510542,38.510542,38.51053,38.510522,38.513894,38.515993]}]],[[{"lng":[-121.415295,-121.413287,-121.411226,-121.405235,-121.405197,-121.401492,-121.396574,-121.397192,-121.397457,-121.399791,-121.400331,-121.399163,-121.408189,-121.40926,-121.40921,-121.409035,-121.40928,-121.410224,-121.414673,-121.415295],"lat":[38.574112,38.574342,38.574323,38.570699,38.572045,38.571479,38.570683,38.568878,38.568035,38.567838,38.563891,38.562605,38.560681,38.560718,38.562301,38.565364,38.56993,38.570824,38.572955,38.574112]}]],[[{"lng":[-121.511566,-121.508266,-121.501537,-121.498892,-121.501271,-121.503924,-121.508266,-121.509138,-121.511566],"lat":[38.575005,38.580505,38.578749,38.578047,38.572539,38.573246,38.574413,38.575563,38.575005]}]],[[{"lng":[-121.477613,-121.473609,-121.470933,-121.464247,-121.465934,-121.466652,-121.473326,-121.475993,-121.480001,-121.477613],"lat":[38.57852,38.577452,38.576737,38.574981,38.571099,38.569448,38.571229,38.571941,38.57301,38.57852]}]],[[{"lng":[-121.427515,-121.420091,-121.419937,-121.414432,-121.411799,-121.409124,-121.409108,-121.414297,-121.419919,-121.427459,-121.427481,-121.427515],"lat":[38.525077,38.525071,38.525071,38.525066,38.525065,38.525063,38.516156,38.51611,38.516061,38.515993,38.519732,38.525077]}]],[[{"lng":[-121.430761,-121.427406,-121.422988,-121.427168,-121.436875,-121.446594,-121.450311,-121.453086,-121.451029,-121.448925,-121.44725,-121.44511,-121.44159,-121.440706,-121.430761],"lat":[38.481616,38.478442,38.474281,38.474217,38.474254,38.474276,38.474295,38.482303,38.482602,38.482128,38.481152,38.479209,38.481301,38.481563,38.481616]}]],[[{"lng":[-121.524002,-121.52352,-121.517356,-121.51256,-121.508085,-121.505114,-121.506278,-121.508668,-121.511566,-121.515766,-121.51895,-121.524954,-121.524824,-121.524061,-121.524017,-121.521554,-121.523693,-121.524002],"lat":[38.6212,38.621766,38.625027,38.614858,38.606233,38.600503,38.59976,38.596803,38.600904,38.602704,38.603212,38.604171,38.605599,38.608103,38.610204,38.613995,38.616104,38.6212]}]],[[{"lng":[-121.449873,-121.448571,-121.441705,-121.440678,-121.433328,-121.431961,-121.431228,-121.43056,-121.421645,-121.423299,-121.436626,-121.44443,-121.454449,-121.451759,-121.449873],"lat":[38.571123,38.571211,38.569405,38.569133,38.567203,38.56731,38.564401,38.562125,38.55102,38.551357,38.555008,38.557161,38.559917,38.566175,38.571123]}]],[[{"lng":[-121.449713,-121.443942,-121.441969,-121.438575,-121.434483,-121.43293,-121.427673,-121.421167,-121.413173,-121.410133,-121.412811,-121.415146,-121.421153,-121.421147,-121.427541,-121.432758,-121.434458,-121.435292,-121.442113,-121.445746,-121.447398,-121.449713],"lat":[38.539418,38.539439,38.539447,38.539483,38.539526,38.539542,38.539597,38.539605,38.539564,38.532081,38.53207,38.53197,38.531984,38.532365,38.532303,38.532347,38.532331,38.532262,38.532146,38.531788,38.535171,38.539418]}]],[[{"lng":[-121.461522,-121.458301,-121.454744,-121.451607,-121.450906,-121.450604,-121.44925,-121.444836,-121.447136,-121.446743,-121.44787,-121.450475,-121.454014,-121.455335,-121.464247,-121.462834,-121.461426,-121.461522],"lat":[38.58197,38.58134,38.581998,38.583857,38.585382,38.585437,38.583669,38.581846,38.57869,38.57842,38.576162,38.571276,38.572235,38.572593,38.574981,38.578234,38.581471,38.58197]}]],[[{"lng":[-121.459275,-121.460444,-121.470798,-121.472346,-121.46702,-121.464996,-121.464485,-121.461422,-121.459275],"lat":[38.496013,38.496047,38.495868,38.500947,38.500979,38.503169,38.500439,38.500045,38.496013]}]],[[{"lng":[-121.464767,-121.464716,-121.46472,-121.461677,-121.460101,-121.4578,-121.45384,-121.451646,-121.449713,-121.45549,-121.459125,-121.460094,-121.463748,-121.46476,-121.464767],"lat":[38.541193,38.546275,38.547184,38.546801,38.546789,38.546748,38.546757,38.542843,38.539418,38.539444,38.539445,38.539446,38.539446,38.539527,38.541193]}]],[[{"lng":[-121.488236,-121.487856,-121.483623,-121.481608,-121.479826,-121.475812,-121.475758,-121.478938,-121.484323,-121.488267,-121.488236],"lat":[38.55264,38.551746,38.549966,38.549895,38.549911,38.549935,38.541064,38.54054,38.539613,38.552542,38.55264]}]],[[{"lng":[-121.393898,-121.39086,-121.379471,-121.375124,-121.377956,-121.378054,-121.386347,-121.386309,-121.390885,-121.398728,-121.398988,-121.39821,-121.399414,-121.400017,-121.403344,-121.403315,-121.402412,-121.396468,-121.393634,-121.393898],"lat":[38.557248,38.557614,38.558852,38.553093,38.55146,38.547071,38.547038,38.543097,38.544396,38.546637,38.54707,38.549008,38.550863,38.550726,38.550712,38.55358,38.554556,38.555903,38.555656,38.557248]}]],[[{"lng":[-121.483144,-121.478672,-121.477548,-121.475265,-121.473596,-121.472346,-121.470798,-121.48035,-121.482736,-121.482759,-121.482839,-121.48291,-121.483036,-121.483144],"lat":[38.521114,38.521312,38.517691,38.510328,38.504916,38.500947,38.495868,38.49577,38.49576,38.499405,38.504563,38.506599,38.510222,38.521114]}]],[[{"lng":[-121.518966,-121.517868,-121.507251,-121.505909,-121.498472,-121.499371,-121.500329,-121.502176,-121.502227,-121.503588,-121.507598,-121.50854,-121.508598,-121.509531,-121.509923,-121.510905,-121.512618,-121.51129,-121.514066,-121.516537,-121.518451,-121.520965,-121.518966],"lat":[38.568705,38.568503,38.565594,38.565232,38.563222,38.561142,38.559533,38.556434,38.556347,38.554032,38.547272,38.545748,38.544753,38.540989,38.540401,38.540784,38.541666,38.54629,38.553105,38.557895,38.561275,38.566205,38.568705]}]],[[{"lng":[-121.450604,-121.449058,-121.447198,-121.443689,-121.440255,-121.434464,-121.432457,-121.430484,-121.425055,-121.422758,-121.420059,-121.419928,-121.421335,-121.42155,-121.422592,-121.431961,-121.432869,-121.433537,-121.434867,-121.436682,-121.439215,-121.444836,-121.44925,-121.450604],"lat":[38.585437,38.58606,38.58773,38.58455,38.58271,38.58157,38.581844,38.582927,38.583576,38.583368,38.581383,38.580593,38.577284,38.575857,38.568914,38.56731,38.570934,38.5736,38.576673,38.578432,38.579655,38.581846,38.583669,38.585437]}]],[[{"lng":[-121.419808,-121.413082,-121.411775,-121.406244,-121.405313,-121.403146,-121.402459,-121.3887,-121.383262,-121.383201,-121.382941,-121.392169,-121.40138,-121.410655,-121.411074,-121.415281,-121.416419,-121.418137,-121.419381,-121.41991,-121.419808],"lat":[38.62523,38.631453,38.63264,38.637756,38.63862,38.64061,38.640608,38.643682,38.642191,38.63989,38.637155,38.6336,38.630075,38.626469,38.62616,38.624541,38.624506,38.623894,38.623247,38.623023,38.62523]}]],[[{"lng":[-121.540138,-121.538214,-121.532897,-121.533312,-121.539859,-121.550488,-121.550512,-121.549967,-121.550085,-121.550091,-121.557897,-121.557886,-121.550099,-121.548885,-121.546811,-121.543754,-121.540138],"lat":[38.667639,38.665393,38.655768,38.655763,38.655716,38.655623,38.661139,38.662011,38.670508,38.671059,38.671057,38.671362,38.671348,38.671278,38.670943,38.669899,38.667639]}]],[[{"lng":[-121.521849,-121.513413,-121.516269,-121.516569,-121.515557,-121.511781,-121.508407,-121.506339,-121.505883,-121.505239,-121.505864,-121.505964,-121.506064,-121.506189,-121.505073,-121.504467,-121.50423,-121.508848,-121.510714,-121.516668,-121.517292,-121.517767,-121.518264,-121.521998,-121.522311,-121.522148,-121.521849],"lat":[38.523163,38.521419,38.516723,38.50779,38.507768,38.506147,38.503308,38.502775,38.504593,38.504641,38.502747,38.499403,38.495632,38.489593,38.481269,38.476831,38.475109,38.478669,38.4812,38.495476,38.497641,38.501317,38.502795,38.509266,38.510844,38.51617,38.523163]}]],[[{"lng":[-121.512483,-121.500517,-121.493202,-121.489341,-121.48416,-121.47781,-121.477782,-121.475538,-121.475527,-121.478881,-121.484272,-121.48847,-121.492895,-121.493242,-121.497899,-121.505744,-121.508451,-121.511117,-121.512483],"lat":[38.627592,38.633878,38.637727,38.63918,38.640024,38.640786,38.638599,38.635261,38.629652,38.629607,38.627519,38.627488,38.627447,38.627445,38.62742,38.62744,38.627449,38.627435,38.627592]}]],[[{"lng":[-121.439229,-121.438352,-121.435862,-121.436855,-121.445145,-121.445203,-121.441457,-121.441526,-121.441562,-121.439229],"lat":[38.517704,38.515855,38.510507,38.510495,38.510459,38.512087,38.512141,38.515795,38.517675,38.517704]}]],[[{"lng":[-121.435862,-121.427447,-121.422496,-121.415863,-121.415222,-121.409081,-121.409042,-121.412333,-121.416679,-121.416685,-121.422066,-121.422856,-121.422939,-121.427404,-121.429708,-121.433196,-121.434576,-121.435862],"lat":[38.510507,38.510522,38.51053,38.510542,38.510542,38.510551,38.503683,38.50377,38.503744,38.503241,38.503234,38.503668,38.504223,38.504692,38.503996,38.504782,38.507712,38.510507]}]],[[{"lng":[-121.45384,-121.444012,-121.439162,-121.438211,-121.434173,-121.432953,-121.42787,-121.427857,-121.416577,-121.414636,-121.413173,-121.421167,-121.427673,-121.43293,-121.434483,-121.438575,-121.441969,-121.443942,-121.449713,-121.451646,-121.45384],"lat":[38.546757,38.546671,38.546722,38.546732,38.546781,38.546796,38.546858,38.544634,38.544644,38.542009,38.539564,38.539605,38.539597,38.539542,38.539526,38.539483,38.539447,38.539439,38.539418,38.542843,38.546757]}]],[[{"lng":[-121.52355,-121.521602,-121.516668,-121.510714,-121.514438,-121.516198,-121.517543,-121.51766,-121.518501,-121.520055,-121.52155,-121.522905,-121.527868,-121.533623,-121.533058,-121.532176,-121.53092,-121.525758,-121.52282,-121.521822,-121.522562,-121.523768,-121.52355],"lat":[38.495906,38.49558,38.495476,38.4812,38.481119,38.48052,38.482763,38.483768,38.483723,38.483203,38.483341,38.483839,38.483515,38.48465,38.485959,38.48695,38.487716,38.489356,38.490551,38.49172,38.492052,38.493322,38.495906]}]],[[{"lng":[-121.54548,-121.540823,-121.531396,-121.531237,-121.530532,-121.530364,-121.527527,-121.52724,-121.52624,-121.525988,-121.527225,-121.528691,-121.530902,-121.533674,-121.534872,-121.535364,-121.537911,-121.541951,-121.542764,-121.54548],"lat":[38.515615,38.516504,38.51727,38.515831,38.513878,38.513908,38.506687,38.506756,38.505034,38.503413,38.501074,38.496955,38.497407,38.498005,38.498697,38.499284,38.504223,38.508499,38.510062,38.515615]}]],[[{"lng":[-121.502176,-121.500395,-121.497139,-121.496039,-121.48867,-121.488267,-121.484323,-121.487087,-121.492829,-121.493612,-121.493768,-121.503146,-121.503631,-121.505036,-121.506206,-121.509923,-121.509531,-121.508598,-121.50854,-121.507598,-121.503588,-121.502227,-121.502176],"lat":[38.556434,38.554838,38.552801,38.5526,38.551278,38.552542,38.539613,38.539138,38.538135,38.535636,38.535148,38.537145,38.537467,38.538386,38.538912,38.540401,38.540989,38.544753,38.545748,38.547272,38.554032,38.556347,38.556434]}]],[[{"lng":[-121.46476,-121.463748,-121.460094,-121.460086,-121.461365,-121.464713,-121.46476],"lat":[38.539527,38.539446,38.539446,38.534025,38.534006,38.533606,38.539527]}],[{"lng":[-121.461643,-121.464626,-121.4647,-121.462318,-121.462327,-121.462323,-121.461634,-121.461643],"lat":[38.52491,38.524951,38.532137,38.532136,38.526143,38.525685,38.525694,38.52491]}]],[[{"lng":[-121.518966,-121.511566,-121.509138,-121.508266,-121.503924,-121.501271,-121.494629,-121.496056,-121.497004,-121.498325,-121.498472,-121.505909,-121.507251,-121.517868,-121.518966],"lat":[38.568705,38.575005,38.575563,38.574413,38.573246,38.572539,38.570769,38.567493,38.56532,38.565672,38.563222,38.565232,38.565594,38.568503,38.568705]}]],[[{"lng":[-121.436875,-121.427168,-121.422988,-121.419924,-121.418969,-121.415495,-121.414695,-121.411296,-121.412655,-121.428525,-121.429241,-121.431261,-121.434755,-121.435429,-121.436875],"lat":[38.474254,38.474217,38.474281,38.474347,38.474339,38.46723,38.465234,38.458132,38.457959,38.458623,38.460007,38.463781,38.470296,38.471555,38.474254]}]],[[{"lng":[-121.505744,-121.497899,-121.497892,-121.497854,-121.503986,-121.50363,-121.505434,-121.505722,-121.505744],"lat":[38.62744,38.62742,38.621255,38.61274,38.612789,38.613761,38.617175,38.621306,38.62744]}]],[[{"lng":[-121.488236,-121.487469,-121.487232,-121.485698,-121.480354,-121.475844,-121.475785,-121.475812,-121.479826,-121.481608,-121.483623,-121.487856,-121.488236],"lat":[38.55264,38.555094,38.555859,38.559828,38.558404,38.557203,38.553605,38.549935,38.549911,38.549895,38.549966,38.551746,38.55264]}]],[[{"lng":[-121.494629,-121.487703,-121.482384,-121.485698,-121.48971,-121.491047,-121.498472,-121.498325,-121.497004,-121.496056,-121.494629],"lat":[38.570769,38.568923,38.567506,38.559828,38.560893,38.561249,38.563222,38.565672,38.56532,38.567493,38.570769]}]],[[{"lng":[-121.526325,-121.520785,-121.518508,-121.532706,-121.533694,-121.53686,-121.53736,-121.535752,-121.537509,-121.539068,-121.539689,-121.539685,-121.539684,-121.535904,-121.532128,-121.526325],"lat":[38.643031,38.632763,38.627774,38.627754,38.62756,38.629585,38.630927,38.636502,38.638743,38.641789,38.642114,38.642319,38.64303,38.642788,38.643043,38.643031]}]],[[{"lng":[-121.524002,-121.523693,-121.521554,-121.524017,-121.524061,-121.524824,-121.524954,-121.527667,-121.534797,-121.534809,-121.536446,-121.537543,-121.540529,-121.54055,-121.544567,-121.545282,-121.546661,-121.547215,-121.538122,-121.536991,-121.534003,-121.530093,-121.52425,-121.524117,-121.524002],"lat":[38.6212,38.616104,38.613995,38.610204,38.608103,38.605599,38.604171,38.604604,38.60295,38.605128,38.604713,38.604228,38.602386,38.600769,38.597704,38.597933,38.599529,38.599737,38.613018,38.614233,38.616274,38.618319,38.621379,38.621449,38.6212]}]],[[{"lng":[-121.40926,-121.408189,-121.399163,-121.393335,-121.382848,-121.382639,-121.382175,-121.379471,-121.39086,-121.393898,-121.409349,-121.409435,-121.40926],"lat":[38.560718,38.560681,38.562605,38.564065,38.566761,38.564114,38.562815,38.558852,38.557614,38.557248,38.55381,38.557678,38.560718]}]],[[{"lng":[-121.467588,-121.463687,-121.459016,-121.456943,-121.452146,-121.447352,-121.447462,-121.447766,-121.448915,-121.449926,-121.450939,-121.446173,-121.445025,-121.443882,-121.443877,-121.448495,-121.457538,-121.45936,-121.461208,-121.466355,-121.466966,-121.466832,-121.467737,-121.467588],"lat":[38.61974,38.62208,38.62402,38.623939,38.625524,38.625395,38.620041,38.618647,38.616579,38.615622,38.61478,38.614743,38.613837,38.612934,38.611121,38.611123,38.611136,38.611125,38.6111,38.611163,38.611176,38.611763,38.617387,38.61974]}]],[[{"lng":[-121.545357,-121.53736,-121.53686,-121.533694,-121.54535,-121.545357],"lat":[38.630908,38.630927,38.629585,38.62756,38.627258,38.630908]}],[{"lng":[-121.533694,-121.532706,-121.518508,-121.518299,-121.517356,-121.52352,-121.524002,-121.524117,-121.52425,-121.52892,-121.533694],"lat":[38.62756,38.627754,38.627774,38.627277,38.625027,38.621766,38.6212,38.621449,38.621379,38.625021,38.62756]}]],[[{"lng":[-121.440706,-121.44159,-121.44511,-121.44725,-121.448925,-121.451029,-121.453086,-121.454097,-121.447233,-121.447213,-121.444435,-121.441188,-121.440706],"lat":[38.481563,38.481301,38.479209,38.481152,38.482128,38.482602,38.482303,38.485173,38.485167,38.483761,38.484253,38.48483,38.481563]}]],[[{"lng":[-121.47972,-121.471747,-121.47086,-121.465796,-121.466931,-121.465524,-121.465854,-121.467277,-121.469706,-121.477548,-121.478672,-121.47972],"lat":[38.524835,38.52489,38.521181,38.52119,38.520669,38.520454,38.516679,38.51776,38.51761,38.517691,38.521312,38.524835]}],[{"lng":[-121.470148,-121.465853,-121.465249,-121.465565,-121.466901,-121.466501,-121.468776,-121.468775,-121.469247,-121.470148],"lat":[38.524903,38.524943,38.524304,38.523738,38.523265,38.522513,38.522477,38.523556,38.52458,38.524903]}]],[[{"lng":[-121.48035,-121.470798,-121.469127,-121.466354,-121.480336,-121.480341,-121.480345,-121.480722,-121.48035],"lat":[38.49577,38.495868,38.490507,38.481526,38.481431,38.486942,38.490448,38.492209,38.49577]}]],[[{"lng":[-121.444649,-121.43979,-121.437826,-121.434118,-121.431675,-121.427937,-121.425872,-121.425223,-121.425795,-121.425617,-121.422777,-121.422177,-121.421029,-121.423741,-121.425127,-121.429857,-121.434696,-121.435436,-121.441507,-121.444652,-121.444649],"lat":[38.448801,38.448882,38.450595,38.451199,38.451293,38.449814,38.449722,38.448124,38.445202,38.442351,38.438677,38.438333,38.43803,38.438001,38.440486,38.441208,38.441258,38.441825,38.443855,38.447177,38.448801]}]],[[{"lng":[-121.480001,-121.475993,-121.473326,-121.466652,-121.469039,-121.470369,-121.475715,-121.477033,-121.478373,-121.482384,-121.481435,-121.480001],"lat":[38.57301,38.571941,38.571229,38.569448,38.563946,38.564301,38.565727,38.566079,38.566436,38.567506,38.5697,38.57301]}]],[[{"lng":[-121.466652,-121.465934,-121.464247,-121.455335,-121.454014,-121.450475,-121.449873,-121.451759,-121.454449,-121.460645,-121.465064,-121.469039,-121.466652],"lat":[38.569448,38.571099,38.574981,38.572593,38.572235,38.571276,38.571123,38.566175,38.559917,38.561618,38.562835,38.563946,38.569448]}]],[[{"lng":[-121.512618,-121.510905,-121.509923,-121.506206,-121.511248,-121.513276,-121.512824,-121.513413,-121.521849,-121.525177,-121.529922,-121.530767,-121.530976,-121.529774,-121.528467,-121.520366,-121.513199,-121.512618],"lat":[38.541666,38.540784,38.540401,38.538912,38.531392,38.528227,38.522729,38.521419,38.523163,38.522682,38.52678,38.527906,38.52897,38.53121,38.532106,38.535367,38.540743,38.541666]}]],[[{"lng":[-121.431961,-121.422592,-121.422665,-121.419995,-121.417224,-121.412425,-121.40926,-121.409435,-121.409349,-121.421645,-121.43056,-121.431228,-121.431961],"lat":[38.56731,38.568914,38.568388,38.562441,38.560357,38.560243,38.560718,38.557678,38.55381,38.55102,38.562125,38.564401,38.56731]}]],[[{"lng":[-121.447352,-121.447346,-121.44666,-121.441658,-121.438394,-121.430594,-121.432226,-121.433619,-121.434092,-121.434292,-121.436117,-121.438209,-121.440396,-121.445025,-121.446173,-121.450939,-121.449926,-121.448915,-121.447766,-121.447462,-121.447352],"lat":[38.625395,38.632864,38.633052,38.633022,38.633002,38.632925,38.627438,38.624402,38.623796,38.623545,38.621237,38.619366,38.617456,38.613837,38.614743,38.61478,38.615622,38.616579,38.618647,38.620041,38.625395]}]],[[{"lng":[-121.507514,-121.504235,-121.495643,-121.493401,-121.493267,-121.495108,-121.507365,-121.50743,-121.507514],"lat":[38.670816,38.670816,38.670808,38.670807,38.656229,38.656215,38.656041,38.659544,38.670816]}],[{"lng":[-121.477105,-121.474322,-121.47613,-121.476392,-121.484065,-121.485358,-121.48408,-121.484108,-121.484125,-121.479361,-121.478817,-121.478377,-121.477105],"lat":[38.669996,38.655347,38.655234,38.656552,38.656303,38.660541,38.663852,38.670924,38.674059,38.67194,38.671542,38.670926,38.669996]}]],[[{"lng":[-121.486996,-121.474761,-121.476192,-121.477613,-121.484264,-121.489851,-121.488659,-121.488423,-121.486996],"lat":[38.58835,38.585069,38.581802,38.57852,38.580295,38.581785,38.584524,38.585069,38.58835]}]],[[{"lng":[-121.533312,-121.529655,-121.526325,-121.532128,-121.535904,-121.539684,-121.539685,-121.539689,-121.544738,-121.546151,-121.548961,-121.54998,-121.552471,-121.558074,-121.56012,-121.557342,-121.550488,-121.539859,-121.533312],"lat":[38.655763,38.64913,38.643031,38.643043,38.642788,38.64303,38.642319,38.642114,38.643525,38.642574,38.643041,38.642076,38.643357,38.654971,38.656339,38.655554,38.655623,38.655716,38.655763]}]],[[{"lng":[-121.54032,-121.540188,-121.540174,-121.540138,-121.543754,-121.546811,-121.548885,-121.550099,-121.557886,-121.557901,-121.557876,-121.54032],"lat":[38.685506,38.674392,38.670797,38.667639,38.669899,38.670943,38.671278,38.671348,38.671362,38.678057,38.685603,38.685506]}]],[[{"lng":[-121.47613,-121.474322,-121.471691,-121.471523,-121.469519,-121.467588,-121.467737,-121.466832,-121.466966,-121.466355,-121.466821,-121.470249,-121.471668,-121.472905,-121.47451,-121.47555,-121.475561,-121.475536,-121.475537,-121.475527,-121.475538,-121.477782,-121.47781,-121.477773,-121.475488,-121.475885,-121.47516,-121.47613],"lat":[38.655234,38.655347,38.641512,38.640632,38.629945,38.61974,38.617387,38.611763,38.611176,38.611163,38.608395,38.600312,38.598439,38.597893,38.59968,38.604366,38.611243,38.617264,38.617601,38.629652,38.635261,38.638599,38.640786,38.641962,38.645825,38.65058,38.650614,38.655234]}]],[[{"lng":[-121.445025,-121.440396,-121.438209,-121.436117,-121.434292,-121.434092,-121.433619,-121.432226,-121.430594,-121.419931,-121.412988,-121.411775,-121.413082,-121.419808,-121.42271,-121.435403,-121.438118,-121.443877,-121.443882,-121.445025],"lat":[38.613837,38.617456,38.619366,38.621237,38.623545,38.623796,38.624402,38.627438,38.632925,38.632739,38.632476,38.63264,38.631453,38.62523,38.622536,38.610821,38.611064,38.611121,38.612934,38.613837]}]],[[{"lng":[-121.471691,-121.468121,-121.45692,-121.449837,-121.447756,-121.447787,-121.447311,-121.447364,-121.447346,-121.447352,-121.452146,-121.456943,-121.459016,-121.463687,-121.467588,-121.469519,-121.471523,-121.471691],"lat":[38.641512,38.641828,38.641524,38.641325,38.641433,38.64032,38.640309,38.636693,38.632864,38.625395,38.625524,38.623939,38.62402,38.62208,38.61974,38.629945,38.640632,38.641512]}]],[[{"lng":[-121.497895,-121.494188,-121.487318,-121.487518,-121.482819,-121.479375,-121.475655,-121.464188,-121.462307,-121.46165,-121.454642,-121.450311,-121.449046,-121.447553,-121.446811,-121.446736,-121.446708,-121.446674,-121.450894,-121.457212,-121.458177,-121.460472,-121.461038,-121.461509,-121.46121,-121.461641,-121.462838,-121.463384,-121.463947,-121.464419,-121.468527,-121.474382,-121.475089,-121.476637,-121.477304,-121.477774,-121.477652,-121.477592,-121.478055,-121.478919,-121.479024,-121.482534,-121.482533,-121.482515,-121.49307,-121.494642,-121.497631,-121.497895],"lat":[38.468173,38.468872,38.470961,38.472514,38.472451,38.473379,38.474287,38.474487,38.474471,38.474516,38.474316,38.474295,38.470966,38.467031,38.464752,38.464065,38.463681,38.463235,38.465189,38.46227,38.462018,38.461576,38.463443,38.463569,38.461445,38.461424,38.460507,38.460057,38.459799,38.459782,38.459778,38.459819,38.459734,38.459041,38.458698,38.458409,38.457871,38.457583,38.457379,38.456301,38.456298,38.456216,38.456278,38.457819,38.456427,38.458525,38.467597,38.468173]}]],[[{"lng":[-121.418969,-121.419924,-121.422988,-121.427406,-121.42085,-121.418969],"lat":[38.474339,38.474347,38.474281,38.478442,38.478378,38.474339]}]],[[{"lng":[-121.445746,-121.445559,-121.44259,-121.444403,-121.449409,-121.450325,-121.452774,-121.452801,-121.455717,-121.452819,-121.453622,-121.455255,-121.453327,-121.445746],"lat":[38.531788,38.531392,38.524974,38.524958,38.524923,38.524922,38.524919,38.528955,38.530594,38.53138,38.53138,38.533697,38.532294,38.531788]}]],[[{"lng":[-121.506206,-121.505036,-121.503631,-121.503146,-121.493768,-121.49536,-121.495899,-121.497206,-121.49842,-121.498601,-121.505773,-121.512474,-121.513413,-121.512824,-121.513276,-121.511248,-121.506206],"lat":[38.538912,38.538386,38.537467,38.537145,38.535148,38.530214,38.528629,38.524723,38.520982,38.520429,38.521612,38.521249,38.521419,38.522729,38.528227,38.531392,38.538912]}]],[[{"lng":[-121.469039,-121.465064,-121.460532,-121.459803,-121.456384,-121.45384,-121.4578,-121.460101,-121.461677,-121.46472,-121.466562,-121.470459,-121.472436,-121.471151,-121.469039],"lat":[38.563946,38.562835,38.557484,38.556701,38.553198,38.546757,38.546748,38.546789,38.546801,38.547184,38.549185,38.553856,38.556125,38.559082,38.563946]}]],[[{"lng":[-121.450311,-121.446594,-121.436875,-121.435429,-121.434755,-121.431261,-121.433327,-121.435315,-121.436125,-121.442399,-121.445968,-121.447553,-121.449046,-121.450311],"lat":[38.474295,38.474276,38.474254,38.471555,38.470296,38.463781,38.463588,38.464581,38.465082,38.466434,38.467142,38.467031,38.470966,38.474295]}]],[[{"lng":[-121.435403,-121.42271,-121.419808,-121.41991,-121.419381,-121.418137,-121.416419,-121.415281,-121.415282,-121.419687,-121.419697,-121.419919,-121.420403,-121.419874,-121.41993,-121.420775,-121.426878,-121.432337,-121.435403],"lat":[38.610821,38.622536,38.62523,38.623023,38.623247,38.623894,38.624506,38.624541,38.618168,38.620097,38.618605,38.617849,38.617051,38.615506,38.610926,38.611079,38.611193,38.611115,38.610821]}]],[[{"lng":[-121.497899,-121.493242,-121.492895,-121.48847,-121.484272,-121.484191,-121.487326,-121.487986,-121.488412,-121.489875,-121.491342,-121.493202,-121.495434,-121.4963,-121.497892,-121.497899],"lat":[38.62742,38.627445,38.627447,38.627488,38.627519,38.622421,38.622501,38.622717,38.622764,38.622886,38.622681,38.622711,38.621851,38.621411,38.621255,38.62742]}]],[[{"lng":[-121.503123,-121.497895,-121.497631,-121.494642,-121.499691,-121.501149,-121.501777,-121.500622,-121.501412,-121.503123],"lat":[38.467189,38.468173,38.467597,38.458525,38.45772,38.459137,38.461522,38.461697,38.465777,38.467189]}]],[[{"lng":[-121.518299,-121.514188,-121.507747,-121.507317,-121.507408,-121.507365,-121.495108,-121.493267,-121.49323,-121.502702,-121.502559,-121.493233,-121.493202,-121.500517,-121.512483,-121.517356,-121.518299],"lat":[38.627277,38.627773,38.632365,38.63955,38.64693,38.656041,38.656215,38.656229,38.649433,38.649312,38.641997,38.641979,38.637727,38.633878,38.627592,38.625027,38.627277]}]],[[{"lng":[-121.52466,-121.523996,-121.521182,-121.516841,-121.512704,-121.509925,-121.507976,-121.507737,-121.507774,-121.507514,-121.50743,-121.507365,-121.521981,-121.525475,-121.52606,-121.526922,-121.532414,-121.533966,-121.531207,-121.525362,-121.52466],"lat":[38.666446,38.66645,38.665996,38.666305,38.665648,38.665646,38.665335,38.66654,38.670816,38.670816,38.659544,38.656041,38.655891,38.655851,38.658395,38.659391,38.662067,38.664898,38.666171,38.666423,38.666446]}]],[[{"lng":[-121.548406,-121.544963,-121.540055,-121.533003,-121.524735,-121.524051,-121.519641,-121.51877,-121.517995,-121.517562,-121.516198,-121.514438,-121.510714,-121.508848,-121.50423,-121.503221,-121.504135,-121.50509,-121.506085,-121.508867,-121.511882,-121.517324,-121.521449,-121.529337,-121.535405,-121.538584,-121.542075,-121.544792,-121.547328,-121.548406],"lat":[38.482515,38.483903,38.479973,38.47693,38.476432,38.476609,38.478819,38.479267,38.479809,38.480046,38.48052,38.481119,38.4812,38.478669,38.475109,38.469744,38.469692,38.469563,38.470948,38.472809,38.473673,38.474501,38.474398,38.47387,38.473952,38.47457,38.47611,38.478193,38.481289,38.482515]}]],[[{"lng":[-121.475758,-121.473931,-121.471217,-121.467459,-121.464767,-121.46476,-121.464713,-121.4647,-121.464626,-121.465853,-121.470148,-121.471747,-121.473437,-121.47396,-121.475758],"lat":[38.541064,38.541158,38.541213,38.541139,38.541193,38.539527,38.533606,38.532137,38.524951,38.524943,38.524903,38.52489,38.53159,38.533665,38.541064]}]],[[{"lng":[-121.409124,-121.390694,-121.370294,-121.36842,-121.368395,-121.367415,-121.367361,-121.369864,-121.369894,-121.367904,-121.367905,-121.362772,-121.36274,-121.36994,-121.369944,-121.371806,-121.372196,-121.375927,-121.379054,-121.379024,-121.390551,-121.390557,-121.395159,-121.395278,-121.394583,-121.399497,-121.409042,-121.409081,-121.409108,-121.409124],"lat":[38.525063,38.525179,38.525245,38.525253,38.523935,38.523956,38.522097,38.522089,38.519235,38.519238,38.518137,38.518149,38.513947,38.513932,38.511372,38.507363,38.503696,38.503664,38.504302,38.503615,38.503863,38.501098,38.501104,38.503236,38.503395,38.503698,38.503683,38.510551,38.516156,38.525063]}]],[[{"lng":[-121.484323,-121.478938,-121.475758,-121.47396,-121.473437,-121.471747,-121.47972,-121.480952,-121.481879,-121.484323],"lat":[38.539613,38.54054,38.541064,38.533665,38.53159,38.52489,38.524835,38.528677,38.531701,38.539613]}]],[[{"lng":[-121.486996,-121.488423,-121.488659,-121.489851,-121.49651,-121.495089,-121.494595,-121.489507,-121.486996],"lat":[38.58835,38.585069,38.584524,38.581785,38.58356,38.586846,38.587802,38.588587,38.58835]}]],[[{"lng":[-121.365164,-121.366336,-121.367674,-121.369613,-121.375283,-121.382941,-121.383201,-121.382008,-121.379344,-121.376914,-121.372431,-121.370274,-121.365164],"lat":[38.64377,38.642852,38.642272,38.641689,38.639978,38.637155,38.63989,38.64102,38.640648,38.642592,38.64457,38.643607,38.64377]}]],[[{"lng":[-121.421645,-121.409349,-121.393898,-121.393634,-121.396468,-121.402412,-121.403315,-121.403344,-121.400017,-121.399414,-121.39821,-121.398988,-121.398728,-121.390885,-121.386309,-121.370952,-121.371,-121.370354,-121.370294,-121.390694,-121.409124,-121.409149,-121.410133,-121.413173,-121.414636,-121.416577,-121.421645],"lat":[38.55102,38.55381,38.557248,38.555656,38.555903,38.554556,38.55358,38.550712,38.550726,38.550863,38.549008,38.54707,38.546637,38.544396,38.543097,38.538759,38.535138,38.52966,38.525245,38.525179,38.525063,38.52927,38.532081,38.539564,38.542009,38.544644,38.55102]}]],[[{"lng":[-121.446674,-121.446481,-121.446238,-121.452064,-121.45255,-121.458177,-121.457212,-121.450894,-121.446674],"lat":[38.463235,38.46069,38.458975,38.459208,38.460345,38.462018,38.46227,38.465189,38.463235]}],[{"lng":[-121.445909,-121.445585,-121.448675,-121.448665,-121.454348,-121.453029,-121.445909],"lat":[38.457307,38.455679,38.45558,38.452573,38.452311,38.457611,38.457307]}]],[[{"lng":[-121.507423,-121.50751,-121.507582,-121.510475,-121.512404,-121.51947,-121.519751,-121.519455,-121.524423,-121.527606,-121.529871,-121.531309,-121.531414,-121.531185,-121.529841,-121.525731,-121.524462,-121.52466,-121.525362,-121.531207,-121.533966,-121.532414,-121.526922,-121.52606,-121.525475,-121.532897,-121.538214,-121.540138,-121.540174,-121.540188,-121.54032,-121.531261,-121.51603,-121.507423],"lat":[38.685217,38.68193,38.676903,38.676892,38.677036,38.675594,38.676905,38.677921,38.679874,38.680004,38.680005,38.680081,38.677607,38.677031,38.674456,38.673443,38.671625,38.666446,38.666423,38.666171,38.664898,38.662067,38.659391,38.658395,38.655851,38.655768,38.665393,38.667639,38.670797,38.674392,38.685506,38.685505,38.685199,38.685217]}]],[[{"lng":[-121.531396,-121.5255,-121.524199,-121.525177,-121.521849,-121.522148,-121.522311,-121.521998,-121.518264,-121.517767,-121.517292,-121.516668,-121.521602,-121.52355,-121.528691,-121.527225,-121.525988,-121.52624,-121.52724,-121.527527,-121.530364,-121.530532,-121.531237,-121.531396],"lat":[38.51727,38.518777,38.52151,38.522682,38.523163,38.51617,38.510844,38.509266,38.502795,38.501317,38.497641,38.495476,38.49558,38.495906,38.496955,38.501074,38.503413,38.505034,38.506756,38.506687,38.513908,38.513878,38.515831,38.51727]}]],[[{"lng":[-121.446882,-121.441276,-121.43864,-121.435047,-121.430626,-121.419986,-121.420005,-121.419948,-121.420059,-121.422758,-121.425055,-121.430484,-121.432457,-121.434464,-121.440255,-121.443689,-121.447198,-121.449428,-121.45019,-121.450046,-121.449431,-121.448161,-121.446882],"lat":[38.600211,38.600252,38.60076,38.603622,38.601347,38.596079,38.588942,38.582004,38.581383,38.583368,38.583576,38.582927,38.581844,38.58157,38.58271,38.58455,38.58773,38.589771,38.590291,38.593839,38.596446,38.598732,38.600211]}],[{"lng":[-121.415338,-121.41534,-121.415295,-121.416662,-121.41874,-121.42012,-121.422592,-121.42155,-121.421335,-121.415988,-121.415338],"lat":[38.575348,38.57487,38.574112,38.573392,38.57096,38.569614,38.568914,38.575857,38.577284,38.575333,38.575348]}]],[[{"lng":[-121.446882,-121.440636,-121.435403,-121.432337,-121.426878,-121.420775,-121.41993,-121.419943,-121.419958,-121.419957,-121.419953,-121.419986,-121.430626,-121.435047,-121.43864,-121.441276,-121.446882],"lat":[38.600211,38.605988,38.610821,38.611115,38.611193,38.611079,38.610926,38.608628,38.603629,38.602349,38.596077,38.596079,38.601347,38.603622,38.60076,38.600252,38.600211]}]],[[{"lng":[-121.475844,-121.474412,-121.472436,-121.470459,-121.466562,-121.46472,-121.464716,-121.464767,-121.467459,-121.471217,-121.473931,-121.475758,-121.475812,-121.475785,-121.475844],"lat":[38.557203,38.556821,38.556125,38.553856,38.549185,38.547184,38.546275,38.541193,38.541139,38.541213,38.541158,38.541064,38.549935,38.553605,38.557203]}]],[[{"lng":[-121.477105,-121.466103,-121.456865,-121.456346,-121.453114,-121.447302,-121.429142,-121.417619,-121.415264,-121.415224,-121.421624,-121.421638,-121.429138,-121.438415,-121.440926,-121.443513,-121.447344,-121.447739,-121.456912,-121.465178,-121.46615,-121.467087,-121.474322,-121.477105],"lat":[38.669996,38.669899,38.669433,38.669116,38.669205,38.669199,38.669109,38.668995,38.668976,38.658164,38.656908,38.654524,38.654607,38.654708,38.654735,38.656463,38.654957,38.654827,38.654932,38.655024,38.655169,38.655334,38.655347,38.669996]}]],[[{"lng":[-121.543404,-121.542106,-121.540593,-121.53473,-121.534727,-121.53495,-121.535595,-121.537785,-121.538928,-121.544963,-121.548406,-121.555932,-121.55467,-121.552482,-121.548803,-121.543404],"lat":[38.496322,38.494158,38.493342,38.493559,38.491532,38.490927,38.489932,38.487636,38.486741,38.483903,38.482515,38.492849,38.493627,38.494536,38.495904,38.496322]}]],[[{"lng":[-121.506064,-121.503261,-121.491264,-121.491876,-121.491963,-121.49239,-121.49335,-121.494648,-121.496695,-121.505073,-121.506189,-121.506064],"lat":[38.495632,38.495679,38.495727,38.493927,38.489044,38.483828,38.481412,38.481396,38.481371,38.481269,38.489593,38.495632]}]],[[{"lng":[-121.434576,-121.433196,-121.432453,-121.432213,-121.436792,-121.43704,-121.436812,-121.436204,-121.434576],"lat":[38.507712,38.504782,38.503237,38.502725,38.503186,38.503212,38.507355,38.507556,38.507712]}]],[[{"lng":[-121.508668,-121.506278,-121.505114,-121.500517,-121.496637,-121.49294,-121.488586,-121.48553,-121.481284,-121.478961,-121.476653,-121.476403,-121.473469,-121.470373,-121.466335,-121.463762,-121.462489,-121.461248,-121.455336,-121.45028,-121.45019,-121.449428,-121.447198,-121.449058,-121.450604,-121.450906,-121.451607,-121.454744,-121.458301,-121.461522,-121.465208,-121.468138,-121.474761,-121.486996,-121.489507,-121.494595,-121.495089,-121.49651,-121.498304,-121.501314,-121.506275,-121.507241,-121.508668],"lat":[38.596803,38.59976,38.600503,38.601902,38.602501,38.602618,38.602401,38.601977,38.600445,38.598938,38.59636,38.596121,38.592956,38.590683,38.588867,38.588618,38.58877,38.589159,38.590591,38.59032,38.590291,38.589771,38.58773,38.58606,38.585437,38.585382,38.583857,38.581998,38.58134,38.58197,38.583056,38.583443,38.585069,38.58835,38.588587,38.587802,38.586846,38.58356,38.582818,38.58362,38.586288,38.591151,38.596803]}]],[[{"lng":[-121.532897,-121.525475,-121.521981,-121.507365,-121.507408,-121.507317,-121.507747,-121.514188,-121.518299,-121.518508,-121.520785,-121.526325,-121.529655,-121.533312,-121.532897],"lat":[38.655768,38.655851,38.655891,38.656041,38.64693,38.63955,38.632365,38.627773,38.627277,38.627774,38.632763,38.643031,38.64913,38.655763,38.655768]}]],[[{"lng":[-121.409042,-121.409037,-121.408979,-121.415351,-121.415553,-121.416152,-121.41621,-121.416685,-121.416679,-121.412333,-121.409042],"lat":[38.503683,38.503334,38.499678,38.499648,38.501233,38.501428,38.503243,38.503241,38.503744,38.50377,38.503683]}]],[[{"lng":[-121.49842,-121.497206,-121.495899,-121.49431,-121.48625,-121.480952,-121.47972,-121.478672,-121.483144,-121.487431,-121.498017,-121.49842],"lat":[38.520982,38.524723,38.528629,38.528351,38.528304,38.528677,38.524835,38.521312,38.521114,38.52097,38.520716,38.520982]}]],[[{"lng":[-121.422592,-121.42012,-121.41874,-121.416662,-121.415295,-121.414673,-121.410224,-121.40928,-121.409035,-121.40921,-121.40926,-121.412425,-121.417224,-121.419995,-121.422665,-121.422592],"lat":[38.568914,38.569614,38.57096,38.573392,38.574112,38.572955,38.570824,38.56993,38.565364,38.562301,38.560718,38.560243,38.560357,38.562441,38.568388,38.568914]}]],[[{"lng":[-121.533623,-121.527868,-121.522905,-121.52155,-121.520055,-121.518501,-121.51766,-121.517543,-121.516198,-121.517562,-121.517995,-121.51877,-121.519641,-121.524051,-121.524735,-121.533003,-121.540055,-121.536971,-121.535153,-121.534336,-121.533623],"lat":[38.48465,38.483515,38.483839,38.483341,38.483203,38.483723,38.483768,38.482763,38.48052,38.480046,38.479809,38.479267,38.478819,38.476609,38.476432,38.47693,38.479973,38.481283,38.482466,38.483155,38.48465]}]],[[{"lng":[-121.480001,-121.481435,-121.482384,-121.487703,-121.494629,-121.493672,-121.492716,-121.488474,-121.486653,-121.485321,-121.480001],"lat":[38.57301,38.5697,38.567506,38.568923,38.570769,38.572962,38.575156,38.574026,38.574785,38.57443,38.57301]}]],[[{"lng":[-121.54548,-121.542764,-121.541951,-121.537911,-121.535364,-121.534872,-121.534985,-121.544121,-121.543404,-121.548803,-121.552482,-121.55467,-121.555932,-121.558271,-121.558799,-121.559036,-121.558701,-121.558676,-121.55855,-121.554985,-121.553569,-121.551782,-121.54548],"lat":[38.515615,38.510062,38.508499,38.504223,38.499284,38.498697,38.498198,38.497596,38.496322,38.495904,38.494536,38.493627,38.492849,38.496356,38.497479,38.499592,38.501939,38.50214,38.503174,38.510164,38.512305,38.513861,38.515615]}]],[[{"lng":[-121.428525,-121.412655,-121.411296,-121.41065,-121.408608,-121.404018,-121.403204,-121.403601,-121.403482,-121.404685,-121.41776,-121.421029,-121.422177,-121.422777,-121.425617,-121.425795,-121.425223,-121.425872,-121.427137,-121.42768,-121.428415,-121.428525],"lat":[38.458623,38.457959,38.458132,38.456781,38.452612,38.442659,38.440041,38.439707,38.437894,38.438018,38.438,38.43803,38.438333,38.438677,38.442351,38.445202,38.448124,38.449722,38.452378,38.454836,38.458311,38.458623]}]]],["X6067007104","X6067000400","X6067006900","X6067003102","X6067006400","X6067001101","X6067009633","X6067001900","X6067009614","X6067004906","X6067007105","X6067003101","X6067007011","X6067004202","X6067007007","X6067004300","X6067006500","X6067000200","X6067003202","X6067004009","X6067009610","X6067004903","X6067007014","X6067006701","X6067002300","X6067001700","X6067003800","X6067003400","X6067000700","X6067004702","X6067003501","X6067003203","X6067005404","X6067000800","X6067001400","X6067003204","X6067004905","X6067007010","X6067001600","X6067003000","X6067000300","X6067004502","X6067002800","X6067002500","X6067005204","X6067004100","X6067002200","X6067000100","X6067007413","X6067007107","X6067004001","X6067007004","X6067004601","X6067004801","X6067002900","X6067004010","X6067004005","X6067002400","X6067004402","X6067002100","X6067009606","X6067007012","X6067002600","X6067002000","X6067007016","X6067007020","X6067005202","X6067006800","X6067007017","X6067004904","X6067004501","X6067004203","X6067009609","X6067001300","X6067001500","X6067003900","X6067005201","X6067006600","X6067007106","X6067000500","X6067007015","X6067007101","X6067007001","X6067006300","X6067006702","X6067009601","X6067005002","X6067004401","X6067003300","X6067001800","X6067009634","X6067006202","X6067007013","X6067009900","X6067007019","X6067007103","X6067004012","X6067003700","X6067009201","X6067003600","X6067000600","X6067007504","X6067005205","X6067009618","X6067007102","X6067004006","X6067005402","X6067005502","X6067002700","X6067007204","X6067004008","X6067004201","X6067004701","X6067005301","X6067007018","X6067004802","X6067003502","X6067005403","X6067004011","X6067001200","X6067004004","X6067009608"],"saccitytracts",{"interactive":true,"className":"","pane":"tmap401","stroke":false,"color":"#666666","weight":1,"opacity":0,"fill":true,"fillColor":["#FB795A","#BB1419","#ED392B","#FB795A","#FB795A","#ED392B","#FEE4D8","#BB1419","#FCB499","#FEE4D8","#FB795A","#FB795A","#FCB499","#FEE4D8","#FCB499","#FEE4D8","#FEE4D8","#BB1419","#FCB499","#FB795A","#FCB499","#FEE4D8","#FCB499","#FCB499","#BB1419","#BB1419","#FEE4D8","#ED392B","#FB795A","#FEE4D8","#ED392B","#FEE4D8","#BB1419","#BB1419","#BB1419","#FEE4D8","#FEE4D8","#ED392B","#BB1419","#ED392B","#BB1419","#FEE4D8","#FCB499","#BB1419","#BB1419","#FEE4D8","#FB795A","#BB1419","#ED392B","#FB795A","#FB795A","#FCB499","#FEE4D8","#FCB499","#BB1419","#FB795A","#ED392B","#BB1419","#FEE4D8","#ED392B","#FCB499","#FB795A","#BB1419","#ED392B","#FCB499","#ED392B","#ED392B","#FEE4D8","#FB795A","#FCB499","#FEE4D8","#FEE4D8","#FCB499","#BB1419","#BB1419","#ED392B","#FCB499","#FCB499","#FB795A","#BB1419","#FB795A","#BB1419","#FCB499","#FB795A","#FCB499","#FEE4D8","#FCB499","#FCB499","#BB1419","#ED392B","#FEE4D8","#FB795A","#FCB499","#ED392B","#FEE4D8","#FB795A","#FB795A","#FEE4D8","#FB795A","#FB795A","#BB1419","#BB1419","#ED392B","#FCB499","#ED392B","#FB795A","#ED392B","#FB795A","#ED392B","#ED392B","#ED392B","#FEE4D8","#FEE4D8","#FB795A","#FCB499","#FEE4D8","#ED392B","#BB1419","#ED392B","#BB1419","#ED392B","#FCB499"],"fillOpacity":[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],"dashArray":"none","smoothFactor":1,"noClip":false},["<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007104<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3558<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067000400<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6313<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067006900<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3831<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003102<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3189<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067006400<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2853<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067001101<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.5075<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009633<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1310<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067001900<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6443<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009614<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1993<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004906<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1313<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007105<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3055<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003101<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2955<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007011<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2064<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004202<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1267<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007007<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1878<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004300<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.0737<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067006500<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1535<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067000200<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.7459<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003202<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2154<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004009<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3039<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009610<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1939<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004903<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1228<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007014<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2574<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067006701<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1701<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002300<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6983<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067001700<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6672<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003800<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1519<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003400<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4449<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067000700<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2828<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004702<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1235<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003501<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4697<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003203<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.0806<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005404<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.7252<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067000800<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6988<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067001400<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6753<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003204<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.0852<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004905<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1691<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007010<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4208<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067001600<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6643<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003000<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3846<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067000300<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.7557<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004502<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1210<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002800<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2276<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002500<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.7396<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005204<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6430<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004100<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1279<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002200<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3249<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067000100<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.8254<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007413<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.5212<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007107<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3603<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004001<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2857<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007004<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1823<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004601<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1502<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004801<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1769<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002900<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6673<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004010<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2978<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004005<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4023<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002400<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.7269<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004402<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1341<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002100<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4580<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009606<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1886<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007012<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3436<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002600<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6364<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002000<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3905<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007016<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2684<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007020<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4952<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005202<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.5335<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067006800<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1398<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007017<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2995<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004904<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2277<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004501<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.0869<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004203<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.0715<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009609<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1793<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067001300<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6320<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067001500<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.7618<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003900<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4763<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005201<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2695<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067006600<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2623<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007106<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3037<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067000500<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.5354<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007015<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3128<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007101<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>1.0000<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007001<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1998<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067006300<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3284<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067006702<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1777<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009601<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.0694<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005002<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2082<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004401<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2115<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003300<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.5800<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067001800<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4942<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009634<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.0563<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067006202<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3370<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007013<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1861<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009900<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4133<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007019<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.0835<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007103<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3453<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004012<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3363<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003700<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1630<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009201<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3236<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003600<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3412<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067000600<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.5592<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007504<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6785<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005205<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3954<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009618<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2281<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007102<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3940<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004006<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3588<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005402<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4602<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005502<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2900<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067002700<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3740<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007204<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.3630<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004008<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4386<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004201<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1388<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004701<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1555<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005301<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2860<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067007018<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.2733<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004802<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1244<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067003502<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4314<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067005403<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6025<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004011<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4079<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067001200<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.6442<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067004004<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.4155<\/nobr><\/td><\/tr><\/table><\/div>","<style> div.leaflet-popup-content {width:auto !important;overflow-y:auto; overflow-x:hidden;}<\/style><div style=\"max-height:25em;padding-right:0px;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>6067009608<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\"><nobr>pwh<\/nobr><\/td><td align=\"right\"><nobr>0.1759<\/nobr><\/td><\/tr><\/table><\/div>"],null,["6067007104","6067000400","6067006900","6067003102","6067006400","6067001101","6067009633","6067001900","6067009614","6067004906","6067007105","6067003101","6067007011","6067004202","6067007007","6067004300","6067006500","6067000200","6067003202","6067004009","6067009610","6067004903","6067007014","6067006701","6067002300","6067001700","6067003800","6067003400","6067000700","6067004702","6067003501","6067003203","6067005404","6067000800","6067001400","6067003204","6067004905","6067007010","6067001600","6067003000","6067000300","6067004502","6067002800","6067002500","6067005204","6067004100","6067002200","6067000100","6067007413","6067007107","6067004001","6067007004","6067004601","6067004801","6067002900","6067004010","6067004005","6067002400","6067004402","6067002100","6067009606","6067007012","6067002600","6067002000","6067007016","6067007020","6067005202","6067006800","6067007017","6067004904","6067004501","6067004203","6067009609","6067001300","6067001500","6067003900","6067005201","6067006600","6067007106","6067000500","6067007015","6067007101","6067007001","6067006300","6067006702","6067009601","6067005002","6067004401","6067003300","6067001800","6067009634","6067006202","6067007013","6067009900","6067007019","6067007103","6067004012","6067003700","6067009201","6067003600","6067000600","6067007504","6067005205","6067009618","6067007102","6067004006","6067005402","6067005502","6067002700","6067007204","6067004008","6067004201","6067004701","6067005301","6067007018","6067004802","6067003502","6067005403","6067004011","6067001200","6067004004","6067009608"],{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]},{"method":"addLegend","args":[{"colors":["#FEE4D8","#FCB499","#FB795A","#ED392B","#BB1419"],"labels":["0.056 to 0.169","0.169 to 0.277","0.277 to 0.362","0.362 to 0.535","0.535 to 1.000"],"na_color":null,"na_label":"NA","opacity":1,"position":"topright","type":"unknown","title":"Percent white","extra":null,"layerId":"legend401","className":"info legend saccitytracts","group":"saccitytracts"}]},{"method":"addLayersControl","args":[["Esri.WorldGrayCanvas","OpenStreetMap","Esri.WorldTopoMap"],"saccitytracts",{"collapsed":true,"autoZIndex":true,"position":"topleft"}]}],"limits":{"lat":[38.437894,38.685603],"lng":[-121.56012,-121.36274]},"fitBounds":[38.437894,-121.56012,38.685603,-121.36274,[]]},"evals":[],"jsHooks":[]}</script>
```

Zoom in and out to find out where percent white is high and low.  To switch back to plotting mode (static), type in 


```r
tmap_mode("plot")
```

```
## tmap mode set to plotting
```


Behind the scenes, the package [leaflet](https://rstudio.github.io/leaflet/) is used for generating the interactive map

<div style="margin-bottom:25px;">
</div>
#### **Dot map**
\

We can create a dot or pin map using `tm_dots()`


```r
tm_shape(saccitytracts) +
  tm_borders()  +
tm_shape(evcharge) +
  tm_dots(col = "red", title="Environmental Exposure Type") +
    tm_layout(main.title = "EV Charge Stations Sacramento", 
            main.title.size = 1, main.title.position="center")
```

![](introspatial_files/figure-html/unnamed-chunk-54-1.png)<!-- -->

Overlay it on top of polygons shaded by percent white. 


```r
tm_shape(saccitytracts, unit = "mi") +
  tm_polygons(col = "pwh", style = "quantile",palette = "Reds",
              title = "Percent white") +
  tm_shape(evcharge) +
  tm_dots(col = "black") +
    tm_scale_bar(breaks = c(0, 1, 2), text.size = 1, position = c("left", "bottom")) +
      tm_compass(type = "4star", position = c("right", "bottom"))  +
  tm_layout(main.title = "EV Charge Stations Sacramento", 
            main.title.size = 1.25, main.title.position="center",
            legend.outside = TRUE, legend.outside.position = "right",
            frame = FALSE)
```

![](introspatial_files/figure-html/unnamed-chunk-55-1.png)<!-- -->

<div style="margin-bottom:25px;">
</div>
#### **Color patch map**
\

OSU page 72 discusses color patch maps, which is a map of polygons based on a categorical variable.  To do this, use `style = "cat"`.  Let's map the categorical variable *medinc*, which has categories "High", "Medium" and "Low".


```r
tm_shape(saccitytracts) +
  tm_polygons("medinc",style="cat",palette="Paired")  +
  tm_layout(main.title = "Median Income Sacramento", 
            main.title.size = 1, main.title.position="left",
            legend.outside = TRUE, legend.outside.position = "right")
```

![](introspatial_files/figure-html/unnamed-chunk-56-1.png)<!-- -->

<div style="margin-bottom:25px;">
</div>
#### **Cartogram**
\

In R, a useful implementation of different types of cartograms is included in the package *cartogram*. Specifically, this supports the area cartograms described in OSU page 78 using `cartogram_cont()`. The result of these functions is a simple features layer, which can then be mapped by means of the usual **tmap** commands.

For example,  we take the *pwh* variable to construct an area cartogram. First, we need to add a [projection](https://pro.arcgis.com/en/pro-app/2.7/help/mapping/properties/coordinate-systems-and-projections.htm) to *saccitytracts*.  We do this by using `st_transform()`, and we use the [UTM projection](https://gisgeography.com/utm-universal-transverse-mercator-projection/).


```r
saccitytracts <-st_transform(saccitytracts, 
                              crs = "+proj=utm +zone=10 +datum=NAD83 +ellps=GRS80") 
```

Next, we pass the layer (*saccitytracts*) and the variable to the `cartogram_cont` function. We check that the result is an **sf** object.


```r
carto.cont <- cartogram_cont(saccitytracts,"pwh")
class(carto.cont)
```

Which we can map


```r
tm_shape(carto.cont) +
  tm_fill("pwh") +
  tm_borders()
```

![](introspatial_files/figure-html/unnamed-chunk-59-1.png)<!-- -->
  


<div style="margin-bottom:25px;">
</div>
## **Writing spatial data**
\                

Earlier you learned how to read in spatial data. We end today's lab by learning how to write spatial data.  The function is `st_write()`.

Let's save the Sacramento tract **sf** object *saccitytracts* as a shapefile `.shp`.



```r
st_write(saccitytracts, "Sacramento_Tract_pwhite.shp", delete_layer = TRUE)
```


The `delete_layer = TRUE` argument tells R to overwrite the file *Sacramento_Tract_pwhite.shp* if it already exists in your current directory.  You should see the files associated with *Sacramento_Tract_pwhite* in your current working directory.


You've completed your introduction to **sf**. Whew! Badge? Yes, please, you earned it!  Time to [celebrate](https://www.youtube.com/watch?v=3GwjfUFyY6M)!

<center>
![](/Users/noli/Documents/UCD/teaching/GEO 200CN/lab/geo200cn.github.io/sf.gif){ width=25% }

</center>

Answer the assignment 4 questions that are uploaded on Canvas. Submit an R Markdown and its knitted document on Canvas by the assignment due date.

<div style="margin-bottom:25px;">
</div>
## **Resources**
\

This is not a **G**eographic **I**nformation **S**cience class, so we did not touch on the many tools that **sf** offers for manipulating and managing your spatial data.  The best references for **sf** and, more broadly, cleaning, managing and analyzing spatial data in R are [Geocomputation with R](https://geocompr.robinlovelace.net/) (GWR) and [Spatial Data Science](https://keen-swartz-3146c4.netlify.com/) (SDS).  The SDS book is not complete, but what is there is really good (albeit a bit more complex in terms of the coding). Both are freely available through the links provided.  

For more on using `ggplot()` to map, check out [this](https://ryanpeek.org/2017-11-05-mapping-with-sf-Part-2/) and [this](https://www.r-spatial.org/r/2018/10/25/ggplot2-sf.html).  Some of this lab guide is adapted from Robert Hijman's [rspatial](https://rspatial.org/raster/index.html) website. Check out his guide for more information on the **sp** package.

***

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.


Website created and maintained by [Noli Brazil](https://nbrazil.faculty.ucdavis.edu/)
