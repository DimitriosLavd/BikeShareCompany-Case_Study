# BikeShareCompany-Case_Study
An -one week- data analysis case study for a bike share company. Includes Python and SQL codes. 

### Project Overview-case study scenario
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

### Data scource 
Trip Data: Our primary data scource is the "202004-divvy-tripdata.CSV", that contains detailed information about the trips made using the company's bikes, for 12 months.

### Tools 
- Google sheets: Data cleaning and visualization
- WPS Sheets: Data cleaning
- Python: Data cleaning and manipulation
- BigQuery (SQL): Data analysis
- Tableau: Data visualization
- 
### Data Cleaning/Preparation
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








