---
title: "P3 Data Enrich/Format/Blend (Revised)"
author: 'Team 1: Adrita Anika, Anantha Sai Sreelekha, Soham Sarda'
date: '2022-03-25'
output:
  html_document:
    df_print: paged
  word_document: default
  pdf_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

We have four datasets: Collision, Person, Vehicles and Weather datsets. The following steps are taken for enrichment, blending and blending the datasets.

##### Enrichment and Formatting:

1.  Creating time-based features: We have one variable named as "CRASH_DATE' which contains the date, month and year information. From that, Crash Year', 'Crash Month', 'Crash Day', and 'Time of day' variables were derived. These variables will be used to visualize the temporal distribution of crashes and look for patterns, if any.

    ```{r, message = FALSE}
    library(lubridate)
    crash_data <- read.csv("NYC_crashes.csv")
    crash_data$CRASH_DAY <- day(crash_data$CRASH_DATE)
    crash_data$CRASH_MONTH <- month(crash_data$CRASH_DATE)
    crash_data$CRASH_YEAR <- year(crash_data$CRASH_DATE)
    ```

2.  We want to see patterns based on weekdays (Sat, Sun,.. etc). Hence, we enhance the data with another variable containing the weekday information.

    ```{r, message=FALSE}
    crash_data$CRASH_WDAY <- weekdays(as.Date(crash_data$CRASH_DATE))
    ```

3.  For time analysis, we divided 24 hours of the day time into 12 timeslots. Like: 00:00-01:59, 2:00-03:59,..., 22:00-23:59. Later we represented this with numbers starting from 0 to 11 as shown later in this report.

4.  We have the geographic information: longitude and latitude and zip code. We will consider the zip codes as location IDs in our analysis and place a marker on the map for each zip code, so we need to fix a longitude and latitude value for the marker to be placed for each zip code. We considered the average of all available longitudes under a zip code for placing a marker on the map. Same thing is done for latitude as well. This data is required for spatial analysis, we save this as "cluster_data.csv'.

    ```{r, message=FALSE}
    library(dplyr)
    cluster_data <- crash_data %>% group_by(ZIP_CODE) %>% summarise(LAT = mean(LATITUDE),
                                                                  LNG = mean(LONGITUDE),
                                                                  Crash = n(),)
    ```

5.  All the column headers are once again formatted with a consistency followed - Upper case letters with underscores between words.

6.  For our analysis we created another variable named as "VEHICLE_TYPE" for the vehicles dataset. We categorized all the vehicles depending on their sizes into four categories: very_large, large, medium, small. This has been done following this paper [[Paper Link](https://www.researchgate.net/publication/337243001_Applying_Big_Data_Analytics_on_Motor_Vehicle_Collision_Predictions_in_New_York_City)]

7.  The weather dataset does not have any common variable apart from geodetic reference. The Lat-Long coordinates were used to find the nearest weather station to each crash activity, and was then the weather data for that station on the particular day of crash was linked to the crash data.

8.  We are considering a classification problem of traffic collision severity based on ten features: emotional status, driver license status, vehicle type, two contributing factors, position in vehicle, safety equipment, day and time. We converted some of these variables into numerical values with one hot encoding. This is done only for the classification problem. Following code chunk shows the one-hot encoding process for only one variable: position in vehicle.

    ```{r, message= FALSE}
    perX <- read.csv("NYC_persons.csv")
    per <- perX[, c("COLLISION_ID","POSITION_IN_VEHICLE")]
    per <- filter(per, per$POSITION_IN_VEHICLE!="Unknown")
    library(caret)
    #define one-hot encoding function
    dummy <- dummyVars(" ~ .", data=per)
    #perform one-hot encoding on data frame
    final_df <- data.frame(predict(dummy, newdata=per))
    ```

#### Merging/Blending:

1.  All of the files will be stored in csv format. As we have huge datasets, we created seperate csv files for some of the analysis. For the classification of Traffic collision based on severity into three classes (No hurt, Injury, Lethal) we will use ten attributes from the three datasets: collision, person and vehicle. All these datasets contain one common attribute "COLLISION ID". We inner joined these datasets and kept the ten required features. First few rows of the dataset is shown below. The result is presented after doing one hot encoding.

    ```{r, message = FALSE}
    clf_data <- read.csv("dfx.csv")
    head(clf_data, 3)
    ```

2.  As mentionied earlier, "cluster_data.csv" wil be used for spatial analysis for finding collision hotspots.

    ```{r, message=FALSE}
    head(cluster_data,3)
    ```

3.  For time analysis, we created another csv file that has the information of weekdays and timeslot of the day and corresponding number of collisions. We need this is data for two analysis ( forecasting and visual analysis)

    ```{r, message=FALSE}
    time_data <- read.csv("refined_time.csv") 
    head(time_data,3)
    ```

4.  As mentioned earlier, we categoried vehicles into four groups. The dataset is shown below:

    ```{r, message=FALSE}
    vehicle_data <- read.csv("veh.csv")
    head(vehicle_data,3)
    ```

5.  Finally, our weather dataset is shown:

```{r, message=FALSE}
weather_data <- read.csv("weather.csv")
head(weather_data, 3)
```
