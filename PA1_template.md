---
title: "Reproducible Research: Peer Assessment 1"
output: html_notebook
---
## Part 1)
## 1. Loading and preprocessing the data
## 1.1. Load the data (i.e. read.csv())
```{r}
setwd("/Users/brecre01/Desktop")
activity <- read.csv("activity.csv",colClasses = c("numeric", "character","integer"))  
```  

## 1.2. Process/transform the data (if necessary) into a format suitable for your analysis
```{r}
dim(activity)
names(activity)
head(activity)
tail(activity)
summary(activity)
str(activity)
library(plyr)
library(dplyr)
library(lubridate)
library(ggplot2)
total.steps <- tapply(activity$steps, activity$date, FUN = sum, na.rm = TRUE)
activity$date <- ymd(activity$date)  
```  

## Part 2)
## 2. What is mean total number of steps taken per day?
### For this part of the assignment, you can ignore the missing values in the dataset.
### 2.1. Calculate the total number of steps taken per day
```{r}
steps <- activity %>%
  filter(!is.na(steps)) %>%
  group_by(date) %>%
  summarize(steps = sum(steps)) %>%
  print  
```  

### 2.2. Make a histogram of the total number of steps taken each day
```{r}
ggplot(steps, aes(x=date, y=steps))+geom_histogram(stat="identity")+ xlab("Dates")+ ylab("Steps")+ labs(title= "Total Numbers of Steps per Day")
```  

### 2.3. Calculate and report the mean and median of the total number of steps taken per day
```{r}
mean(total.steps)
median(total.steps)  
```

## Part 3)
## 3. What is the average daily activity pattern?
```{r}
daily <- activity %>%
        filter(!is.na(steps)) %>%
        group_by(interval) %>%
        summarize(steps=mean(steps)) %>%
        print
```  

### 3.1. Make a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
```{r}
plot(daily, type = "l")
```  


### 3.2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r}
daily[which.max(daily$steps), ]$interval  
```  

## Part 4
## 4. Imputing missing values
### Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.
### 4.1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
```{r}
missing <- sum(is.na(activity))
```  

### 4.2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
### 4.3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
```{r}
new <- activity %>%
        group_by(interval) %>%
        mutate(steps = ifelse(is.na(steps), mean(steps, na.rm=TRUE), steps))
summary(new)
```  

### 4.4a. Make a histogram of the total number of steps taken each day
```{r}
new.steps <- new %>%
  group_by(date) %>%
  summarize(steps = sum(steps)) %>%
  print 
ggplot(new.steps, aes(x=date, y=steps))+geom_histogram(stat="identity")+ xlab("Dates")+ ylab("Imputed Steps")+ labs(title= "Total numbers of Steps per day (missing data imputed)")
```  

### 4.4b. Calculate and report the mean and median total number of steps taken per day.
```{r}
imputed.steps <- tapply(new$steps, new$date, FUN = sum, na.rm = TRUE)
new$date <- ymd(new$date)
mean(imputed.steps)
median(imputed.steps)  
```  

### 4.4c. Do these values differ from the estimates from the first part of the assignment?
```{r}
mean(total.steps)==mean(imputed.steps)
median(total.steps)==median(imputed.steps)
summary(total.steps)
summary(imputed.steps)  
```  
### 4.4d. What is the impact of imputing missing data on the estimates of the total daily number of steps? 
### Answer to 2.3.4d.) The estimates of the number of steps increased by 41, 3041, 370, 1416, 0, 0.
```{r}
summary(imputed.steps) - summary(total.steps)
par(mfrow=c(2,1))
hist(imputed.steps,col="red")
hist(total.steps,col="blue")  
```  

## Part 5)
## Are there differences in activity patterns between weekdays and weekends?
### For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part. 
### 5.1 Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
```{r}
dayofweek <- function(date) {
    if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
        "weekend"
    } else {
        "weekday"
    }
}
new$daytype <- as.factor(sapply(new$date, dayofweek))  
```    

### 5.2 Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.
```{r}
par(mfrow = c(2, 1))
for (type in c("weekend", "weekday")) {
    steps.type <- aggregate(steps ~ interval, data = new, subset = new$daytype == 
        type, FUN = mean)
    plot(steps.type, type = "l", main = type)
}
```  


```{r}
sessionInfo()
```
