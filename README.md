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
-	The files were renamed, the "dailyActivity_merged.csv" is now "activity_merged.csv"; the "dailyCalories_merged.csv" is now "calories_merged.csv"; the "dailyIntensities_merged.csv" is now "intensities_merged.csv"; the "sleepDay_merged.csv" is now "sleep_day_merged.csv"; the "weightLogInfo_merged.csv" is now "weight_info.csv"; the "dailySteps_merged.csv" is now "steps_merged.csv"; the "hourlySteps_merged" is now "steps_hour_converted".
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
-- With this query we can discover the total of individuals that provided their data for the research.
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
- With this query we can check for duplicates in the activity_merged table:

```sql
-- This query can be used to check for duplicates in the activity_merged table, there seems to be no duplicates
SELECT DISTINCT *
FROM `elegant-atom-395419.bellabeat.activity_merged`
```
- And with this query we can gather some insights about the steps count of the users:
```sql
-- Using this query we can understand that, on average, each user walks 7638 steps per day.
SELECT
  AVG(TotalSteps) AS average_steps
FROM 
  `elegant-atom-395419.bellabeat.activity_merged`
```
- Using this query we can clean the "sleep_day_merged table" and create a new one, the "clean_sleepday_merged"
```sql
-- This query creates a new table with removed duplicates from the table sleep_day_merged
CREATE OR REPLACE TABLE `elegant-atom-395419.bellabeat.clean_sleepday_merged` AS
SELECT DISTINCT *
FROM `elegant-atom-395419.bellabeat.sleep_day_merged` ;
```
- We can run a query to gather some insights about the "clean_sleepday_mergeg" table:
```sql
-- This query shows the average sleep time of the users wich is 419 minutes (6.98 hours);
-- Also the average bed time wich is 458 minutes (7.63 hours)
-- And also, that each user, on average, stays on bed for almost 39 minutes with no sleep, daily. Later we will check if the activity level has any impact on this.
SELECT
  ROUND(AVG(TotalTimeInBed)) AS avg_bed_time,
  ROUND(AVG(TotalMinutesAsleep)) AS avg_sleep_time,
  ROUND(AVG(TotalTimeInBed) - AVG(TotalMinutesAsleep)) AS avg_awake_time_on_bed
FROM
  `elegant-atom-395419.bellabeat.clean_sleepday_merged`;
```

- Now We will merge the "clean_sleep_day_merged" table with the "activity_merged", but first we need to rename a column:
```sql
-- This query is being used to change the name of one column in the sleep_day_merged table, so we can merge it with the activity_merged table using the columns ID and ActivityDate as the common columns
ALTER TABLE `elegant-atom-395419.bellabeat.sleep_day_merged`
RENAME COLUMN SleepDay TO ActivityDate;
```

- Now we can merge the 2 tables and save the merged table in BigQuery as a new file named "Activity_Sleep_table":
```sql
-- With this query we can join the tables activity_merged and clean_sleepday_merged into one single table. We use the common columns (Id and ActivityDate) between them to join the data
SELECT
  t1.Id,
  t1.ActivityDate,
  t1.TotalSteps AS TotalSteps,
  t1.TotalDistance AS TotalDistance,
  t1.TrackerDistance AS TrackerDistance,
  t1.LoggedActivitiesDistance AS LoggedActivitiesDistance,
  t1.VeryActiveDistance AS VeryActiveDistance,
  t1.ModeratelyActiveDistance AS ModeratelyActiveDistance,
  t1.LightActiveDistance AS LightActiveDistance,
  t1.SedentaryActiveDistance AS SedentaryActiveDistance,
  t1.VeryActiveMinutes AS VeryActiveMinutes,
  t1.FairlyActiveMinutes AS FairlyActiveMinutes,
  t1.LightlyActiveMinutes AS LightlyActiveMinutes,
  t1.SedentaryMinutes AS SedentaryMinutes,
  t1.Calories AS Calories,
  t2.TotalSleepRecords AS TotalSleepRecords,
  t2.TotalMinutesAsleep AS TotalMinutesAsleep,
  t2.TotalTimeInBed AS TotalTimeInBed
FROM
  `elegant-atom-395419.bellabeat.activity_merged` AS t1
LEFT JOIN
  `elegant-atom-395419.bellabeat.clean_sleepday_merged` AS t2
ON
  t1.Id = t2.Id
  AND t1.ActivityDate = t2.ActivityDate;
```

-
## 4- Analysis
- Using this query we can add a new column to the "Activity_Sleep_table" that classifies the users into 4 types based on the amount of steps they walk daily: "sedentary", "lightly active", "fairly active" and "very active". We will now be using the resulting table from the query and name it "Activity_Sleep_table_02":
```sql
--This query adds a new column to the Activity_Sleep_table with some conditions:
SELECT
  *,
  CASE
    WHEN TotalSteps > 0 AND TotalSteps < 5000 THEN 'sedentary'
    WHEN TotalSteps >= 5000 AND TotalSteps < 7499 THEN 'lightly active'
    WHEN TotalSteps >= 7499 AND TotalSteps < 9999 THEN 'fairly active'
    WHEN TotalSteps >= 10000 THEN 'very active'
    ELSE 'not_measured' -- Add a default value or handle other cases as needed
  END AS ActivityLevel
FROM
  `elegant-atom-395419.bellabeat.Activity_Sleep_table`
```
- By using this query we can observe the average sleep time by each group and create a visualization for better understanding later.
```sql
-- This query checks the average sleep time in minutes for each activity level group
SELECT
    ActivityLevel,
    AVG(TotalMinutesAsleep) AS AvgSleepTime
FROM
    `elegant-atom-395419.bellabeat.Activity_Sleep_table_02`
GROUP BY
    ActivityLevel
ORDER BY
    ActivityLevel;
```
- This...
```sql
-- This query can be used to select the desired columns to create a visualization that can show the correlation between TotalSteps and Calories
SELECT 
  Id,
  ActivityDate,
  TotalSteps,
  Calories
FROM 
  `elegant-atom-395419.bellabeat.activity_merged`
```
- This query can show the average caloric expenditure by each group:
```sql
--This query returns the average calories burned by each activity level group
SELECT
    ActivityLevel,
    AVG(Calories) AS AvgCaloriesBurned
FROM
    `elegant-atom-395419.bellabeat.Activity_Sleep_table_02`
GROUP BY
    ActivityLevel
ORDER BY
    ActivityLevel;
```
- This one can be used to check the avergae daily steps from each activity level group and create some viz
```sql
-- With this query we can check the avergae daily steps from each activity level group
SELECT
    ActivityLevel,
    AVG(DailyTotalSteps) AS AvgDailySteps
FROM (
    SELECT
        ActivityLevel,
        ActivityDate,
        AVG(TotalSteps) AS DailyTotalSteps
    FROM
        `elegant-atom-395419.bellabeat.Activity_Sleep_table_02`
    GROUP BY
        ActivityLevel,
        ActivityDate
) AS DailyTotals
GROUP BY
    ActivityLevel
ORDER BY
    ActivityLevel;
```
- Now we'll do some manipulation on the "steps_hour_converted" table:
```sql
-- Using this query we can convert the datetime from the steps_hour_converted file to the format I need by creating a new table using the CREATE OR REPLACE TABLE command.
CREATE OR REPLACE TABLE `elegant-atom-395419.bellabeat.steps_hour_datetime_convert` AS
SELECT
  Id,
  FORMAT_TIMESTAMP('%y/%m/%d %H:%M:%S', ActivityHour) AS ActivityHour,
  StepTotal
FROM
  `elegant-atom-395419.bellabeat.steps_hour_converted`
```
- And now with the "steps_hour_datetime_convert" table we create a new table with an extra column (Weekdays) named "steps_hour_datetime_convert_weekdays":
```sql
-- With this query we create a new table from the "steps_hour_datetime_convert", adding the weekdays as a new column:
CREATE OR REPLACE TABLE `elegant-atom-395419.bellabeat.steps_hour_datetime_convert_weekdays` AS
SELECT
  *,
  CASE EXTRACT(DAYOFWEEK FROM PARSE_DATETIME('%y/%m/%d %H:%M:%S', ActivityHour))
    WHEN 1 THEN 'Sunday'
    WHEN 2 THEN 'Monday'
    WHEN 3 THEN 'Tuesday'
    WHEN 4 THEN 'Wednesday'
    WHEN 5 THEN 'Thursday'
    WHEN 6 THEN 'Friday'
    WHEN 7 THEN 'Saturday'
    ELSE NULL
  END AS Weekday
FROM
  `elegant-atom-395419.bellabeat.steps_hour_datetime_convert`
```
- And this query can be used to show us the average steps by hour of each day on the week, we can use it to build a visualization for better understanding
```sql
-- With this query we can get a table that will be used to build a visualization model later
SELECT
  CASE EXTRACT(DAYOFWEEK FROM PARSE_TIMESTAMP('%y/%m/%d %H:%M:%S', ActivityHour))
    WHEN 1 THEN "Sunday"
    WHEN 2 THEN "Monday"
    WHEN 3 THEN "Tuesday"
    WHEN 4 THEN "Wednesday"
    WHEN 5 THEN "Thursday"
    WHEN 6 THEN "Friday"
    WHEN 7 THEN "Saturday"
  END AS WeekdayName,
  CAST(EXTRACT(HOUR FROM PARSE_TIMESTAMP('%y/%m/%d %H:%M:%S', ActivityHour)) AS INT64) AS Hour,
  AVG(StepTotal) AS AvgSteps
FROM
  `elegant-atom-395419.bellabeat.steps_hour_datetime_convert_weekdays`
GROUP BY
  WeekdayName,
  Hour
ORDER BY
  WeekdayName,
  Hour;
```
- And last but not least:
```sql
-- With this query we can check how many SleepRecords there are for each user and classify them by usage into 4 different types: High, Average, Low and No record. We can see that from the 33 users 9 of them have no SleepRecords and are classified as No record.
SELECT
    Id,
    SUM(TotalSleepRecords) AS TotalRecords,
    CASE
        WHEN SUM(TotalSleepRecords) > 20 THEN "High"
        WHEN SUM(TotalSleepRecords) > 10 AND SUM(TotalSleepRecords) < 21 THEN "Average"
        WHEN SUM(TotalSleepRecords) > 0 AND SUM(TotalSleepRecords) < 11 THEN "Low"
        ELSE "No record"
    END AS Usage
FROM
    `elegant-atom-395419.bellabeat.Activity_Sleep_table_02`
GROUP BY
    Id
ORDER BY
    Id;
```

Data limitations observed:

-	The total of users that provided data is 33, it’s a small sample size, because of that, the final results and findings may not be very reliable or useful;
-	The data is from 2016 and this project is happening in 2023. So, it might be outdated and may not reflect the real scenario nowadays;
-	It would be useful to have gender information for this project because the company’s (Bellabeat) main audience and consumers are women in general, but we don’t have that.

## 5- Sharing (Visualizations)

![Sheet 1](https://github.com/PessOak/COURSERA_Google_Capstone/assets/105830948/1e3ea7b3-9ca5-45e0-a89e-3c4a7fa55332)


