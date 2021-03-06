# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data

### 1. Load the data


```r
data <- read.csv("activity.csv", header = TRUE, colClasses = c("integer", "Date", 
    "integer"))
```


## What is mean total number of steps taken per day?

### 1. Make a histogram of the total number of steps taken each day


```r
data2 <- data[complete.cases(data), ]
totalSteps <- tapply(data2$steps, data2$date, sum)
hist(totalSteps, breaks = 10, main = "Histogram of the total number of steps taken each day", 
    xlab = "The total number of steps taken each day", col = "blue")
```

![plot of chunk histogram1](figure/histogram1.png) 


### 2. Calculate and report the mean and median total number of steps taken per day


```r
mean <- mean(totalSteps)
median <- mean(totalSteps)
```


The mean of the total number of steps taken per day is 1.0766 &times; 10<sup>4</sup>.

The median of the total number of steps taken per day is 1.0766 &times; 10<sup>4</sup>.

## What is the average daily activity pattern?

### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).


```r
averageSteps <- tapply(data2$steps, data2$interval, mean)
plot(dimnames(averageSteps)[[1]], averageSteps, type = "l", main = "Time series plot", 
    xlab = "Interval", ylab = "The average number of steps taken")
```

![plot of chunk time_series_plot](figure/time_series_plot.png) 


### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?



```r
numericValues <- as.numeric(averageSteps)
max <- dimnames(averageSteps)[[1]][numericValues == max(numericValues)]
```


The 5-minute interval that contains the maximum number of steps is 835.

## Imputing missing values

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum <- sum(!complete.cases(data))
```


The total number of rows with NAs is 2304.

### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
fillingIn <- function(data) {
    data2 <- data[complete.cases(data), ]
    average <- tapply(data2$steps, data2$interval, mean, simplify = FALSE)
    for (i in 1:nrow(data)) {
        if (is.na(data[i, 1])) {
            data[i, 1] = average[as.character(data[i, 3])]
        }
    }
    return(data)
}
```


The above function fills in all of the missing values with the mean for that 5-minute interval averaged across all the days.

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
data3 <- fillingIn(data)
```


### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
totalSteps <- tapply(data3$steps, data3$date, sum)
hist(totalSteps, breaks = 10, main = "Histogram of the total number of steps taken each day", 
    xlab = "The total number of steps taken each day", col = "blue")
```

![plot of chunk histogram2](figure/histogram2.png) 

```r
mean2 <- mean(totalSteps)
median2 <- mean(totalSteps)
```


The mean of the total number of steps taken per day is 1.0766 &times; 10<sup>4</sup>.

The median of the total number of steps taken per day is 1.0766 &times; 10<sup>4</sup>.

These values are the same as the estimates from the first part of the assignment.

Impuing missing data has no impact on the estimates of the total daily number of steps.

## Are there differences in activity patterns between weekdays and weekends?

### 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.



```r
values <- as.character()
for (i in 1:nrow(data3)) {
    value <- as.character()
    if (weekdays(data3[i, 2]) == "Saturday" | weekdays(data3[i, 2]) == "Sunday") {
        value = "weekend"
    } else {
        value = "weekday"
    }
    values <- append(values, value)
}

day <- factor(values, levels = c("weekday", "weekend"))
data3$day <- day
```


### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
library(lattice)
```

```
## Warning: package 'lattice' was built under R version 3.0.3
```

```r
dataWeekday <- data3[data3$day == "weekday", ]
dataWeekend <- data3[data3$day == "weekend", ]

averageStepsWeekday <- tapply(dataWeekday$steps, dataWeekday$interval, mean)
averageStepsWeekend <- tapply(dataWeekend$steps, dataWeekend$interval, mean)

data4a <- data.frame(interval = as.numeric(dimnames(averageStepsWeekend)[[1]]), 
    avgStep = averageStepsWeekend, day = rep("weekend", times = length(averageStepsWeekend)))
data4b <- data.frame(interval = as.numeric(dimnames(averageStepsWeekday)[[1]]), 
    avgStep = averageStepsWeekday, day = rep("weekday", times = length(averageStepsWeekday)))

data4 <- rbind(data4a, data4b)

xyplot(avgStep ~ interval | day, data = data4, type = "l", layout = c(1, 2), 
    xlab = "Interval", ylab = "Number of steps")
```

![plot of chunk panel_plot](figure/panel_plot.png) 

