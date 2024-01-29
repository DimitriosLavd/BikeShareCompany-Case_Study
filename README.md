# BikeShareCompany-Case_Study
An -one week- data analysis case study for a bike share company. Includes Python and SQL codes. 

## Table of Contents
- [Project Overview and Case Study Scenario](#project-overview-and-case-study-scenario)

- [Data Source](#data-source)

- [Tools](#tools)

- [Data Cleaning and Preparation](#data-cleaning-and-preparation)

- [Exploratory Data Analysis](#exploratory-data-analysis)

- [Data Analysis](#data-analysis)

- [Results and Findings](#results-and-findings)

- [Recomendations](#recomendations)

- [Refrences](#refrences)


### Project Overview and Case Study Scenario
You are a junior data analyst working on the marketing analyst team at Cyclistic, a
bike-share company in Chicago. The director of marketing believes the company’s
future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members
use Cyclistic bikes differently. From these insights, your team will design a new
marketing strategy to convert casual riders into annual members. But first, Cyclistic
executives must approve your recommendations, so they must be backed up with
compelling data insights and professional data visualizations. 1.2 Data Analysis Objectives
The company leaders are determined to make data driven decisions regarding their
future strategy. The future marketing program will be guided by the following
questions. 

1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?

We are assigned to provide data-driven insights in order to answer the first
question. We need to analyze the data, in order to identify trends that show how
annual members and casual users use the company’s services differently.Analyzing
these differences correctly will be the key for answering the other two questions
and achieving the company’s objective.

### Data Source 
Trip Data: Our primary data scource is the "202004-divvy-tripdata.CSV", that contains detailed information about the trips made using the company's bikes, for 12 months.

### Tools 
- Google sheets: Data cleaning and visualization
- WPS Sheets: Data cleaning
- Python: Data cleaning and manipulation
- BigQuery (SQL): Data analysis
- Tableau: Data visualization
- 
### Data Cleaning and Preparation
In the initial data preparation phase, we performed the following tasks:
1. Loading and inspecting the data set.
2. Correcting the format of the date-time columns using Google Sheets.
3. Managing missing values using Google Sheets.
4. Formatting the latitude and longitude columns using Goodgle Sheets and a Python script.
``` python
"""
Created on Mon Jan  8 23:30:33 2024

@author: jim47
"""
import pandas as pd
import numpy as np


""" Import the semi-cleaned dataset """
tripdata = pd.read_csv(r"D:\data analysis_2\CASE_STUDY\tripdata_cleaning_phase1.csv")

""" fix the wrong lat-long values"""
for i in range(84776):
    if tripdata.start_lat[i]>100 :
        tripdata.start_lat[i] = tripdata.start_lat[i]*0.1
    else:
        tripdata.start_lat[i] = tripdata.start_lat[i]*1 

for i in range(84776):
    if tripdata.end_lat[i]>100 :
        tripdata.end_lat[i] = tripdata.end_lat[i]*0.1
    else:
        tripdata.end_lat[i] = tripdata.end_lat[i]*1
        
for i in range(84776):
   if tripdata.start_lng[i]<-100 :
      tripdata.start_lng[i] = tripdata.start_lng[i]*0.1
   else:
      tripdata.start_lng[i] = tripdata.start_lng[i]*1

for i in range(84776):
    if tripdata.end_lng[i]<-100 :
        tripdata.end_lng[i] = tripdata.end_lng[i]*0.1
    else:
        tripdata.end_lng[i] = tripdata.end_lng[i]*1
        
cleaned_tripdata = tripdata

cleaned_tripdata.to_csv(r"D:\data analysis_2\CASE_STUDY\tripdata_cleaning_phase2.csv")
```
5. Adding two useful columns for our analysis -> ride_length and day_of_week
6. Deleting rows that contained wrong information in the ride_length column.
   
### Exploratory Data Analysis
After the data cleaning and preparation phase we produced the "tripdata_CLEANED.CSV" file, on whitch we based our analysis. Our analytical process was based on some key questions such as the following.
1. How many bike trips did each of the two user groups made for 12 past months?
2. What is the average trip duration for each of the two groups?
3. How the trip percentage composition change throuought an average week?
4. In which areas of Chicago City do the two user groups' members tend to start their bike trips?

### Data Analysis
In this section we present some interesting sql queries we wrote in order to analyze our data set:
1. Creating a new column that calculates the ride length in DAYS/HOURS/MINUTES/SECONDS format:
``` sql
WITH
  difference_in_seconds AS (
  SELECT
    ride_id,
    started_at,
    ended_at,
    member_casual,
    ride_length,
    TIMESTAMP_DIFF(ended_at, started_at,SECOND) AS seconds
  FROM
    `bike-share-case-study-410714.TripData.trip_data` ),
  differences AS (
  SELECT
    ride_id,
    started_at,
    ended_at,
    ride_length,
    member_casual,
    seconds,
    MOD(seconds, 60) AS seconds_part,
    MOD(seconds, 3600) AS minutes_part,
    MOD(seconds, 3600 * 24) AS hours_part
  FROM
    difference_in_seconds )
SELECT
  ride_id,
  started_at,
  ended_at,
  ride_length,
  member_casual,
  CONCAT( FLOOR(seconds / 3600 / 24), ' days ', FLOOR(hours_part / 3600), ' hours ', FLOOR(minutes_part / 60), ' minutes ', seconds_part, ' seconds' ) AS difference
FROM
  differences
```
2. Calculating how many trips were made by each of the two user groups:
``` sql
SELECT
  member_casual,
  COUNT (*) AS total_rides
FROM
  `bike-share-case-study-410714.TripData.trip_data`
GROUP BY
  member_casual
```
3. Calculating the average trip duration for each of the two groups:
```sql
WITH
  trip_duration_in_minutes AS (
  SELECT
    ride_id,
    started_at,
    ended_at,
    member_casual,
    ride_length,
    TIMESTAMP_DIFF(ended_at, started_at,MINUTE) AS minutes
  FROM
    `bike-share-case-study-410714.TripData.trip_data`)
SELECT
  member_casual,
  AVG(minutes) AS average_trip_duration,
FROM
  trip_duration_in_minutes
GROUP BY
  member_casual
```
4. Calculating how many trips (and their percentage) were made by the two groups, each day, on an average week:
```sql
WITH
  rides_in_specific_day_and_memebertype AS (
  SELECT
    member_casual,
    COUNT (*) AS total_rides_per_day_and_type,
    day_of_week
  FROM
    `bike-share-case-study-410714.TripData.trip_data`
  GROUP BY
    member_casual,
    day_of_week),
  rides_per_specific_day AS (
  SELECT
    COUNT(*) AS total_rides_per_day,
    day_of_week
  FROM
    `bike-share-case-study-410714.TripData.trip_data`
  GROUP BY
    day_of_week)
SELECT
  rides_in_specific_day_and_memebertype.member_casual,
  rides_in_specific_day_and_memebertype.day_of_week,
  rides_in_specific_day_and_memebertype.total_rides_per_day_and_type,
  rides_per_specific_day.total_rides_per_day,
  (rides_in_specific_day_and_memebertype.total_rides_per_day_and_type / rides_per_specific_day.total_rides_per_day)*100 AS ride_percentage
FROM
  rides_in_specific_day_and_memebertype
LEFT JOIN
  rides_per_specific_day
ON
  rides_in_specific_day_and_memebertype.day_of_week = rides_per_specific_day.day_of_week
ORDER BY
  rides_in_specific_day_and_memebertype.day_of_week
```
5. Finding the top 50 start stations for each of the two groups:
``` sql
WITH
  start_count_top50 AS (
  SELECT
    start_station_name,
    COUNT(*) AS start_count
  FROM
    `bike-share-case-study-410714.TripData.trip_data`
  WHERE
    member_casual = "member" #we run the code a second time with member_casual="casual"
  GROUP BY
    start_station_name
  ORDER BY
    start_count DESC
  LIMIT
    50),
  start_lat_lng AS (
  SELECT
    DISTINCT start_station_name,
    start_lat,
    start_lng
  FROM
    `bike-share-case-study-410714.TripData.trip_data`)
SELECT
  start_count_top50.start_station_name,
  start_count_top50.start_count,
  start_lat_lng.start_lat,
  start_lat_lng.start_lng
FROM
  start_count_top10
JOIN
  start_lat_lng
ON
  start_count_top10.start_station_name = start_lat_lng.start_station_name
```

### Results and Findings

1. The annual members have made over 2.5 times more trips than the
casual users. These numbers may indicate that annual members use the company’s
bikes,primarily for their everyday transports.

2. The average trip of the casual user lasts 5 times longer than the one of
the annual member. This is a strong indication that casual members use the bike
share service for long rides, for leisure and entertainment purposes.

3. On weekends and especially on Sunday, the casual users’ trips represent
a much larger percentage of the total trips in contrast with what happens midweek. The above information can be summarized in the following graph, that enable us to
extract some useful trends that govern the habits of the two user groups.

Summarized insight (summury of the 3 previous insights): The annual members make many trips that
last a relatively small amount of time. These data indicate that annual members
use the company's bikes,mainly, for everyday transports such as commuting to
work or going for their groceries. This is also backed by the fact that we do not see
considerable differences in the average trip duration during the week. On the
other hand, casual users made a lot less trips, but for the better part of an average
week, their trips last a lot longer. A longer bike trip has more probabilities to be a
leisure activity than a way to complete an everyday task. This observation is also
supported by the fact, that we see large differences in trip duration throughout the
week.

4. We can see that the two user groups select a similar general location to
start their bike trips. A close look at the heat maps reveals, that the points of
interest for the casual users are not that much concentrated. On the contrary, they
are spread over a wider area in the City of Chicago. This observation may show a
tendency for city exploration. Yet another fact that supports the theory that the
casual users use the bikes for entertainment purposes.

### Recomendations
The main business goal is to convert casual users to annual members. A potential
effective way to achieve this goal is to offer annual membership perks and bonuses
that are tailor made for the habits of the average casual user. Some of our top
recommendations are the following.

1. Bonus and perks that rewards longer trips: As we saw, casual users tend to make
longer bike trips. The company can offer bonuses when the user makes trips that
exceed a specified period of time. For example, if a member makes many long trips, gets a discount in the next year membership.

2. Rewarding city exploration: We established that the majority of the casual users
select a larger variety of areas to start their bike trips. The company can offer
bonuses for each user that visits over a specified number of bike stations. This idea
could be also be supported with a social media page, where users could upload
content of their bike exploration of Chicago.

3. Focus on specific days: If a casual member can not be persuaded to buy the
standard annual membership, the company could offer them slightly cheaper annual
memberships, that grants unlimited rides only for specific days. The company can
focus on days such as Saturday and Sunday, when the percentage of the rides made
by casual users is higher.

### Refrences
1. Big Query Sandbox
2. [Google Data Analytics professional Certificate on coursera](https://www.coursera.org/professional-certificates/google-data-analytics)
3. Anaconda Navigator/Spyder









