---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

```r
library(ggplot2)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
setwd("~/Documents/Online Courses/John Hopkins Data Science Specialization/(Course 5) Reproducible Research/RepData_PeerAssessment1-master")

rawdata <- read.csv("activity.csv", header = TRUE)
data <- na.omit(rawdata)
data[2] <- lapply(data[2], as.Date)
```

## What is mean total number of steps taken per day?
Make a histogram of the total number of steps taken each day


```r
data %>% group_by(date) %>% summarise(stepsPerDay = sum(steps)) %>% 
        ggplot(aes(x=stepsPerDay)) + geom_histogram(bins=10) + xlab("Steps per day") + ylab("frequency") + ggtitle("Mean total number of steps taken per day") + theme(plot.title = element_text(size=15, face="bold"))
```

![](PA1_template_files/figure-html/plot1-1.png)<!-- -->

Calculate the mean and median of the total number of steps taken per day


```r
data2 <- data %>% group_by(date) %>% summarise(stepsPerDay = sum(steps))
mean(data2$stepsPerDay)
```

```
## [1] 10766.19
```

```r
median(data2$stepsPerDay)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

```r
data %>% group_by(interval) %>% summarise(meanSteps = mean(steps, na.rm = TRUE)) %>%
        ggplot(aes(x = interval, y = meanSteps)) + geom_line() + xlab("Interval (0-2355)") + ylab("average number of steps taken") + ggtitle("Average daily activity pattern")
```

![](PA1_template_files/figure-html/plot2-1.png)<!-- -->

```r
data3 <- data %>% group_by(interval) %>% summarise(meanSteps = mean(steps, na.rm = TRUE))
data3[which.max(data3$meanSteps),]
```

```
## # A tibble: 1 x 2
##   interval meanSteps
##      <int>     <dbl>
## 1      835      206.
```

The 835th interval has the highest number of steps on average. This can be seen grapihcally as well.

## Imputing missing values
1. Calculate and report the total number of missing values in the dataset


```r
sum(is.na(rawdata))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
rawdata <- read.csv("activity.csv", header = TRUE)
data <- subset(rawdata, !is.na(rawdata$steps))
rawdata_input <- rawdata
NAValues <- is.na(rawdata_input$steps)
averageInt <- tapply(data$steps, data$interval, mean, na.rm=TRUE, simplify = TRUE)
rawdata_input$steps[NAValues] <- averageInt[as.character(rawdata_input$interval[NAValues])]
```

4.1 Make a histogram of the total number of steps taken each day


```r
rawdata_input %>% group_by(date) %>% summarise(stepsPerDay=sum(steps)) %>% ggplot(aes(x=stepsPerDay)) + geom_histogram(bins=15)
```

![](PA1_template_files/figure-html/plot3-1.png)<!-- -->

4.2 Calculate the mean and median


```r
rawdata_input2 <-  rawdata_input %>% group_by(date) %>% summarise(stepsPerDay=sum(steps)) 
mean(rawdata_input2$stepsPerDay)
```

```
## [1] 10766.19
```

```r
median(rawdata_input2$stepsPerDay)
```

```
## [1] 10766.19
```

These values do not differ from the estimates from the first part of the assignment in which the NA's were not takes into account. 

## Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
rawdata_input[2] <- lapply(rawdata_input[2], as.Date)
WeekDay<- function(d) {
        if (weekdays(d)=="Sunday" | weekdays(d)=="Saturday")
                day <- "Weekend" 
        else 
                day <- "Weekday"
                day
        } 

rawdata_input$DayOfWeek <- sapply(rawdata_input$date, WeekDay)
```

2. Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
rawdata_input %>% group_by(interval, DayOfWeek) %>% summarise(MeanStepsPerDay = mean(steps)) %>%
       ggplot(aes(x=interval, y=MeanStepsPerDay, color=DayOfWeek)) + geom_line() + facet_wrap(~DayOfWeek, ncol = 1, nrow=2)
```

![](PA1_template_files/figure-html/plot4-1.png)<!-- -->
