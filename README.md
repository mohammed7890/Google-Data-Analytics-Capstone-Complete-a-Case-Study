# Google Data Analytics Capstone Project Case Study 1

# Introduction 
 You are a junior data analyst working on the marketing analyst team at Cyclistic, a bike-share
 company in Chicago. The director of marketing believes the company’s future success
 depends on maximizing the number of annual memberships. Therefore, your team wants to
 understand how casual riders and annual members use Cyclistic bikes di erently. From these
 insights, your team will design a new marketing strategy to convert casual riders into annual
 members. But first, Cyclistic executives must approve your recommendations, so they must be
 backed up with compeling data insights and professional data visualizations.

# ASK

Business Task:
Design marketing strategies aimed at converting casual riders into annual members.
Business Questions:
1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?

I am assigned to answer the first question, and thus my analysis will be focused on comparing the annual members and casual riders.

Key Stakeholders:

1. Lily Moreno — Director of the marketing team and my manager.

2. Cyclistic executive team

# Prepare

To conduct a thorough analysis, we have to obtain the Cyclistic historical data trips from June 2024 to Dec 2024

Upon further examination, it is observed that all the files possess the same 13 columns or attributes, and the data types are appropriate and consistent throughout. These columns are as follows:

1. ride_id
2. rideable_type
3. started_at
4. ended_at
5. start_station_name
6. start_station_id
7. end_station_name
8. end_station_id
9. start_lat
10. start_lng
11. end_lat
12. end_lng
13. member_casual

## Process
To clean, analyze, and aggregate the large amount of monthly data stored in the folder, we will be using Jupyter — an integrated development environment designed for the programming language Python.

Next, we will be loading all the packages needed to perform the data manipulation.

import os
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import datetime 
import time

data_file_folder = r"E:\Courses\Google\8.Google Data Analytics Capstone Complete a Case Study\Module 2\Datasets\202401-202412-divvy-tripdata"

df = []
for file in os.listdir(data_file_folder):
    if file.endswith('.csv'):
        print('loading file {0}....'.format(file))
        df.append(pd.read_csv(os.path.join(data_file_folder,file)))

# concat the files
df_master = pd.concat(df)

# check the length of the dataset 
len(df_master)

df_master.head()

df_master.dtypes

df_master['started_at'] = pd.to_datetime(df_master['started_at'], errors='coerce')
df_master['ended_at'] = pd.to_datetime(df_master['ended_at'], errors='coerce')

Check the missing values

df_master.isnull().sum()

# check if the missing data are in same row
df_master[df_master['start_station_name'].isnull()]

Remove the Missing Values

df_cleaned=df_master.dropna()

# Check the missing values again
df_cleaned.isnull().sum()

# check the unique ride_id count
df_cleaned['ride_id'].nunique()

# check the unque rideable_type values 
df_cleaned['rideable_type'].unique()

# check the unque member_casual values 
df_cleaned['member_casual'].unique()

# create a date, year, quarter and a month column
df_cleaned['date_of_the_ride'] = df_cleaned['started_at'].dt.date
df_cleaned['quarter'] = df_cleaned['started_at'].dt.quarter
df_cleaned['month'] = df_cleaned['started_at'].dt.month
df_cleaned['hour'] = df_cleaned['started_at'].dt.hour


# check the df again
df_cleaned.head()

# convert the month number into names
df_cleaned['month'] = pd.to_datetime(df_cleaned['month'], format='%m').dt.month_name()

# convert the day number into names
df_cleaned['day_of_week'] = df_cleaned['started_at'].dt.day_name()

#column for time the trip started
df_cleaned['time'] = df_cleaned['started_at'].dt.strftime('%H:%M')

#column for trip duration in min
df_cleaned['trip_duration_in_min'] = (df_cleaned['ended_at'] - df_cleaned['started_at']).dt.total_seconds().div(60).astype(float)

df_cleaned.head(5)

len(df_cleaned)

# Remove the rows with trip_duration that smaller than 0
df_cleaned = df_cleaned[df_cleaned['trip_duration_in_min'] >= 0]

len(df_cleaned)

df_cleaned.dtypes

#dropping rows containing Tests in 'start_station_name' column

index_num = df_cleaned[df_cleaned['start_station_name'].str.contains("TEST" or "test" or "Test")==True].index
all_trips = df_cleaned.drop(index_num)

len(all_trips)

It is important to make sure that customer_type column has only two distinct values. Let's confirm the same.

pd.DataFrame(all_trips['member_casual'].value_counts()).rename(columns={'customer_type':'No. of Customers'})

all_trips.drop(['start_station_id','end_station_id','ride_id'],axis=1,inplace=True)

all_trips.head()

Rename some columns to be more readable

all_trips.rename(columns={'rideable_type':'bike_type','started_at':'start_time','ended_at':'end_time','member_casual':'membership'}, inplace=True)

all_trips.head()

Creating a csv file of the clean data for futher analysis or visualizations in other tools like SQL, Tableau, Power BI, etc.

all_trips.to_csv('all_trips.csv',index=False,header=True)

## Analyze
The dataframe is now ready for descriptive analysis that will help us uncover some insights on how the casual riders and members use Cyclistic rideshare differently

all_trips= pd.read_csv('all_trips.csv')

#statistical summary of 'trip_duration_in_min' column

pd.DataFrame(all_trips['trip_duration_in_min'].describe().apply(lambda x: format(x, 'f')))

#statistical summary of 'trip_duration_in_min' of Members column

pd.DataFrame(all_trips.loc[all_trips['membership'] == 'member', 'trip_duration_in_min'].describe().apply(lambda x: format(x,'f'))).rename(columns={'trip_duration_in_min':'Members'})

#statistical summary of 'trip_duration_in_min' of Casuals column

pd.DataFrame(all_trips.loc[all_trips['membership'] == 'casual', 'trip_duration_in_min'].describe().apply(lambda x: format(x,'f'))).rename(columns={'trip_duration_in_min':'Casuals'})

The mean trip duration of member riders is lower than the mean trip duration of all trips.

Number of trips by casual customers and members

all_trips['membership'].value_counts()

There are Members rented more bikes than Casual in 2024

Top 5 station where riders started

#Top 5 stations by trip count
all_trips.groupby('start_station_name')['start_time'].nunique().nlargest(5)

We can clearly see that Streeter Dr & Grand Ave has the greatest number of starts of 62124 in 2024.

Total Number of trips per quarter

# totlal trips per quarter

all_trips['quarter'].value_counts()

The third quarter has the most numbers of rides

## Share

Show numbers of trips per month

plt.hist(all_trips['day_of_week'],edgecolor='black')
plt.title('Number of Trips Every Day')
plt.xlabel('Days')
plt.ylabel('Number of Trips')

plt.hist(all_trips['month'],edgecolor='black')
plt.title('Number of Trips Every Month')
plt.xlabel('Months')
plt.ylabel('Number of Trips')
plt.legend()
plt.show()

July, Augst and september have the most tirps in 2024

plt.hist(all_trips['membership'], edgecolor='black', alpha=0.7, color='blue')

# Add titles and labels
plt.title('Compare Between Memeberships')
plt.xlabel('Membership')
plt.ylabel('Number of Trips')
plt.legend()
plt.show()

member riders is the most number of riders in 2024

member_trips=all_trips[all_trips['membership']=='member']
casual_trips=all_trips[all_trips['membership']=='casual']


plt.hist(member_trips['day_of_week'], bins=30, edgecolor='black', alpha=0.7, color='blue', label='member')
plt.hist(casual_trips['day_of_week'], bins=30, edgecolor='black', alpha=0.7, color='red', label='casual')

# Add titles and labels
plt.title('Compare between Casual and Member')
plt.xlabel('Days')
plt.ylabel('Number of Trips')
plt.legend()
plt.show()

From this histogram we can see member riders are rides the bikes all days but the casual using bikes mostly in weekends

plt.hist(member_trips['bike_type'], bins=30, edgecolor='black', alpha=0.7, color='blue', label='member')
plt.hist(casual_trips['bike_type'], bins=30, edgecolor='black', alpha=0.7, color='red', label='casual')

# Add titles and labels
plt.title('Compare between Casual and Member')
plt.xlabel('Bike Type')
plt.ylabel('Number of Trips')
plt.legend()
plt.show()

Most annual members use the classic bike in their rides and they don't use the electric scooter

## Act
#How do annual members and casual riders use Cyclistic bikes differently?
#1.Annual members use cyclistic bickes all days but the casual riders use it most in weekends
#2.The annual members don't use the electric scooters otherwise the casual riders use all types of bikes
