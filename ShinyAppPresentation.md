Exploring Maryland Bridges With Shiny
========================================================
author: Mark Potter
date: January 19, 2017
autosize: true



Overview
========================================================

The Shiny Application Exploring Maryland Bridges was developed using RStudio.  The data from the presentation was obtained from the DOT website with the following link https://catalog.data.gov/organization/dot-gov . The state of Maryland only bridges was extracted from the dataset.  In addition, a zip code dataset was then used to allow the user to explore the bridges by zip code.  Detailed information about the bridges is shown when the user clicks on the circular marker.  The Shiny Application was published using Shinyapps.io and is available for viewing from a standard HTML5 compliant browser at https://mspinc54.shinyapps.io/shinyappweek4/ .  The application source code is available at https://github.com/mpotter54/ShinyAppWeek4 .  The contents of this presentation may be viewed at http://rpubs.com/mpotter54/ShinyAppWeek4

Preparing the Data
========================================================

Several lines of r code were stored in a helpers.R file to pre-process the data.


```r
# read in zip codes
setwd("C:/DDP/Assignment3/ShinyAppWeek4")
fileZips <- "data/MarylandZipCodes.csv"
zipInput <- read.csv(file=fileZips, head=TRUE, sep=",",na.strings=c('#DIV/0', '', 'NA'))
# read in bridges
fileBridges <- "data/MarylandBridges.csv"
bridgeInput <- read.csv(file=fileBridges, head=TRUE, sep=",",na.strings=c('#DIV/0', '', 'NA'))
# create longname column for leaflet columns
bridgeInput$longName <- paste("<table>",
                              "<tr><td>ID&nbsp;&nbsp;&nbsp;</td>", "<td><b>", as.character(bridgeInput$id), "</b></td></tr>",
                              "<tr><td>ROAD&nbsp;&nbsp;&nbsp;</td>", "<td><b>", bridgeInput$ROAD, "</b></td></tr>",
                              "<tr><td>LOCATION&nbsp;&nbsp;&nbsp;</td>", "<td><b>", bridgeInput$LOCATION, "</b></td></tr>",
                              "<tr><td>ZIP&nbsp;&nbsp;&nbsp;</td>", "<td><b>", as.character(bridgeInput$ZIPCODE), "</b></td></tr>",
                              "<tr><td>OPEN&nbsp;CODE&nbsp;&nbsp;&nbsp;</td>", "<td><b>", bridgeInput$OPEN_CODE, "</b></td></tr>", 
                              "<tr><td>OPEN&nbsp;DESC&nbsp;&nbsp;&nbsp;</td>", "<td><b>", bridgeInput$OPEN_DESC, "</b></td></tr>", 
                              "<tr><td>YEAR&nbsp;BUILT&nbsp;&nbsp;&nbsp;</td>", "<td><b>", as.character(bridgeInput$YEAR_BUILT), "</b></td></tr>", sep="")
# create zip code with number of brides in zip in parens for drop down
fun <- function(x) {nrow(subset(bridgeInput, ZIPCODE == x))}
zipInput$Cnt <- lapply(zipInput$ZIP, fun)
zipInput$Cnt[1] = nrow(bridgeInput)
zipInput$ZIPCNT <- paste(as.character(zipInput$ZIP), '(', as.character(zipInput$Cnt), ')', sep = '')
# only show zip codes that have bridges
zipInput <- subset(zipInput, Cnt > 0)
```

Server Side Code
========================================================

Server.r held the server code.  Reactive input was done on the zip code drop down to re-render a leaflet map.


```r
# server.R
source("helpers.R")
shinyServer(function(input, output, session) {
        fb <- reactive({
                # react to change in zip code drop down by subsetting bridge df by zip code
                z = as.character(input$zipCodes)
                z2 = substring(z, 1, 3)
                if (z2 != 'All')
                {
                        # use numeric comparison for speed optimization
                        z2 = as.numeric(substring(z, 1, 5))
                        subset(bridgeInput, ZIPCODE == z2)
                }
                else
                {
                        bridgeInput
                }
        })
        observe({
                # render leaflet map, for all use mark cluster options, for individual zip do not cluster
                if (substring(as.character(input$zipCodes), 1, 3) != 'All')
                {
                        output$map <- renderLeaflet({
                        fb() %>% leaflet() %>%
                                 addTiles() %>%
                                 addCircleMarkers(weight=1, popup=fb()$longName)})
                }
                else
                {
                        output$map <- renderLeaflet({
                                fb() %>% leaflet() %>%
                                        addTiles() %>%
                                        addCircleMarkers(weight=1, popup=fb()$longName, 
                                                         clusterOptions = markerClusterOptions())})
                }
        })
})
```

Client Side Code
========================================================

ui.r held the client code.  This is very straightforward simple layout


```r
library(leaflet)
shinyUI(fluidPage(
        titlePanel("Maryland Bridges"),
        sidebarLayout(
                sidebarPanel(
                        helpText("Explore Maryland Bridges by Zip Code or All"),
                        selectInput("zipCodes", "Zip Codes", c("All (0)")
                        ), width=3),
                 mainPanel(
                         leafletOutput("map"),
                         br(),
                         htmlOutput("bridges"),
                         br(),
                         htmlOutput("bridgeHelp"), 
                         width=9)
        )
))
```
