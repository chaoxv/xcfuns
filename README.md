---
output: github_document
---

<!-- README.md is generated from README.Rmd. Please edit that file -->

```{r, include = FALSE}
knitr::opts_chunk$set(
  collapse = TRUE,
  comment = "#>",
  fig.path = "man/figures/README-",
  out.width = "100%"
)
```

# nppr

<img src="https://github.com/chaoxv/figures/blob/main/nppr.png?raw=true" height="200" align="right" /></a>

```{r comment="", echo=FALSE, results='asis'}
cat(packageDescription('nppr')$Description)
```


## :writing_hand: Authors

Chao Xu
`r badger::badge_custom("follow me on", "WeChat", "green", "https://github.com/chaoxv/figures/blob/main/wechat.jpg?raw=true")`

Xiamen University


## :arrow_double_down: Installation

You can install the development version of nppr like so:

``` r
# install.packages("remotes")
remotes::install_github("chaoxv/nppr")
```

## Example

This is a basic example which shows you how to solve a common problem:

```{r example}
library(nppr)
```

1. Download ocean productivity data

```{r eval=FALSE}
# Load supporting packages
library(RCurl)
library(XML)
library(R.utils)
library(tidyverse)
library(lubridate)

# Create an empty folder to save your ocean produtivity files.
yourfolder <- paste0(getwd(), '/', 'vgpm') 
dir.create(yourfolder)

# Dowmload the VGPM data. If it is not defined, the default  grid size is 'low', time span is 'monthly', satellite is 'MODIS'.
get_npp_vgpm(file.path = yourfolder,
             grid.size = 'low',
             time.span = 'monthly',
             satellite = 'MODIS',
             mindate = '2016-01-15', 
             maxdate = '2016-03-15') 

```


2. Read hdf file

```{r eval=FALSE}
# Use the hdf file you have downloaded yet.
yourfile <- paste0(yourfolder, '/201601.hdf')
read_hdf(file.path = yourfile)    

```

3. Match the ocean productivity data by your longitude, latitude and date.

```{r eval=FALSE}

# Extract a single value
match_sig(file.path = yourfolder,
          lon = 120, lat = 20, date = '2016-03-01')

# Extract multiple values
mydat <- data.frame(lon = c(120, 112, 116), 
                    lat = c(17, 15, 18),
                    date = c('2016-03-04', '2016-03-07', '2016-02-04'))

match_df(mydat, file.path = yourfolder)  

```

4. Ocean productivity data visualization.

```{r fig.dim = c(7.5, 7.5), warning=FALSE, message=FALSE}
# Load supporting packages



library(viridis)
library(raster)
library(ggplot2)
library(ggspatial)
library(rnaturalearth)
library(rnaturalearthdata)
data(nppdata)

  ggplot()+
  geom_oce(nppdata, aes(x = lon, y = lat, fill = npp), 
           lonlim = c(100, 120), latlim = c(7, 25))+
  scale_fill_viridis(option = "D", 
                     direction = -1,
                     breaks = seq(50, 1050, 100),
                     limits = c(50, 1050))+
  labs(x = 'Longitude', y = 'Latitude',
  fill = expression(NPP*~'('*mg~C~m^-2~d^-1*')'))

```

## :sparkling_heart: Contributing

We welcome any contributions! By participating in this project you agree to
abide by the terms outlined in the [Contributor Code of Conduct](CONDUCT.md).
