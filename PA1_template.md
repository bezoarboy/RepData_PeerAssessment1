---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

* load the data
* create a separate copy of the data, with NA steps rows removed


```r
setwd("~/datasciencecoursera/05_Reproducible_Research/RepData_PeerAssessment1")
data <- read.csv("activity.csv")
data.rmNA <- data[ !is.na(data$steps), ]
```

## What is mean total number of steps taken per day?

Make a histogram of the total number of steps taken each day

* used the aggregate function to sum the number of steps over each date

Report the mean and median total number of steps taken per day


```r
#http://stackoverflow.com/questions/16296657/row-aggregation-according-to-date-in-r-and-sum-corresponding-other-column-values
bydate <- aggregate( data.rmNA$steps, by = list(data.rmNA$date), FUN = "sum")
names(bydate) = c( "date", "steps")
hist(bydate$steps, main = 'Total number of steps per day', xlab = '# of steps')
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

```r
mean(bydate$steps)
```

```
## [1] 10766.19
```

```r
median(bydate$steps)
```

```
## [1] 10765
```


## What is the average daily activity pattern?

Make a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days

* used the aggregate function to calculate the mean number of steps over each interval

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

* determine the interval row with the highest # of steps, and display the row


```r
byinterval <- aggregate( data.rmNA$steps, by = list(data.rmNA$interval), FUN = "mean")
names(byinterval) <- c("interval","steps")
plot(byinterval$interval, byinterval$steps, type = "l", main = "Total number of steps by time interval", xlab = "5-minute time interval", ylab = "# of steps")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

```r
maxintervalindex <- which.max(byinterval$steps)
byinterval[maxintervalindex,]
```

```
##     interval    steps
## 104      835 206.1698
```


## Imputing missing values

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

* prior inspection of the data showed NA's only in the steps column
* also can confirm with complete.cases()

Strategy for filling in all of the missing steps values in the dataset

* for any time interval with NA number of steps, filled in the mean for that 5-minute time interval

Create a new dataset that is equal to the original dataset but with the missing data filled in.

* done by making a copy of the dataset and iterating over every row looking for and replacing NA step values
* relies on the "byinterval" dataframe created previously
* there's almost certainly a better, vectorized way to do this, but I couldn't figure it out

Histogram of the total number of steps taken each day and mean / median reported as before

* based on the method of imputing values, would expect minimal effect on the median / mean on the imputed dataset


```r
sum( is.na(data$steps) )
```

```
## [1] 2304
```

```r
sum(!complete.cases(data))
```

```
## [1] 2304
```

```r
imputed <- data
for (i in 1:nrow(imputed)) { # iterate over every row of the copied dataset
    if ( is.na(imputed[i,]$steps) ) { # if the number of steps for that row is NA...
        interval <- imputed[i,]$interval # figure out what the interval is for that row...
        imputed[i,]$steps <- byinterval[ byinterval$interval == interval, ]$steps # fill in mean # of steps for that interval
    }
}
bydate <- aggregate( imputed$steps, by = list(imputed$date), FUN = "sum")
names(bydate) = c( "date", "steps")
hist(bydate$steps, main = 'Total number of steps per day, Imputed dataset', xlab = '# of steps')
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

```r
mean(bydate$steps)
```

```
## [1] 10766.19
```

```r
median(bydate$steps)
```

```
## [1] 10766.19
```


## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

Plot a comparison of # of steps in each 5-minute interval of the average number of steps taken


```r
library(car) # from http://stackoverflow.com/questions/14841468/how-to-create-a-new-factor-based-on-a-current-factor
imputed$date <- as.Date(imputed$date)
imputed$dayofweek = as.factor( weekdays(imputed$date) )
imputed$weekfactor <- recode( imputed$dayofweek, "c('Friday','Monday','Thursday','Tuesday','Wednesday') = 'weekday';
                              c('Saturday','Sunday') = 'weekend' ")

byinterval <- aggregate( imputed$steps, by = list(imputed$interval, imputed$weekfactor), FUN = "mean")
names(byinterval) <- c("interval","weekfactor","steps")

library(ggplot2)
g <- ggplot( byinterval, aes(interval, steps, group = weekfactor) )
g + geom_line() + facet_wrap( ~ weekfactor, ncol = 1 ) + labs(title = "Comparison of weekday vs. weekend steps", x = "5-minute time interval", y = "# of steps")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

## Submission

Requires:

* library(knitr)
* knit2html("PA1_template.Rmd")
