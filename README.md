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

## Content
- [Asking questions](#Asking-questions)
- [Preparing data](#-2.-Preparing-data)
- [Processing data](#-3.-Processing-data)
- [Analyzing data](#-4.-Analyzing-data)
- [Conclusions and suggested actions](#Conclusions-and-suggested-actions)


## 1. Asking questions
- What are the emerging trends in smart device usage?
- How can Bellabeat leverage these trends to better serve its customers?
- How can these trends inform and influence Bellabeat's marketing strategy?

## 2. Preparing data
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
We got 33 unique values that match the dataset description.
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

## 4. Analyzing data

4.1. Generate summary
```
summary(daily_activity_condensed)
```
![1  Summary](https://github.com/user-attachments/assets/33944763-3fe5-4c38-a437-54e97749ecc0)
- The average daily steps is 7,638 steps, which is less than the 10,000 steps suggested by many health organizations. 
- The average active minutes were 21.16, and for fair activity, the average was 13.56 minutes, which is somehow among Health Canada's recommendations ( at least 21.4 minutes of moderate-to-vigorous physical activity per day)
- Most people stay on average 193 minutes in light activity and an average of 991 minutes in sedentary activity. 
- About the calories burned per day, the average is about 2,304 Kcal. 
- The average time sleeping is 419 min or 6.9 hours, which is less that The National Sleep Foundation recommends (7-9 hours of sleep per night for adults). 
- The average Weight is 72 kg. Healthy weight depends on height and body composition, which are data not included in this dataset.


4.2.  Find out more active day
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
![2  Steps vs day](https://github.com/user-attachments/assets/2148c960-b817-41e3-9851-7dc55bf10a58)

4.3. Find out the day of the week with the most calories burned
```
ggplot(data= daily_activity_condensed, mapping= aes(x= WeekDay, y= Calories)) +
  geom_bar(stat="identity", fill="steelblue", width = 0.7) +
  theme_minimal() +
  labs(title="Total Calories per Day") +
  theme(plot.title = element_text(hjust = 0.5)) +
 scale_y_continuous(labels = scales::comma)
```
![3  Total Calories per Day](https://github.com/user-attachments/assets/a6b0592c-8edd-4581-a04b-3834d17d4e56)

4.4. Find out the day of the week with the most sleeping time
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
![4  Total time asleep](https://github.com/user-attachments/assets/f805e04d-c166-4497-b79f-58c4b648aa22)

  4.5. Comparing active, fair, light, and sedentary activity distribution. 
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
<img width="793" alt="5  Kind of activity" src="https://github.com/user-attachments/assets/c03fa8d0-7f99-4b08-a8d5-58bce92b378b">

4.6.  Correlation between sedentary activity and time asleep
```
ggplot(data= daily_activity_condensed, aes(x=SedentaryMinutes, y=TotalMinutesAsleep)) + 
  geom_point(colour="blue") + geom_smooth(color = "cadetblue4")+
  labs(title="Sedentary Time vs Time Asleep", x="Sedentary Time (minutes)", y="Time Asleep (minutes)")+ 
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5))
```
<img width="793" alt="6  Sedentary time vs time asleep" src="https://github.com/user-attachments/assets/b3338740-867c-41c7-9c59-11d2db465b17">

This chart shows that participants with moderate sedentary time reached long hours of sleep. On the other hand, both low and high sedentary time creates an impact on total sleep time. 

4.7 Correlation between Active time and total time asleep
```
ggplot(data= daily_activity_condensed, aes(x=VeryActiveMinutes, y=TotalMinutesAsleep)) + 
  geom_point(colour="orange") + geom_smooth(color = "cadetblue4")+
  labs(title="Active Time vs Time Asleep", x="Active Time (minutes)", y="Time Asleep (minutes)")+ 
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5))
```
<img width="793" alt="7  Active time vs sleep" src="https://github.com/user-attachments/assets/a201419f-96e7-4e49-8492-59c573cab1e3">


The plot suggests minimal active time is associated with lower sleep duration, but as active time increases sleep duration tends to increase. However, individual variability is high, and other factors might influence it.

4.8 Correlation between calories burned and weight
```
ggplot(data= daily_activity_condensed, aes(x=Calories, y=WeightKg)) + 
  geom_point(colour="green") + geom_smooth(color = "cadetblue4")+
  labs(title="Calories burned vs Weight", x="Calories burned(kCal)", y="Weight (KG)")+ 
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5))
```
<img width="793" alt="8  Calories vs Weight" src="https://github.com/user-attachments/assets/4e475ab3-5cf0-4552-bef5-528fe32ee402">

This graphic shows a non-linear correlation. Moderate to low calories burned, less weight, and more calories burned show higher weight. 

## 5. Conclusions and suggested actions
- A significant 80% of the participants' regular day is spent in sedentary activities. Only about 4 hours of the day is dedicated to light activity, and even less time (about 30 minutes) is actually dedicated to vigorous activity.
- On Tuesdays, more participants showed a significant increase in steps taken, calories burned, and overall more intense activity. Sundays and Mondays are among the days of the week when participants spend less time being active.
- The relationship between activity levels during the day and sleep duration at night shows that participants with moderate activity levels can achieve better sleep duration. Unver and over activity shows a decrease in sleep time. 
- It is difficult to draw conclusions about the weight of the participants because the data set does not provide the additional information needed to get a clear picture (height, age, etc.).

What can Bellabeat do with these insights?

- Become a coach for customers by shifting from a supporting device role to actively promoting a better lifestyle. This can be achieved through comprehensive educational campaigns that highlight the numerous benefits of incorporating regular activities, such as walking, ensuring adequate sleep, etc. These campaigns should provide practical tips, motivational content, and personalized guidance to help users make sustainable lifestyle changes. By fostering a supportive and informative environment, customers will be more likely to adopt healthier habits.
- Create a robust points or incentive system to reward positive changes in customer behavior. Implement a structured program where users earn points for tracking their daily activities, achieving specific health milestones, and consistently engaging with wellness content. These points can be redeemed for rewards such as discounts on products, exclusive memberships, or other perks. This system will not only motivate more people to diligently track their daily activities but also enhance customer engagement and loyalty, ultimately providing the company with a richer dataset to refine and optimize its models.
- Develop and program personalized alerts. These alerts can be tailored based on individual activity patterns and preferences, offering reminders to move, suggestions for quick exercises, or encouragement to stay active. Additionally, the alerts can help users track special events, such as upcoming wellness challenges or community activities, making it easier for them to stay informed and engaged. By providing timely and relevant prompts, the system can effectively support users in maintaining a consistent and active lifestyle




