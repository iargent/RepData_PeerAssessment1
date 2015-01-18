# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Load the data (you will have to change the appropriate working directory for your environment):


```r
setwd("I:/!Mooc doing/Reproducible Research/Assignment1")
dat<-read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

Aggregate the total number of steps by date and plot a histogram:

```r
steps_by_date<-aggregate(steps ~ date, dat, sum)
hist(steps_by_date$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

Work out the mean number of steps per day:

```r
mean(steps_by_date$steps)
```

```
## [1] 10766.19
```

And the median number of steps per day:

```r
median(steps_by_date$steps)
```

```
## [1] 10765
```


## What is the average daily activity pattern?

Aggregate the total number of steps by interval (across all days) and plot a time series:

```r
mean_steps_by_interval<-aggregate(steps ~ interval, dat, mean)
plot(mean_steps_by_interval$interval, mean_steps_by_interval$steps, type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max_row<-which.max(mean_steps_by_interval$steps)
mean_steps_by_interval[max_row,]$interval
```

```
## [1] 835
```


## Imputing missing values

Fill in missing values by replacing them with the mean for their time interval. New data with the replaced values is stored in "dat2":

```r
dat2<-dat
missing <- which(is.na(dat$steps), arr.ind=TRUE)
lookup<-match(dat[missing,]$interval, mean_steps_by_interval$interval)
dat2[missing,]$steps<-mean_steps_by_interval[lookup,]$steps
```


Histogram

```r
steps_by_date2<-aggregate(steps ~ date, dat2, sum)
hist(steps_by_date2$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png) 


Work out the mean number of steps per day:

```r
mean(steps_by_date2$steps)
```

```
## [1] 10766.19
```

And the median number of steps per day:

```r
median(steps_by_date2$steps)
```

```
## [1] 10766.19
```

These values are the same as the mean for the original data (without imputing the missing values). Replacing missing values with interval means has meant that the numbers of steps is not guaranteed to be a whole number any more, and thus the median does also not have to be an integer.


## Are there differences in activity patterns between weekdays and weekends?


Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
day.type<-factor(weekdays(as.Date(dat2$date)) %in% c('Saturday','Sunday'), labels=c('weekday', 'weekend') )
dat2["day.type"]<-day.type
```

Split the data into weekday and weekend data frames.

```r
wkend.dat<-subset(dat2, day.type=="weekend")
wkday.dat<-subset(dat2, day.type=="weekday")
```

Aggregate each data frame by by interval:

```r
wkend_steps_by_interval<-aggregate(steps ~ interval, wkend.dat, mean)
wkday_steps_by_interval<-aggregate(steps ~ interval, wkday.dat, mean)
```

Join the two aggregated data frames together, adding back the factor column showing which data is for weekdays and which for weekends:

```r
wkend_steps_by_interval["day.type"]<-factor('weekend')
wkday_steps_by_interval["day.type"]<-factor('weekday')
mean_steps_by_interval2<-rbind(wkend_steps_by_interval, wkday_steps_by_interval)
```

Plot a lattice graph to compare the weekend and weekday time series:

```r
library(lattice)
xyplot( steps ~ interval | day.type, mean_steps_by_interval2, layout=c(1,2), type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png) 
