---
title: "course5wk2"
output: html_document
date: "2023-07-07"
---

##load packages

```r
library(ggplot2)
library(dplyr)
library(lubridate)
library(knitr)
```

##downloading and reading the file

```r
fileurl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
zipfile <- "./course5week2/data.zip"

filedir <- "./course5week2"
unzip_path <- "./course5week2/data"  ##### path for storing the unzipped files #######
if (!file.exists(filedir)){
  dir.create(filedir)
}
download.file(fileurl,file.path(zipfile))
unzip(zipfile,exdir=unzip_path) ####### exdir is the extract directory ##########
datafile <- file.path(unzip_path,"activity.csv")

activity <- read.csv(datafile)

activity$date <- ymd(activity$date)
activity$weekend <- as.factor(ifelse(weekdays(activity$date)=="Saturday" | weekdays(activity$date)=="Sunday","weekend","weekday"))
activity$dayofweek <- as.factor(weekdays(activity$date))
```

##Histogram of the total number of steps taken each day

```r
stepsByDay <- activity %>% group_by(date) %>% summarise(stepsperday = sum(steps,na.rm = TRUE))

qplot(stepsperday,data=stepsByDay,na.rm=TRUE,binwidth=500,xlab='Total steps per day', ylab='Frequency using binwith 500',main = 'Histogram of the total number of steps taken each day')
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

##Mean and median number of steps taken each day

```r
meanstepsperday <- stepsByDay %>% summarise(average = mean(stepsperday,na.rm = TRUE),median=median(stepsperday,na.rm = TRUE))

meanstepsperday
```

##time series plot of the 5-minute interval and the average number of steps across all days

```r
interval_average <- activity %>% group_by(interval) %>% summarise(average = mean(steps,na.rm = TRUE))

qplot(interval,average,data=interval_average,geom="line",xlab = "5-minute intervals",ylab = "Average steps taken across all days")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

##Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
interval_average[which.max(interval_average$average),]
```

##Imputing missing values

```r
activity_no_NA <- activity[which(!is.na(activity$steps)),]
  
interval_only <- activity_no_NA %>% group_by(interval) %>% summarise(average=mean(steps)) #calculate mean steps for each interval

interval_only$average <- as.integer(interval_only$average) #convert average to integer
    
activity_na <- activity[which(is.na(activity$steps)),] #subset dataset where steps have NAs
    
activity_na$steps <- ifelse(activity_na$interval==interval_only$interval,interval_only$average) #fill NAs with average steps based on interval
    
activity_impute <- rbind(activity_no_NA,activity_na) #row bind datasets that do not have NAs and  dataset where NAs are replaced with mean values
```

##number of missing values in the dataset

```r
nrow(activity_na)
```

```
## [1] 2304
```

##Histogram of the total number of steps taken each day after missing values are imputed

```r
stepsByDay_impute <- activity_impute %>% group_by(date) %>% summarise(stepsperday = sum(steps))

qplot(stepsperday,data=stepsByDay_impute,na.rm=TRUE,binwidth=500,xlab='Total steps per day', ylab='Frequency using binwith 500',main = 'Histogram of the total number of steps taken each day')
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)

##Mean and median number of steps taken each day

```r
totalstepsperday_impute <- activity_impute %>% group_by(date) %>% summarise(stepsperday = sum(steps))

mean_n_median <- totalstepsperday_impute %>% summarise(average=mean(stepsperday),median=median(stepsperday))

mean_n_median
```

##Are there differences in activity patterns between weekdays and weekends?

```r
meansteps <- activity_impute %>% group_by(interval,weekend) %>%   summarise(average = mean(steps))
```

```
## `summarise()` has grouped output by 'interval'. You can override using the
## `.groups` argument.
```

```r
qplot(interval,average,data=meansteps,geom="line",facets=weekend~.,xlab="5-minute interval",ylab="average number of steps",main="Average steps pattern between Weekday and Weekend")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png)
