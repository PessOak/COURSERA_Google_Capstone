# COURSERA_Google_Capstone_Bellabeat
This repository contains the capstone project from Google Data Analytics Professional Certificate course at Coursera.
The Case Study follows the 6 Steps Data Analysis Process taught throughout the Course

## Introduction
In recent years, wearable devices have undergone a remarkable metamorphosis, evolving from basic step counters into comprehensive health and lifestyle companions. These smart gadgets now offer an array of functionalities, from heart rate monitoring to sleep analysis and even stress management. 

As we gaze into the future, the trajectory of wearables appears set to further blur the lines between technology and human experience. Forecasts suggest real-time health diagnostics, proactive disease prevention, and the integration of augmented and virtual reality into our daily lives. 

Furthermore, wearables are forging stronger connections with the broader ecosystem of smart devices. The fusion of wearables with the Internet of Things (IoT) promises seamless, synchronized living.

The graph below shows the growth of the wearable devices market in recent years and projections for the future

![image](https://github.com/PessOak/COURSERA_Google_Capstone/assets/105830948/4c754216-582a-4178-8ed0-452a882a2db6)

Source: Gran View Research Inc. (https://www.grandviewresearch.com/). Sample Report: "Wearable Technology Market Analysis, 2022 - Pg.43"

## 1- Ask
Business tasks: 

a) Identify the business task -> The task is to transform raw smart device usage data into actionable insights that can guide Bellabeat's marketing strategy, enhancing their product positioning and communication efforts in a way that aligns with current consumer preferences and behaviors in the smart device market.

b) Consider key stakeholders -> Urška Sršen (chief creative officer) and Sando Mur are the primary stakeholders. And Bellabeat’s marketing analytics team are the secondary stakeholders.

## 2- Prepare
There are 18 CSV files available from FitBit Fitness Tracker Data (CC0: Public Domain, dataset made available through Mobius) that can be used for the project. This Kaggle dataset contains personal fitness tracker from 33 Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes information about daily activity, steps, and heart rate that can be used to explore users’ habits. The files can be found on the following link: https://www.kaggle.com/datasets/arashnic/fitbit  

We will use SQL as our main tool for the Process and Analyze phases. As for the Visualization phase we will be using Tableau to create customized graphs.

Main steps of the Prepare phase:
-	Open and look into each dataset file;
-	Choose the useful datasets for the analysis;
-	Upload the chosen files into BigQuery (SQL);
-	Some files returned a datetime format error when trying to upload them into BigQuery, but after I did some datetime formatting in google sheets the upload worked out;
-	Files with metrics in minutes and seconds weren’t selected for the analysis. As we are looking for trends, only files containing metrics with day and hours will be used for this project;
-	The files were renamed, the "dailyActivity_merged.csv" is now "activity_merged.csv"; the "dailyCalories_merged.csv" is now "calories_merged.csv"; the "dailyIntensities_merged.csv" is now "intensities_merged.csv"; the "sleepDay_merged.csv" is now "sleep_day_merged.csv"; the "weightLogInfo_merged.csv" is now "weight_info.csv"; the "dailySteps_merged.csv" is now "steps_merged.csv"
-	Except for the weight_info (67 rows) and sleep_day_merged (413 rows) datasets there are 940 rows of data in each selected dataset;
```sql
-- This basic query can be used to give us a general idea of the file. It has 940 rows of data, all columns contain numeric data and except for the "weight_info" and "sleep_day_merged" datasets there are 940 rows of data in each dataset.
SELECT
  *
FROM
  `elegant-atom-395419.bellabeat.activity_merged`
```
-	After a quick look into each dataset, we can observe that all of them have 1 column in common, the “Id” column. And the “activity_merged” file seems to contain the same data as the files “calories_merged”, “intensities_merged” and “steps_merged”.
-	As we'll be using the ID column as reference for comparison with other datasets I will just check if all IDs have the same amount of numerals on the "activity_merged" file:
```sql
-- With this query we can check if there is any ID with more or less than 10 numerals, there is not.
SELECT  
  *
FROM 
  `elegant-atom-395419.bellabeat.activity_merged` 
WHERE
  LENGTH(CAST(Id AS STRING)) <> 10
```
-	Now, I will check with some queries if the data in those datasets match. If they don’t, I will use JOIN statements to join the smaller datasets into the large one (activity_merged). If they have the same data, I will just use the large one and remove the others.
- First I check calories x activity:
```sql
-- This query checks if the data from the "calories_merged" dataset is already inside the "activity_merged" dataset. As it does not return any data it means that the first one is already part of the second one.
SELECT
  calories.Id,
  calories.ActivityDay AS activity_date,
  calories.Calories AS calories
FROM
  `elegant-atom-395419.bellabeat.calories_merged` AS calories
LEFT JOIN
  `elegant-atom-395419.bellabeat.activity_merged` AS daily_activity
ON
  calories.Id = daily_activity.Id AND 
  calories.ActivityDay = daily_activity.ActivityDate AND 
  calories.Calories = daily_activity.Calories
WHERE
  daily_activity.Id IS NULL OR
  daily_activity.ActivityDate IS NULL OR
  daily_activity.Calories IS NULL
```
- Than I check intensities x activity:
```sql
-- This query checks if the data from the "intensities_merged" dataset is already inside the "activity_merged" dataset. As it does not return any data it means that the first one is already part of the second one.
SELECT
  intensities.Id,
  intensities.ActivityDay AS activity_date,
  intensities.SedentaryMinutes AS sedentary_mins,
  intensities.SedentaryActiveDistance AS sedentary_dist
FROM
  `elegant-atom-395419.bellabeat.intensities_merged` AS intensities
LEFT JOIN
  `elegant-atom-395419.bellabeat.activity_merged` AS daily_activity
ON
  intensities.Id = daily_activity.Id AND 
  intensities.ActivityDay = daily_activity.ActivityDate AND 
  intensities.SedentaryMinutes = daily_activity.SedentaryMinutes AND 
  intensities.SedentaryActiveDistance = daily_activity.SedentaryActiveDistance
WHERE
  daily_activity.Id IS NULL OR 
  daily_activity.ActivityDate IS NULL OR 
  daily_activity.SedentaryMinutes IS NULL OR 
  daily_activity.SedentaryActiveDistance IS NULL
```
- And lastly I check steps x activity:
```sql
-- This query checks if the data from the "steps_merged" dataset is already inside the "activity_merged" dataset. As it does not return any data it means that the first one is already part of the second one.
SELECT
  steps.Id,
  steps.ActivityDay AS activity_date,
  steps.StepTotal AS steps
FROM
  `elegant-atom-395419.bellabeat.steps_merged` AS steps
LEFT JOIN
  `elegant-atom-395419.bellabeat.activity_merged` AS daily_activity
ON
  steps.Id = daily_activity.Id AND 
  steps.ActivityDay = daily_activity.ActivityDate AND 
  steps.StepTotal = daily_activity.TotalSteps
WHERE
  daily_activity.Id IS NULL OR 
  daily_activity.ActivityDate IS NULL OR 
  daily_activity.TotalSteps IS NULL
```
- After comparing the datasets I've seen that the steps, intensities and calories datasets are already merged into the activity_merged one. So I will just worj with the activity_merged and remove the other three datasets.
- Next I will use a query  to check how many unique entries there are for activity, we will also know how many users provided their data for the research:
```sql
-- With this query we can discover the total of individuals that provided their data for the research
SELECT
DISTINCT
  Id
FROM
  `elegant-atom-395419.bellabeat.activity_merged`
```
- Now checking how many entries there are for sleep:
```sql
-- With this query we can check the number of users info for sleep, there are 24 rows, the sample size is rather small but we will keep it for now, it might be useful
SELECT
  DISTINCT Id
FROM 
  `elegant-atom-395419.bellabeat.sleep_day_merged`
```
- And now checking how many entries there are for weigth:
```sql
-- With this query we can check how many values there are for weigth, as we can see there are only 8 rows, due to the small sample size we won't be using this dataset for I don't believe it will bring useful and meaningful insights for the project. 
SELECT
  DISTINCT Id
FROM 
  `elegant-atom-395419.bellabeat.weight_info`
```

## 3- Process

Data limitations observed:

-	The total of users that provided data is 33, it’s a small sample size, because of that, the final results and findings may not be very reliable or useful;
-	The data is from 2016 and this project is happening in 2023. So, it might be outdated and may not reflect the real scenario nowadays;
-	It would be useful to have gender information for this project for the company’s (Bellabeat) main audience and consumers are women in general, but we don’t have that.


