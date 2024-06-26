# Google-Data-Analytics-Cyclistic-Case-Study
Capstone project for the Google Data Analytics Certificate on Coursera

Visualization Analysis can be found in [My Tableau](https://public.tableau.com/app/profile/yung.chyi.yang/viz/GoogleDataAnalyticsCapstoneProject_16929053599130/Dashboard1)

## Table of Contents

1. [Introduction](#introduction)
2. [Ask](#ask)
3. [Prepare](#prepare)
4. [Process](#process)
5. [Analyze](#analyze)
   * [Visualization](#visualization)
7. [Act (Recommendations)](README.md##Act-(Recommendations))
   
## Introduction
This Google Data Analytics Cyclistic Case Study is to work for a fictional company, Cyclistic. In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. The goal is to design marketing strategies to convert casual riders into annual members and my own task is to understand how casual riders and Cyclistic members behave differently. In order to answer the key business questions, the steps of the data analysis process: ask, prepare, process, analyze, share, and act will be launched.

## Ask
> **Business Task**: To clean, analyze and visualize the data to observe how casual riders use the bike rentals differently from annual member riders and determine the best marketing strategies to turn casual bike riders into annual members.

## Prepare
The datasets are retrieved from https://divvy-tripdata.s3.amazonaws.com/index.html and are in .csv format. Given that the data is in large amounts, I decided to use Rstudio to prepare and clean up the data.

First, Install the required packages for the project.
```
install.packages("tidyverse")
install.packages("janitor")
install.packages("ggmap")
library(tidyverse)
library(tibble)
library(readr)
library(janitor)
library(rmarkdown)
library(ggplot2)
library(ggmap)
```

Save each csv files into data frames.
```
df1<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202201-divvy-tripdata.csv", header = TRUE)
df2<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202202-divvy-tripdata.csv", header = TRUE)
df3<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202203-divvy-tripdata.csv", header = TRUE)
df4<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202204-divvy-tripdata.csv", header = TRUE)
df5<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202205-divvy-tripdata.csv", header = TRUE)
df6<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202206-divvy-tripdata.csv", header = TRUE)
df7<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202207-divvy-tripdata.csv", header = TRUE)
df8<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202208-divvy-tripdata.csv", header = TRUE)
df9<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202209-divvy-publictripdata.csv", header = TRUE)
df10<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202210-divvy-tripdata.csv", header = TRUE)
df11<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202211-divvy-tripdata.csv", header = TRUE)
df12<-read.csv("/Users/yangyungchyi/Documents/Learn/Cyclistic/202212-divvy-tripdata.csv", header = TRUE)
```
Check if there is any mismatch of the column name before combining
```
compare_df_cols(df1,df2,df3,df4,df5,df6,df7,df8,df9,df10,df11,df12, return = "mismatch")
```
Bind the dataframes into one dataframe
```
data2022<-rbind(df1,df2,df3,df4,df5,df6,df7,df8,df9,df10,df11,df12)
```

## Process
Clean the data by dropping NA values and using the distinct function.
```
data2022_clean<-data2022%>%
  drop_na()%>%
  distinct()
```

Add columns that separate the dates into month, day, year and day of the week:
```
data2022_clean$Date<-as.Date(data2022_clean$started_at)
data2022_clean$Month<- format(as.Date(data2022_clean$Date), "%m")
data2022_clean$Day<- format(as.Date(data2022_clean$Date), "%d")
data2022_clean$Year<- format(as.Date(data2022_clean$Date), "%Y")
data2022_clean$Day_of_week<- format(as.Date(data2022_clean$Date), "%A")
```

Create a new columns called "ride length". The unit is second.
```
data2022_clean$ride_length <- difftime(data2022_clean$ended_at,data2022_clean$started_at)
```

Inspect the structure of the columns
```
str(data2022_clean)
```

Convert "ride_length" from Factor to numeric so we can run calculations on the data
```
is.factor(data2022_clean$ride_length)
data2022_clean$ride_length <- as.numeric(as.character(data2022_clean$ride_length))
is.numeric(data2022_clean$ride_length)
```

Remove "bad" data: The dataframe includes a few hundred entries when bikes were taken out of docks and checked for quality by Divvy or ride_length was negative
```
data2022_clean <- data2022_clean[!(data2022_clean$start_station_name == "HQ QR" | data2022_clean$ride_length<0),]
```
## Analyze
Descriptive analysis on ride_length (all figures in seconds)
```
summary(data2022_clean$ride_length)
Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
0.0     349.0     616.0     979.8    1105.0 2057644.0 
```

Compare members and casual users
```
aggregate(data2022_clean$ride_length ~ data2022_clean$member_casual, FUN = mean)
  data2022_clean$member_casual data2022_clean$ride_length
1                       casual                  1319.2825
2                       member                   744.7387
aggregate(data2022_clean$ride_length ~ data2022_clean$member_casual, FUN = median)
  data2022_clean$member_casual data2022_clean$ride_length
1                       casual                        778
2                       member                        530
aggregate(data2022_clean$ride_length ~ data2022_clean$member_casual, FUN = max)
  data2022_clean$member_casual data2022_clean$ride_length
1                       casual                    2057644
2                       member                      89996
aggregate(data2022_clean$ride_length ~ data2022_clean$member_casual, FUN = min)
  data2022_clean$member_casual data2022_clean$ride_length
1                       casual                          0
2                       member                          0
```

The average ride time by each day for members vs casual users, therefore reorder days of the week and rerun
```
aggregate(data2022_clean$ride_length ~ data2022_clean$member_casual + data2022_clean$Day_of_week, FUN = mean)
data2022_clean$Day_of_week <- ordered(data2022_clean$Day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))
aggregate(data2022_clean$ride_length ~ data2022_clean$member_casual + data2022_clean$Day_of_week, FUN = mean)
```
Result:
```
data2022_clean$member_casual data2022_clean$Day_of_week data2022_clean$ride_length
1                        casual                     Sunday                  1505.9880
2                        member                     Sunday                   821.7778
3                        casual                     Monday                  1357.7269
4                        member                     Monday                   720.2955
5                        casual                    Tuesday                  1177.9052
6                        member                    Tuesday                   708.6599
7                        casual                  Wednesday                  1140.0512
8                        member                  Wednesday                   710.0717
9                        casual                   Thursday                  1180.4805
10                       member                   Thursday                   720.6420
11                       casual                     Friday                  1231.8108
12                       member                     Friday                   733.5506
13                       casual                   Saturday                  1478.6859
14                       member                   Saturday                   827.4545
```

analyze ridership data by type and weekday
```
data2022_clean %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>%  
  group_by(member_casual, weekday) %>%
  summarise(number_of_rides = n()							
            ,average_duration = mean(ride_length)) %>% 		
  arrange(member_casual, weekday)
```
Result:
```
# A tibble: 14 × 4
# Groups:   member_casual [2]
   member_casual weekday number_of_rides average_duration
   <chr>         <ord>             <int>            <dbl>
 1 casual        Sun              388011            1506.
 2 casual        Mon              277054            1358.
 3 casual        Tue              263187            1178.
 4 casual        Wed              273823            1140.
 5 casual        Thu              308713            1180.
 6 casual        Fri              333938            1232.
 7 casual        Sat              472082            1479.
 8 member        Sun              387117             822.
 9 member        Mon              473249             720.
10 member        Tue              518507             709.
11 member        Wed              523770             710.
12 member        Thu              532154             721.
13 member        Fri              466985             734.
14 member        Sat              443169             827.
```
### Visualization
I chose Tableau as my visualization tool. I have created a dashboard that can be found here: https://public.tableau.com/app/profile/yung.chyi.yang/viz/GoogleDataAnalyticsCapstoneProject_16929053599130/Dashboard1

## Act (Recommendations)
The recommendations I would provide to fulfill the business task are:
* Enhance incentives for cold-weather riding, such as distributing coupons and offering discounts. The aim is to promote rides during different seasons, considering that the Summer season currently stands as the most favored.
  
* Casual riders are more active on weekends, therefore implementing discount campaigns targeted at casual riders during weekdays could serve as a motivational factor for them to choose the service for their daily commute, potentially leading to their conversion into regular members.

* As shown in the analysis, casual riders frequently choose routes along the coastline. To capitalize on this trend, the company could offer membership benefits at stations located near the coast and on specific routes that are particularly favored by casual riders.
