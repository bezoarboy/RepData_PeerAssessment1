# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data


```r
setwd("~/datasciencecoursera/05_Reproducible_Research/RepData_PeerAssessment1")
data <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

```r
#http://stackoverflow.com/questions/16296657/row-aggregation-according-to-date-in-r-and-sum-corresponding-other-column-values
# x <- data[ !is.na(data$steps), ] ## not needed for this step -- NA's dont affect
x <- aggregate( data$steps, by = list(data$date), FUN = "sum")
names(x) = c( "date", "steps")
hist(x$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

```r
summarize <- summary(x$steps)
summarize["Mean"]
```

```
##  Mean 
## 10770
```

```r
summarize["Median"]
```

```
## Median 
##  10760
```

```r
library(data.table) # or can do it with a data table
dt <- data.table(data, key = "date")
bydate <- dt[,lapply(.SD, sum), by = "date"]
hist(bydate$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-2.png) 

```r
summary(bydate$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##      41    8841   10760   10770   13290   21190       8
```

## What is the average daily activity pattern?


```r
dataNoNA <- data[ !is.na(data$steps), ]
x <- aggregate( dataNoNA$steps, by = list(dataNoNA$interval), FUN = "mean")
names(x) <- c("interval","steps")
plot(x$interval, x$steps, type = "l")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 


## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
