# **Boosting Bellabeat Growth: How Fitbit App User Data Can Drive User Engagement and Acquisition**
## Background
Launched in 2013, Bellabeat focuses on creating stylish health-focused smart devices for women. Their mission is to inspire and educate women about their health through user-friendly technology. Urška Sršen, Bellabeat CEO, believes studying how people use other fitness trackers, like Fitbit, can uncover new ways for Bellabeat to succeed
## Business task
This case study proposes to analyze user interaction data from the Fitbit app to identify trends. These insights are intended to develop targeted marketing strategies, ultimately driving user acquisition and business growth for Bellabeat.
## Stakeholders 
- Urška Sršen: Bellabeat’s co-founder and Chief Creative Officer.
- Sando Mur: Bellabeat’s co-founder and mathematician.
- Bellabeat marketing analytics team.
## Dataset
Kaggle datasets download -d arashnic/fitbit CC0: Public Domain
## Tools
- Microsoft Excel
- R programming, 
- OpenAI- ChatGPT.

## 1. Asking questions
- What are the emerging trends in smart device usage?
- How can Bellabeat leverage these trends to better serve its customers?
- How can these trends inform and influence Bellabeat's marketing strategy?

## 2. Prepare data
After retrieving and analyzing the main characteristics of the dataset, can be inferred that: 
- The data set presents a total of 18 files with details about 30 participants' activities tracked through the Fitbit app.
- Limited sample: the dataset reflects data from a limited number of participants.
- Data: the data is outdated (2016) and includes a range of activities from one month (March to May) 
- For this case study will be used 3 files containing the daily activity summary, Sleep time summary, and Weight summary. 

## 3. Processing data
3.1. A first data screening and preparation was performed using Microsoft Excel:
  - Basic data cleaning was performed checking data type, validity of ranges, blank cells, and duplicate data.
  - Standardized date format: Data was in date/hour format. Using the split function, a single date format was obtained. 
  - Data set merge: Using the vlookup () function, information about sleep time and weight was merged in the daily activity summary file. 
  
3.2. Launching R studio Club and installing needed packages to perform data process:
```
install.packages("tidyverse")
library(tidyverse)
install.packages("janitor")
library (janitor)
install.packages("ggplot2")
library(ggplot2)
install.packages("skimr")
library(skimr)
```
3.3. Loading dataset
```
daily_activity_merged <-read.csv("dailyActivity_merged.csv")
````
3.4. Preview the dataset and check for unique values
```
View(daily_activity_merged)
n_distinct(daily_activity_merged$Id)
```
    We got 33 unique, values that match the dataset description.
3.5. Change data type for date and totalMinutesAsleep
```
daily_activity_merged$ActivityDate <- as.Date (daily_activity_merged$ActivityDate, format = '%m/%d/%Y')
daily_activity_merged$TotalMinutesAsleep <- as.integer(daily_activity_merged$TotalMinutesAsleep)
daily_activity_merged$WeightKg <-as.numeric(daily_activity_merged$WeightKg)
```
3.6. Checking new data types
```
str(daily_activity_merged)
View(daily_activity_merged)
```
3.7. Select and transfer key column/data
```
daily_activity_merged %>% 
  select(Id, ActivityDate, TotalSteps, TotalDistance, VeryActiveMinutes,FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes,Calories, WeightKg)
daily_activity_condensed <- daily_activity_merged %>% 
  select(Id, ActivityDate, TotalSteps, TotalDistance, VeryActiveMinutes,FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes,Calories,TotalMinutesAsleep, WeightKg)
```
3.8. Check for missing values
```
daily_activity_condensed %>% 
  select(Id, ActivityDate, TotalSteps, TotalDistance, VeryActiveMinutes,FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes,Calories,TotalMinutesAsleep, WeightKg) %>%
  filter(complete.cases(.))
```
  We have only 67 complete cases (vs 940 obs). But since the only column with missing data is Weight, we will keep it all. 

## 4. Analysing data

4.1. Generate summary
```
summary(daily_activity_condensed)
```
- The average daily steps is about 7,638 steps, which is less than the 10,000 steps suggested. 
- The average active minutes were 21.16, versus 13.56 minutes on average on fair activity
- Most people stay on average 193 minutes in light activity and an average of 991 minutes in sedentary activity. 
- About the calories burned per day, the average is about 2,304 
- The average time sleeping is 419 min or 6.9 hours, wich is... 
- Average weight is 72 kg, winch is under...

4.2.  Find out more active weekday
- First, obtain the Day of the week from the dates and arrange the results
```
daily_activity_condensed <- daily_activity_condensed %>%
  mutate(WeekDay = weekdays(ActivityDate))
daily_activity_condensed$WeekDay <- factor(daily_activity_condensed$WeekDay, levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))
```
- Create geom_bar to see steps vs day of the week
```
ggplot(data= daily_activity_condensed, mapping= aes(x= WeekDay, y= TotalSteps))+  
  geom_bar(stat="identity", fill="steelblue",width = 0.7)+ 
  theme_minimal() + 
  labs(title="Total Steps per Day") +
  theme(plot.title = element_text(hjust = 0.5))
```
4.3 Find out the day of the week with the most calories burned
```
ggplot(data= daily_activity_condensed, mapping= aes(x= WeekDay, y= Calories)) +
  geom_bar(stat="identity", fill="steelblue", width = 0.7) +
  theme_minimal() +
  labs(title="Total Calories per Day") +
  theme(plot.title = element_text(hjust = 0.5)) +
 scale_y_continuous(labels = scales::comma)
```
4.4 Find out the day of the week with the most sleeping time
- Convert minutes to hours 
```
daily_activity_condensed <- daily_activity_condensed %>%
  mutate(TotalHoursAsleep = TotalMinutesAsleep/60)
```
- Calculate total sleep hours per day
```
daily_totals <- daily_activity_condensed %>%
  group_by(WeekDay) %>%
  summarise(TotalSleepHours = sum(TotalHoursAsleep))
  ```
- Generating Total Sleep Time
  ```
  ggplot(data=daily_activity_condensed, aes(x=WeekDay, y=TotalHoursAsleep, fill=Weekday))+ 
  geom_bar(stat="identity", fill="steelblue")+
  labs(title="Total Minutes Asleep During the Week", y="Total Minutes Asleep")
  ```
  4.5 Comparing the distribution of active, fair, light, and sedentary activity. 
  - Find the average for each category and calculate percentages
  ```
  avg_times <- colMeans(daily_activity_condensed[c("VeryActiveMinutes", "FairlyActiveMinutes","LightlyActiveMinutes", "SedentaryMinutes")])
  total_time <- sum(avg_times)
  percentages <- round(avg_times / total_time * 100, 1)
  ```
  - Creating data frame with results
  ```
  activity_data <- data.frame(
  activity = c("Very Active", "Fairly Active", "Lightly Active", "Sedentary"),
  time = avg_times, percent= percentages)
  ```
  - Create a pie chart
  ```
  ggplot(activity_data, aes(x = "", y = time, fill = activity)) +
  geom_bar(stat = "identity", width = 1, color = "white") +
  coord_polar("y", start = 0) +
  geom_text(aes(label = paste0(percent, "%"), angle = 80), position = position_stack(vjust = 0.5)) +
  labs(title = "Average Time Spent in Different Activity Levels") +
  theme_minimal() +
  theme(legend.position = "right", plot.title = element_text(hjust = 0.5))

  ```
  
## Correlation between sedentary activity and time asleep
ggplot(data= daily_activity_condensed, aes(x=SedentaryMinutes, y=TotalMinutesAsleep)) + 
  geom_point(colour="blue") + geom_smooth(color = "cadetblue4")+
  labs(title="Sedentary Time vs Time Asleep", x="Sedentary Time (minutes)", y="Time Asleep (minutes)")+ 
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5))
## This plot suggest a the plot suggests that moderate sedentary time is associated with the most sleep, 
## while both low and high amounts of sedentary time are associated with less sleep.
## Correlation between Very Active day and time asleep
ggplot(data= daily_activity_condensed, aes(x=VeryActiveMinutes, y=TotalMinutesAsleep)) + 
  geom_point(colour="orange") + geom_smooth(color = "cadetblue4")+
  labs(title="Active Time vs Time Asleep", x="Active Time (minutes)", y="Time Asleep (minutes)")+ 
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5))
## the plot suggests that minimal active time is associated with slightly lower sleep duration, 
##but as active time increases, sleep duration tends to increase as well. 
##However, individual variability is high, and other factors might influence.
## Correlation between calories burned and weight 
ggplot(data= daily_activity_condensed, aes(x=Calories, y=WeightKg)) + 
  geom_point(colour="green") + geom_smooth(color = "cadetblue4")+
  labs(title="Calories burned vs Weight", x="Calories burned(kCal)", y="Weight (KG)")+ 
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5))
## No linear correlation. Moderate to low calories burned, less weight and more calories burned show higher weight. 




