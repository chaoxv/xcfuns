# xcfuns
---
title: "Demo for BGC_Argo_R_toolbox"
author: "Yibin Huang & Marin Cornec"
date: "`r Sys.Date()`"
output: github_document
---
-------------------------------

[Video tutorial](https://www.youtube.com/watch?v=w_6pEGNXQQ4)

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/tutorial.jpg")
```

**Link: https://www.youtube.com/watch?v=w_6pEGNXQQ4**

------------------------------------

# Installation

**Fill the path to the code directory, you can instead set the code
directory as the working directory with setwd()**
```{r}
path_code = ""
```

**Load the functions and libraries**
```{r, eval=FALSE}
setwd(path_code)
func.sources = list.files(path_code,pattern="*.R")
func.sources = func.sources[which(func.sources %in% c('Main_workshop.R',
                                                      "bgc_argo_workshop_R_license.R")==F)]

if(length(grep("Rproj",func.sources))!=0){
  func.sources = func.sources[-grep("Rproj",func.sources)]
}
invisible(sapply(paste0(func.sources),source,.GlobalEnv))

aux.func.sources = list.files(paste0(path_code,"/auxil"),pattern="*.R")
invisible(sapply(paste0(path_code,"/auxil/",aux.func.sources),source,.GlobalEnv))
```

------------------------------------

# Exercise 0: Initialize 

This function defines standard settings and paths and creates Index
and Profiles folders in your current path. It also downloads the Sprof 
index file from the GDAC to your Index folder. The Sprof index is 
referenced when downloading and subsetting float data based on user 
specified criteria in other functions.

```{r, eval=FALSE}
initialize_argo() # Take some minutes to download the global Index
```
[1] "success!"

```{r, eval=FALSE}
float_idx <-which(Float$wmoid=='5906439') # Float IDs for float 5906439 (WMOID) in the S_file index
```

[1] 580 

```{r, eval=FALSE}
prof_ids = c(Float$prof_idx1[float_idx]:Float$prof_idx2[float_idx]) 
# Profile IDs for float 5906439 (WMOID) in the S_file index
```


```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/profile_id.png")
```

```{r, eval=FALSE}
dates = Sprof$date[prof_ids] # Dates of each profile from float #5906439
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/date.png")
```

```{r, eval=FALSE}
sensors = unique(Sprof$sens[prof_ids]) # Sensors available for float #5906439
```
[1] "PRES TEMP PSAL DOXY CHLA BBP700 PH_IN_SITU_TOTAL NITRATE"

------------------------------------

# Exercise 1: SOCCOM float 
In this exercise, we download the NetCDF file for a Southern Ocean BGC float, inspect its contents, show the trajectory, plot profiles for unadjusted and adjusted data, and show the effect of adjustments  made to the nitrate concentrations.

**Download NetCDF file for float #5904183, a SOCCOM float with multiple seasons**
```{r, eval=FALSE}
WMO = 5904859 
success = download_float(WMO)
```
[1] "success!"

**Show the trajectory of the downloaded float**
```{r, eval=FALSE}
show_trajectories(float_ids=WMO, 
                  return_ggplot="True" # Return the plot to ggplot panel
                  )
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/track_excercise_1.png")
```

```{r, eval=FALSE}
show_profiles( float_ids=WMO, 
               variables=c('PSAL','NITRATE'),
               obs='on', # 'on' Shows points on the profile at which each measurement was made
               raw="yes" # Show the unadjusted data ,
              
)
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/profile_1_excercise_1.jpg")
```

**This plots the adjusted data**
```{r, eval=FALSE}
show_profiles(float_ids=WMO, 
              variables=c('PSAL','NITRATE'),
              obs='on', # 'on' Shows points on the profile at which each measurement was made
              raw="no"
)
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/profile_2_excercise_1.jpg")
```

**Show sections for nitrate.** 

This shows the raw, unadjusted data (pcolor plot) mixed layer depth is shown based on the temperature threshold (set the value to 2 after 'mld' to use the density threshold instead)


```{r, eval=FALSE}
show_sections(float_ids=WMO, 
              variables= c('NITRATE'),
              plot_mld=1,       # Tells the function to plot mixed layer depth
              raw="yes") # Tells the function to plot raw data
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/section_1_excercise_1.png")
```

```{r, eval=FALSE}
show_sections( float_ids=WMO ,
               variables= c('NITRATE'),
               plot_mld=1, # Tells the function to plot mixed layer depth
               raw="no",
               qc=c(1:2) # Tells the function to plot good and probably-good data
)

```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/section_2_excercise_1.png")
```

------------------------------------

# Exercise 2: Ocean Station Papa floats
In this exercise, we define a region in the Northeast Pacific along with a duration of time, and identify the float profiles matching that criteria. We show the trajectories of all the matching floats and plot
profiles that match the criteria for one of the floats.

**Set limits near Ocean Station Papa from 2008 to 2018**

```{r, eval=FALSE}
lat_lim=c(45, 60)
lon_lim=c(-150, -135)
start_date="2008-01-01"
end_date="2018-12-31"
```

**Select profiles based on those limits with specified sensor (NITRATE)**

```{r, eval=FALSE}
OSP_data= select_profiles ( lon_lim,
                            lat_lim,
                            start_date,
                            end_date,
                            sensor=c('NITRATE'), # This selects only floats with nitrate sensors
                            outside="both") # This allows the user to specify whether to retain profiles  
                                            # lie outside the space limits ('space')
```

**Display the number of matching floats and profiles**

```{r, eval=FALSE}
print(paste('# of matching profiles:',sum(lengths(OSP_data$float_profs))))
```

[1] "# of matching profiles: 1850"

```{r, eval=FALSE}
print(paste('# of matching floats:',length(OSP_data$float_ids)))
```

[1] "# of matching floats: 7"

**Load the data for the matching float with format of data frame**

```{r, eval=FALSE}
data_OSP_df= load_float_data( float_ids= OSP_data$float_ids, # Specify WMO number
                           float_profs=OSP_data$float_profs, # Specify selected profiles
                            variables="ALL", # Load all the variables
                            format="dataframe" # Specify data output format;  
)
```

**Show trajectories for the matching floats**

This function downloads the specified floats from the GDAC (unless the
files have already been downloaded) and then loads the data for plotting.
Adding the optional input pair 'color','multiple' will plot different
floats in different colors

```{r, eval=FALSE}
show_trajectories(float_ids = OSP_data$float_ids,
                               return_ggplot = TRUE  # This plots different floats in different colors
)+ geom_rect( aes(xmin = lon_lim[1],
                 xmax = lon_lim[2],
                 ymin = lat_lim[1],
                 ymax = lat_lim[2]),
                  color="black",fill=NA # Add a frame around the selected area
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/track_excercise_2.png")
```

**Show profile plots for the first of these matching floats**

**Case #1: all Salinity and Oxygen profiles from one float (1)**

```{r, eval=FALSE}
show_profiles(float_ids=OSP_data$float_ids[1], 
              variables=c('PSAL','DOXY'),
              obs='on',# 'on' Shows points on the profile at which each measurement was made
)

```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/profile_1_excercise_2.jpg")
```

**Case #2: mean and standard deviation of all profiles from one float (1)**

```{r, eval=FALSE}
show_profiles(float_ids=OSP_data$float_ids[1], 
              variables=c('PSAL','DOXY'),
              obs='on', # 'on' Shows points on the profile at which each measurement was made
              method="mean" # This tells the function to just plot the mean profile
)
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/profile_2_excercise_2.png")
```

------------------------------------
# Exercise 3: Hawaii floats

In this exercise, we define a region near Hawaii along with a duration of time. Again, we identify the float profiles matching those criteria, show their trajectories, plot all the matching profiles on one figure, and show sections for the unadjusted and adjusted values of salinity and dissolved oxygen.

**Set limits near Hawaii from 2017 to 2019**

```{r, eval=FALSE}
lat_lim=c(22, 26)
lon_lim=c(-160, -155)
start_date="2017-01-01"
end_date="2019-12-31"
```

**Select profiles based on those limits**

```{r, eval=FALSE}
HW_data= select_profiles ( lon_lim, 
                           lat_lim, 
                           start_date,
                           end_date,
                           outside="none" # Exclude profiles outside the time and space limits
)
```

**Display the number of matching floats and profiles**
```{r, eval=FALSE}
print(paste('# of matching profiles:',sum(lengths(HW_data$float_profs))))
```
[1] "# of matching profiles: 252"

```{r, eval=FALSE}
print(paste('# of matching floats:',length(HW_data$float_ids)))
```
[1] "# of matching floats: 5"

**Show trajectories for the matching floats, along with the geo limits.**

This function downloads the specified floats from the GDAC (unless the
files have already been downloaded) and then loads the data for plotting.

```{r, eval=FALSE}
show_trajectories(float_ids=HW_data$float_ids, 
                  return_ggplot=TRUE)
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/track_excercise_3.png")
```

**Show matching profiles from all floats**

Show profiles (from all floats) within specified domain and times

```{r, eval=FALSE}
show_profiles( float_ids=HW_data$float_ids, 
               variables=c('PSAL','DOXY'),
               float_profs=HW_data$float_profs,
               per_float=F,  # Show all profiles in one plot
               qc_flags =c(1,2) # Tells the function to plot good and probably-good data
)
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/profile_1_excercise_3.jpg")
```

**Show only matching profiles from September.** 

Determine profiles that occur in September for each float separately

```{r, eval=FALSE}
date<-get_lon_lat_time(float_ids=HW_data$float_ids,
                       float_profs=HW_data$float_profs)$time
obably-good data
)

HW_float_profs_Sep<-list()
for (f in 1:length(HW_data$float_ids)){
  HW_float_profs_Sep[[f]] <-
    HW_data$float_profs[[f]][which(month(as.POSIXct(date[[f]]))==9)]
  
}
```


```{r, eval=FALSE}
show_profiles( float_ids=HW_data$float_ids, 
               variables=c('PSAL','DOXY'),
               float_profs = HW_float_profs_Sep, 
               per_float=F,  # Show all profiles in one plot:
               obs='on', # Plot a marker at each observation
               qc_flags=c(1,2),  # Apply QC flags
               title_add= 'September' 
)
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/profile_2_excercise_3.jpg")
```

**Show sections for pH and oxygen for the fifth float in the list of Hawaii floats**. 

This shows the raw, unadjusted data mixed layer depth is shown based on the temperature threshold (set the value to 2 after 'mld' to use the density threshold instead)


```{r, eval=FALSE}
show_sections( float_ids=HW_data$float_ids[5],
               variables=c('PH_IN_SITU_TOTAL','DOXY'),
               plot_mld=1, # Plot the mixed layer depth
               raw="yes" # Plot raw (unadjusted) data
) 
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/section_1_excercise_3.png")
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/section_2_excercise_3.png")
```


**Show time series of near-surface pH and oxygen for two floats.**

The pH sensor for float 5906039 failed in late 2019, which is evident from the premature end to the line halfway through the first figure.

```{r, eval=FALSE}
show_time_series ( float_ids=HW_data$float_ids[4:5],
               variables=c('PH_IN_SITU_TOTAL','DOXY'),
               plot_depth=20, # Plot the time-series for the given depth (20m)
               raw="no"   # Plot adjusted data
) 
```

```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/timeseries_1_excercise_3.png")
```
```{r echo=FALSE, fig.cap="", out.width = '60%'}
knitr::include_graphics("https://raw.githubusercontent.com/287408731/BGC-Argo-toolbox-figure/main/figures%20for%20demo/timeseries_2_excercise_3.png")
```

</p>
