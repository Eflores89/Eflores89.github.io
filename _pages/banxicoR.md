---
layout: post
title: "banxicoR"
author: "e"
permalink: /banxicoR/
---


The **banxicoR** package is now available on CRAN. Much like [inegiR](https://github.com/Eflores89/inegiR) this package aims to bring official Mexican data easily into R, in this case by scrapping *iqy* calls to the SIE (Sistema de Información Económica) webservice of the Bank of Mexico. 

### Introduction 

The major difference with inegiR is that the Bank of Mexico does not have an API, so this package basically uses `rvest` to scrape the generated html. The package then does what it can to save it in a convenient `data.frame` or `list` (same as inegiR).

A few caveats:

  - I do not control the data definitions at the source (Banxico), so I can't guarantee continuos use. 
  - Finding the ID of each series has proven a manual, slow and tedious process because they are not available in the webpage. I'm trying to contact Banxico about getting a catalog, but basically use at your own risk. 


### Finding series ID's

To find a specific series ID, I would recommend going to the [SIE webpage](http://www.banxico.org.mx/SieInternet/), navigating towards the desired indicators and then consulting them via HTML. The column name should be the series id (they are usually in this format: "SF60653", with two characters followed by numbers). The package includes a **small and non exhaustive** catalog of series **in spanish**. You can access this by `data("BanxicoCatalog")`.


Then you can find some id's...

~~~~~~~
library(banxicoR)
library(dplyr)
data("BanxicoCatalog")

# To see how many id's by parent subject 
BanxicoCatalog %>% 
  group_by(PARENT) %>% 
  summarise("Id's" = n())

# Source: local data frame [4 x 2]

#                       PARENT     Id's
#                       (chr)     (int)
#  1          Billetes y Monedas    45
#  2 Intermediarios Financierios    37
#  3            Sistemas de Pago    49
#  4              Tipo de Cambio     2

# to bring the specific id of the average duration of 200 peso bills...
BanxicoCatalog %>% 
  filter(PARENT == "Billetes y Monedas") %>% 
  filter(LEVEL_1 == "Duración promedio del billete") %>% 
  filter(LEVEL_2 == "200 pesos") %>% 
  select(ID)

# Source: local data frame [1 x 1]

#      ID
#    (chr)
#1    SM32
~~~~~~~

### Usage
Now that you have some id's to download, we can use the `banxico_series` function...

~~~~~~~
# Download the Bank of Mexico international reserves
rsv <- banxico_series(series = "SF110168")

tail(rsv)
#          DATE SF110168
#    2016-01-01 176321.4
#    2016-02-01 178408.8
#    2016-03-01 179708.0
#    2016-04-01 182118.8
#    2016-05-01 179351.0
#    2016-06-01 178829.9
~~~~~~~

If you want some other fancy things, you can use the options... 

~~~~~~~
rsv <- banxico_series(series = "SF110168", 
                      metadata = TRUE, 
                      verbose = TRUE)
# [1] "Data series: SF110168 downloaded"
# [1] "Data series in monthly frequency"
# [1] "Parsing data with 198 rows"

str(rsv)
#List of 2
# $ MetaData:List of 6
#  ..$ IndicatorName: chr "I. Official Reserve Assets And Other Foreign Currency Assets - A. Official Reserve Assets"
#  ..$ IndicatorId  : chr "SF110168"
#  ..$ Units        : chr "millions of u.s. dollar"
#  ..$ DataType     : chr "market value/price"
#  ..$ Period       : chr "jan 2000 - jun 2016"
#  ..$ Frequency    : chr "monthly"
# $ Data    :'data.frame':	198 obs. of  2 variables:
#  ..$ Dates   : Date[1:198], format: "2000-01-01" ...
#  ..$ SF110168: num [1:198] 33689 33382 36435 34749 33624 ...
~~~~~~~
Finally, we can graph this... 

~~~~~~~
library(ggplot2)
library(eem) # theme from: https://github.com/Eflores89/eem

ggplot(rsv$Data, 
       aes(x = Dates, y = SF110168))+
       geom_path(colour = eem_colors[1])+
       theme_eem()+
       labs(x = "Dates", 
            y = paste0("Reserves in U.S. Dollars \n (", rsv$MetaData$Units, ")"), 
            title = "Bank of Mexico International Reserves")
~~~~~~~

This data series is also available at INEGI and can be downloaded with [inegiR](https://github.com/Eflores89/inegiR) but Banxico has other interesting series exclusive to them, like financial loans or money in circulation, which I encourage everyone to check out!  

[Tweet](https://twitter.com/eflores89) me up if you have any suggestions / improvements or open [an issue at Github](https://github.com/Eflores89/banxicoR)
