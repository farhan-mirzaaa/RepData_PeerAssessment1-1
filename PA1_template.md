---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data


```r
library(lubridate)
library(dplyr)
library(reshape2)
library(stringr)
library(lattice)
library(knitr)
```


```r
#unzipping and opening the file
unzip("activity.zip")

walkingData <- read.csv("activity.csv",na.strings = "NA")
walkingData$date <- as.Date(walkingData$date)
```

## What is mean total number of steps taken per day?


```r
dailyActivity <- aggregate(steps ~ date,walkingData,sum)
walkingMean <- round(mean(dailyActivity$steps,na.rm=TRUE),0)
walkingMedian <- median(dailyActivity$steps,na.rm = TRUE)

hist(dailyActivity$steps,breaks = 12,col = "green",xlab="daily steps",
     main = "Daily frequency of walking")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

The mean number of steps per day is 10766 and the median is 10765


## What is the average daily activity pattern?


```r
#formatting times
walkingData$interval <- str_pad(walkingData$interval, 4, pad = "0") 
walkingData$interval <- paste0(walkingData$interval,":00")
walkingData$interval <- paste(substring(walkingData$interval,1,2),substring(walkingData$interval,3,7),sep=":") 

walkingData$datetime <- as.POSIXct(paste(walkingData$date,walkingData$interval),format="%Y-%m-%d %H:%M:%S")

#plot
plot(walkingData$datetime,walkingData$steps,type = "l",xlab="date",ylab="number of steps per interval",
     sub= sprintf("mean steps per interval is %1.1f", mean(walkingData$steps,na.rm = TRUE)))
abline(a = mean(walkingData$steps,na.rm = TRUE),b=0,col="green")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
#finding where max val is
maxval <- which(walkingData$steps - max(walkingData$steps,na.rm = TRUE) == 0)
```

The time period with the most activity is 2012-11-27 06:15:00

## Imputing missing values


```r
#NA values

missingNum <- sum(is.na(walkingData$steps))
missingFrac <- sum(is.na(walkingData$steps))/length(walkingData$steps)
```

The number of missing values is 2304 out of 17568


```r
#replacing missing NA fraction with mean from that hourly time interval

hoursub <- aggregate(steps ~ hour(datetime),walkingData,mean)
colnames(hoursub) <- c("hour","steps")

#make sure subbing makes sense
barplot(names.arg =hoursub$hour,hoursub$steps,ylab="steps",xlab="hour")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
#make the substitution

walkingDataComp <- walkingData

for (i in 0:23) {
       
        walkingDataComp$steps[is.na(walkingDataComp$steps) & 
                                      hour(walkingDataComp$datetime) == i] <- hoursub[i+1,2] 
}

#plot the result
par(mfrow=c(1,2))
plot(walkingData$datetime,walkingData$steps,type = "l",main="before NA replacement",xlab="date",ylab="steps per 5min interval")
plot(walkingDataComp$datetime,walkingDataComp$steps,type = "l",main="after NA replacement",xlab="date",ylab="")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-2.png)<!-- -->

It looks like the hourly cycle of replacement makes sense.


```r
#comparing new substituted values
dailyActivityComp <- aggregate(steps ~ date,walkingDataComp,sum)

#historgram, plust mean and median of daily activity
walkingMeanComp <- mean(dailyActivityComp$steps,na.rm=TRUE)
walkingMedianComp <- round(median(dailyActivityComp$steps,na.rm = TRUE))

par(mfrow=c(1,2))

hist(dailyActivity$steps,breaks = 12,col = "green",xlab="daily steps",
     main = "Pre-NA substitution",sub=sprintf("mean = %2.1f steps, median = %2.0f steps",walkingMean,walkingMedian))

hist(dailyActivityComp$steps,breaks = 12,col = "blue",xlab="daily steps",
     main = "Post-NA substitution",sub=sprintf("mean = %2.1f steps, median = %2.0f steps",walkingMeanComp,walkingMedianComp))
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

Adding missing values does not appear to strongly affect the distribution. 

## Are there differences in activity patterns between weekdays and weekends?


```r
#comparing weekday vs weekends

walkingDataComp$weekday <- weekdays(walkingDataComp$datetime)

walkingDataComp$weekday[walkingDataComp$weekday == "Saturday" | walkingDataComp$weekday == "Sunday"] <- "weekend"
walkingDataComp$weekday[walkingDataComp$weekday != "weekend"] <- "weekday"

#creating panel plot

walkingMeanWknd <- round(mean(walkingDataComp$steps[walkingDataComp$weekday=="weekend"]))
walkingMeanWday <- round(mean(walkingDataComp$steps[walkingDataComp$weekday=="weekday"]))

with(walkingDataComp,xyplot(steps~datetime|weekday,
                            type="l",sub=sprintf("weekend = %d steps/day vs weekday = %d steps/day",
                                                  walkingMeanWknd*24*12,walkingMeanWday*24*12)))
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

It looks like there are indeed differences between the weekend and weekdays. 
