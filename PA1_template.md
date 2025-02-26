Reproducible Research
================
Viswa

## Introduction

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a Fitbit, Nike
Fuelband, or Jawbone Up. These type of devices are part of the
“quantified self” movement – a group of enthusiasts who take
measurements about themselves regularly to improve their health, to find
patterns in their behavior, or because they are tech geeks. But these
data remain under-utilized both because the raw data are hard to obtain
and there is a lack of statistical methods and software for processing
and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and
include the number of steps taken in 5 minute intervals each day.

he data for this assignment can be downloaded from the course web site:

  - Dataset: [Activity monitoring
    data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

## Loading and preprocessing the data

Unzip data to obtain a csv file.

``` r
library("data.table")
```

    ## Warning: package 'data.table' was built under R version 3.5.3

``` r
library(ggplot2)
```

    ## Warning: package 'ggplot2' was built under R version 3.5.3

``` r
library(scales)
```

    ## Warning: package 'scales' was built under R version 3.5.3

``` r
library(Hmisc)
```

    ## Warning: package 'Hmisc' was built under R version 3.5.3

    ## Loading required package: lattice

    ## Loading required package: survival

    ## Loading required package: Formula

    ## 
    ## Attaching package: 'Hmisc'

    ## The following objects are masked from 'package:base':
    ## 
    ##     format.pval, units

``` r
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl, destfile = paste0(getwd(), '/repdata_data_activity.zip'), method = "curl")
unzip("repdata_data_activity.zip",exdir = "data")
```

## Reading csv Data into Data.Table.

``` r
activityData <- data.table::fread(input = "data/activity.csv")
```

## Histogram of the total number of steps taken each day

``` r
stepsPerDay <- tapply(activityData$steps, activityData$date, sum, na.rm=TRUE)
qplot(stepsPerDay, xlab='Total steps per day', ylab='Frequency using binwith 1000', binwidth=1000)
```

![](Reproducible_research_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

## Mean and median number of steps taken each day

``` r
stepsPerDay_mean <- mean(stepsPerDay)
stepsPerDay_median <- median(stepsPerDay)
```

  - Mean: 9354.2295082
  - Median:
10395

## Time series plot of the average number of steps taken

``` r
averageStepsPerTimeBlock <- aggregate(x=list(meanSteps=activityData$steps), by=list(interval=activityData$interval), FUN=mean, na.rm=TRUE)

ggplot(data=averageStepsPerTimeBlock, aes(x=interval, y=meanSteps)) +
    geom_line() +
    xlab("5-minute interval") +
    ylab("average number of steps taken") 
```

![](Reproducible_research_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## The 5-minute interval that, on average, contains the maximum number of steps

``` r
maxSteps <- which.max(averageStepsPerTimeBlock$meanSteps)
TODMaxSteps <-  gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2", averageStepsPerTimeBlock[maxSteps,'interval'])
```

  - Most Steps at: 8:35

## Code to describe and show a strategy for imputing missing data

``` r
missingData <- length(which(is.na(activityData$steps)))
```

  - Number of missing values:
2304

## Histogram of the total number of steps taken each day after missing values are imputed

``` r
activityDataImputed <- activityData
activityDataImputed$steps <- impute(activityData$steps, fun=mean)

stepsByDayImputed <- tapply(activityDataImputed$steps, activityDataImputed$date, sum)
qplot(stepsByDayImputed, xlab='Total steps per day (Imputed)', ylab='Frequency using binwith 500', binwidth=500)
```

![](Reproducible_research_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

## Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

``` r
activityDataImputed$dateType <-  ifelse(as.POSIXlt(activityDataImputed$date)$wday %in% c(0,6), 'weekend', 'weekday')

averagedActivityDataImputed <- aggregate(steps ~ interval + dateType, data=activityDataImputed, mean)
ggplot(averagedActivityDataImputed, aes(interval, steps)) + 
    geom_line() + 
    facet_grid(dateType ~ .) +
    xlab("5-minute interval") + 
    ylab("avarage number of steps")
```

![](Reproducible_research_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

## All of the R code needed to reproduce the results (numbers, plots, etc.) in the report
