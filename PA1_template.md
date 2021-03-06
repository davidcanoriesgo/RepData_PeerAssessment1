---
title: 'Reproducible Research: Peer Assessment 1'
author: "David Cano"
output: html_document
---

##Loading and preprocessing the data

Show any code that is needed to

1. Load the data (i.e. read.csv())
2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
activity <- read.csv("activity.csv", colClasses = c("numeric", "character", 
    "numeric"))
activity$date <- as.Date(activity$date, "%Y-%m-%d") 
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

##What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day
2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```r
activity.no.na <- na.omit(activity)
steps_perday <- aggregate(activity.no.na$steps,list(activity.no.na$date),sum)
##creating a histogram of the total number of steps per day
library(ggplot2)
steps_perday <- data.frame(steps_perday)
names(steps_perday)<-c("Day","Nr.Steps")
qplot(x=Day,y=Nr.Steps,scale_x_date(labels = date_format("%m/%d")),data=steps_perday,geom="bar",stat="identity")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day



```r
## Mean
StepsbydayMean <- mean(steps_perday$Nr.Steps, na.rm = TRUE)
print(StepsbydayMean)
```

```
## [1] 10766.19
```

```r
## Median
StepsbydayMedian <- median(steps_perday$Nr.Steps, na.rm = TRUE)
print(StepsbydayMedian)
```

```
## [1] 10765
```

##What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
## Calculate mean per interval
steps_by_interval <- aggregate(steps ~ interval, activity, mean)
## Plot
plot(steps_by_interval$interval,steps_by_interval$steps, type="l", xlab="Interval", ylab="Nr Steps",main="Avg Nr Steps per Day by Interval")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
Max <- steps_by_interval[steps_by_interval$steps==max(steps_by_interval$steps),]
print(Max)
```

```
##     interval    steps
## 104      835 206.1698
```

##Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
countNA <- nrow(subset(activity, is.na(activity$steps)))
print(countNA)
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
## we calculated the mean in the previous section, we fill in values with this mean, tapply used to calculate the mean per interval and assign it to na values


stepValues<-data.frame(activity$steps)
stepValues[is.na(stepValues),] <- ceiling(tapply(X=activity$steps,INDEX=activity$interval,FUN=mean,na.rm=TRUE))
newData <- cbind(stepValues, activity[,2:3])
colnames(newData) <- c("Steps", "Date", "Interval")
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?




```r
# Sum to calculate steps per day
Newsteps_perday <- aggregate(newData$Steps,list(newData$Date),sum)
##creating a histogram of the total number of steps per day
Newsteps_perday <- data.frame(Newsteps_perday)
# Plot
names(Newsteps_perday)<-c("Date","Steps")
qplot(x=Date,y=Steps,scale_x_date(labels = date_format("%m/%d")),data=Newsteps_perday,geom="bar",stat="identity")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

```r
## Mean
NewStepsbydayMean <- mean(Newsteps_perday$Steps, na.rm = TRUE)
print(NewStepsbydayMean)
```

```
## [1] 10784.92
```

```r
## Median
NewStepsbydayMedian <- median(Newsteps_perday$Steps, na.rm = TRUE)
print(NewStepsbydayMedian)
```

```
## [1] 10909
```

As per calculation:
- Mean has changed from 1.0766189 &times; 10<sup>4</sup> to 1.0784918 &times; 10<sup>4</sup> with the refilling
- Median has changed from 1.0765 &times; 10<sup>4</sup> to 1.0909 &times; 10<sup>4</sup> with the refilling

##Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
dateDayType <- data.frame(sapply(X = newData$Date, FUN = function(day) {
    if (weekdays(as.Date(day)) %in% c("Monday", "Tuesday", "Wednesday", "Thursday", 
        "Friday","lunes","martes","mi�rcoles","jueves","viernes")) {
        day <- "weekday"
    } else {
        day <- "weekend"
    }
}))


newDataWithDayType <- cbind(newData, dateDayType)

names(newDataWithDayType) <- c("Steps", "Date", "Interval", "DayType")
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
## Calculate Data with mean per day type and interval

dayTypeIntervalSteps <- aggregate(
    data=newDataWithDayType,
    Steps ~ DayType + Interval,
    FUN=mean
)

## plotting as 2 rows schema
library("lattice")


xyplot(
    type="l",
    data=dayTypeIntervalSteps,
    Steps ~ Interval | DayType,
    xlab="Interval",
    ylab="Number of steps",
    layout=c(1,2)
)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 
