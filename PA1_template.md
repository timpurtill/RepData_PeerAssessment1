---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the dataraw

```r
rawStepsData <- read.csv("activity.csv", header = TRUE)
```

## What is mean total number of steps taken per day?
1. Calculate the total number of steps taken per day

```r
stepsPerDay <- tapply(rawStepsData$steps, rawStepsData$date, sum, na.rm = TRUE)
```


2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
hist(stepsPerDay, breaks = 10, main = "Histrogram of Steps Per Day", xlab="Steps Per Day", ylab="Occurances")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)

3. Calculate and report the mean and median of the total number of steps taken per day

```r
mean(stepsPerDay)
```

```
## [1] 9354.23
```

```r
median(stepsPerDay)
```

```
## [1] 10395
```

## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
averageStepsPerInterval <- aggregate( rawStepsData$steps ~ rawStepsData$interval, FUN = mean, na.rm = TRUE )
plot(averageStepsPerInterval$`rawStepsData$interval`,averageStepsPerInterval$`rawStepsData$steps`, xlab="5 Minute Intervals", ylab="Average Steps", type = "l")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
averageStepsPerInterval$`rawStepsData$interval`[which.max(averageStepsPerInterval$`rawStepsData$steps`)]
```

```
## [1] 835
```

## Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(rawStepsData$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
Create a new dataset that is equal to the original dataset but with the missing data filled in.

To achieve this I created a short lookup function that looped over all the NA values and replaced them with the mean steps per interval for that same time interval from the previous step


```r
naReplacedData <- rawStepsData
naVector <- is.na(naReplacedData$steps)
total = 0
for (i in seq_along(naVector)){
       
        if (naVector[i] == TRUE){

                naReplacedData$steps[i] <- averageStepsPerInterval[averageStepsPerInterval$`rawStepsData$interval` == naReplacedData$interval[i],]$`rawStepsData$steps`

        }
}
```

3. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
nnaStepsPerDay <- tapply(naReplacedData$steps, naReplacedData$date, sum, na.rm = TRUE)
hist(nnaStepsPerDay, breaks = 10, main = "Histrogram of Steps Per Day (replaced NAs)", xlab="Steps Per Day", ylab="Occurances")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png)

```r
print(paste("Mean: ", mean(nnaStepsPerDay)))
```

```
## [1] "Mean:  10766.1886792453"
```

```r
print(paste("Median: ", median(nnaStepsPerDay)))
```

```
## [1] "Median:  10766.1886792453"
```

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
library(chron)
```

```
## Warning: package 'chron' was built under R version 3.3.1
```

```r
weekdayNaReplacedData <- naReplacedData

weekdayNaReplacedData$dayType <- is.weekend(weekdayNaReplacedData$date)
weekdayNaReplacedData$dayType <- factor(weekdayNaReplacedData$dayType, levels = c(FALSE, TRUE), labels = c("weekday", "weekend"))
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
library(lattice)
averageStepsPerIntervalPerDayType <- aggregate( weekdayNaReplacedData$steps ~ weekdayNaReplacedData$interval+weekdayNaReplacedData$dayType, FUN = mean, na.rm = TRUE )

xyplot(averageStepsPerIntervalPerDayType$`weekdayNaReplacedData$steps` ~ averageStepsPerIntervalPerDayType$`weekdayNaReplacedData$interval` | averageStepsPerIntervalPerDayType$`weekdayNaReplacedData$dayType`, data = averageStepsPerIntervalPerDayType, layout = c(1, 2), type = "l", xlab = "Interval", ylab = "Number of Steps")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png)
