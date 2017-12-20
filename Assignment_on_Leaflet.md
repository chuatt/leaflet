---
title: "Assignment: R Markdown and Leaflet"
author: "Tong Tat"
date: "December 20, 2017"
output: 
  html_document:
    keep_md: true
    highlight: tango
    theme: yeti
---

**Rpub Link:** [Click Here](http://rpubs.com/chuatt/343300)


# 1. Introduction

This is an document is for Coursera Developing Data Product assignment. The objective is to create a webpage using R Markdown that features a map created with Leaflet.

The dataset is coming from the **zipcode** package. This package contains a database of city, state, latitude and longitude information for US ZIP codes from CivicSpace Database (August 2004) augmented by Daniel Coven's
federalgovernmentzipcodes.us website (updated January 22,2012).


# 2. Loading Dataset


```r
# Clear cache first
rm(list=ls())

# Check for missing dependencies and load necessary R packages
if(!require(zipcode)){install.packages('zipcode')}; library(zipcode)
if(!require(leaflet)){install.packages('leaflet')}; library(leaflet)
if(!require(dplyr)){install.packages('dplyr')}; library(dplyr)
if(!require(ggplot2)){install.packages('ggplot2')}; library(ggplot2)

# Load dataset
data(zipcode)
```


# 3. Data Exploration

Now, lets take a look at the zipcode dataset.

```r
summary(zipcode)
```

```
##      zip                city              state              latitude     
##  Length:44336       Length:44336       Length:44336       Min.   :-44.25  
##  Class :character   Class :character   Class :character   1st Qu.: 34.96  
##  Mode  :character   Mode  :character   Mode  :character   Median : 39.10  
##                                                           Mean   : 38.47  
##                                                           3rd Qu.: 41.86  
##                                                           Max.   : 71.30  
##                                                           NA's   :647     
##    longitude      
##  Min.   :-176.64  
##  1st Qu.: -97.28  
##  Median : -87.82  
##  Mean   : -90.84  
##  3rd Qu.: -80.06  
##  Max.   : 171.18  
##  NA's   :647
```

Interesting, looks like it contains all the zipcodes from all the cities across all the different states, along with their latitude & longitude information. There seemed to be some NA values inside the latitude and longitude columns so I'm removing it now.


```r
zipcode2 <-na.omit(zipcode)
```

# 4. Data Transformation

Using dplyr package, we'll summarise the data into city, latitude, longitude and number of zipcodes per state.
Latitude and Longitude will be computed as the average of each state. Renaming latitude as **lat** and longitude as **lng** so that leaflet can consume later.


```r
zipcode3 <- zipcode2 %>% group_by(state) %>% summarise(lat=mean(latitude), lng=mean(longitude), count_zipcode=n())
```

Now, take a quick look at the transformed dataset.


```r
summary(zipcode3)
```

```
##     state                lat              lng          count_zipcode   
##  Length:60          Min.   :-44.25   Min.   :-170.77   Min.   :   1.0  
##  Class :character   1st Qu.: 32.96   1st Qu.: -99.27   1st Qu.: 293.0  
##  Mode  :character   Median : 38.71   Median : -86.37   Median : 609.5  
##                     Mean   : 34.16   Mean   : -71.43   Mean   : 728.1  
##                     3rd Qu.: 42.20   3rd Qu.: -74.92   3rd Qu.:1035.5  
##                     Max.   : 61.48   Max.   : 169.46   Max.   :2768.0
```

```r
View(zipcode3)
```


Noticed there are some states with very few zipcodes and their latitude and longitude are way too off, these are likely to be errors in the original dataset. So I'm filtering states that has latitude between 28 & 50.


```r
zipcode4 <- subset(zipcode3, lat>28 & lat<50)
```


# 5. Data Visualization

Alright, let's apply the leaflet package and plot the markers. Size of the markers will indicate the number zipcode in that state.


```r
zipcode4 %>% leaflet() %>% addTiles() %>% addCircles(weight=1, radius=zipcode4$count_zipcode*150)
```

<!--html_preserve--><div id="htmlwidget-aa7a11b4a6fe3c399d3e" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-aa7a11b4a6fe3c399d3e">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["//{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addCircles","args":[[32.8586747491329,35.1109913748309,33.5824816962457,36.1345112847802,39.250712384058,41.5552841022222,38.8951914463087,39.2874733398058,28.1489504167742,32.993620183908,42.0293569550459,44.5295802366864,40.4306352245763,39.9268366952748,38.5157170770186,37.6143418647343,30.9164080708661,42.2340320893782,39.05388271406,44.5807254435946,43.4531259348185,45.5634924981061,38.3847942596549,32.8218552141561,46.9392556235012,35.5721267896613,47.5112979280742,41.1945510516432,43.3921728354839,40.3901586740838,34.6682325045045,37.8830032617187,42.1949357541557,40.3854032564784,35.5163251770574,44.53311909,40.6231097835593,41.7033563297872,33.9626825906643,44.3051733287037,35.7945040537241,31.2715200740607,39.9288439944598,37.7250879223602,44.0347763394495,47.3309594047619,44.1006178391534,38.4677574316013,42.891537445],[-86.8421761942196,-92.3806400730717,-111.684405508532,-119.76701027461,-105.292483266667,-72.8145636355556,-77.0191435671141,-75.5283352912621,-82.013117696129,-83.7336035641762,-93.3512747917431,-114.868868707101,-88.982859964891,-86.2831591658631,-97.1338809590062,-84.8205435111111,-91.7202169829396,-71.4698935272021,-76.7690851248025,-69.4348348680688,-84.7380763506601,-94.1140633418561,-92.4538914823336,-89.7047727912886,-110.301706052758,-79.5412661185383,-99.6473657401392,-98.172743,-71.5672904483871,-74.5149732133508,-106.156820970721,-116.64699009375,-75.0511997528434,-82.743376,-96.9968852169576,-122.086067508,-77.5478256012243,-71.5091485106383,-81.0330604506284,-99.1477312800926,-86.4616643064713,-98.0461524530347,-111.827015060942,-78.1649987414596,-72.6447321865443,-121.19897855291,-89.5289975470899,-80.9729302841994,-107.25199105],[129750,110850,87900,412950,103500,67500,44700,15450,232500,156600,163500,50700,247800,155550,120750,155250,114300,115800,94950,78450,181800,158400,182550,82650,62550,168300,64650,95850,46500,114600,66600,38400,342900,225750,120300,75000,343050,14100,83550,64800,122850,415200,54150,193200,49050,113400,141750,141450,30000],null,null,{"lineCap":null,"lineJoin":null,"clickable":true,"pointerEvents":null,"className":"","stroke":true,"color":"#03F","weight":1,"opacity":0.5,"fill":true,"fillColor":"#03F","fillOpacity":0.2,"dashArray":null},null,null,null,null,null,null]}],"limits":{"lat":[28.1489504167742,47.5112979280742],"lng":[-122.086067508,-69.4348348680688]}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


Plotting this in a pareto chart to see more clearly.


```r
# Filter for top10 states
zipcode5 <- arrange(zipcode4,desc(count_zipcode))[1:20,]

plot1 <- ggplot(zipcode5, aes(x=reorder(state,count_zipcode), y=count_zipcode, fill=count_zipcode)) + 
  geom_bar(stat="identity") + 
  scale_fill_gradient2(low='red', mid='snow3', high='red', space='Lab') +
  labs(title="Top 20 US State with Most Number of Zipcodes", y="Number of Zipcodes", x="State") +
  coord_flip()
plot1
```

![](Assignment_on_Leaflet_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


We can see that Texas is the state with the highest number of zipcodes, follow by California and Pennsylvania.
