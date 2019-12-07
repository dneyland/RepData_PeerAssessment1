---
title: "Reproducible Research W2 Project 1"
output: html_document
---



##Preprocessing Data
First the data is read into the R environment, dates are converted to a more useful
format and the *dplyr* library is loaded.

```r
data <- read.csv("activity.csv")
data$date <- as.Date(data$date,"%d/%m/%Y")
library(dplyr)
```

##Daily Step Count
The first lot of processing requires aggregation to a daily step count level.  
The data is summed to the daily level and a histogram can be plotted.

```r
dailysteps <- data %>% group_by(date) %>% summarise_all(sum)
hist(dailysteps$steps, main="Histogram of the number of steps taken each day", 
                    xlab="Steps taken for the day")
```

![plot of chunk next chunk](figure/next chunk-1.png)

It is relatively simple from here to calculate the daily mean:

```r
mean(dailysteps$steps,na.rm=TRUE)
```

```
## [1] 10766.19
```

... and median:

```r
median(dailysteps$steps,na.rm=TRUE)
```

```
## [1] 10765
```

##Time Interval Analysis
The second lot of processing requires aggregation to the time interval level.  
From this point, plot the time series across all intervals.

```r
timeintervalsteps <- data %>% 
                        group_by(interval) %>%
                        summarise(meansteps=mean(steps,na.rm=TRUE),totalsteps=sum(steps,na.rm=TRUE))
plot(timeintervalsteps$interval, timeintervalsteps$meansteps, type="l",
                main="Time Series of Mean Steps per Time Interval", xlab="Time Interval",
                ylab="Mean Steps Taken")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

The interval in which the largest total of steps is taken can be found by indexing to the maximum
of the total steps across all intervals:

```r
with(timeintervalsteps,interval[totalsteps==max(totalsteps)])
```

```
## [1] 835
```

##Missing Values
The number of missing values can be calculated:

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

To remove bias introduced by the missing values the median for that time interval shall be used to replace missing values.
Create a new variable *replacementsteps* that is the *steps* variable but with the missing values replaced with the median for the time interval.

```r
datanew <- data %>% group_by(interval) %>% mutate(replacementsteps = ifelse(is.na(steps),median(steps,na.rm=TRUE),steps))
```

The data is summed to the daily level and a histogram can be plotted.

```r
dailysteps <- datanew %>% group_by(date) %>% summarise_all(sum)
hist(dailysteps$replacementsteps, main="Histogram of the number of steps taken each day", 
                    xlab="Steps taken for the day")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png)

It is relatively simple from here to calculate the daily mean:

```r
mean(dailysteps$replacementsteps,na.rm=TRUE)
```

```
## [1] 9503.869
```

... and median:

```r
median(dailysteps$replacementsteps,na.rm=TRUE)
```

```
## [1] 10395
```
It can be seen that imputing missing data in this way has increased the frequency of days with a lower number of steps per day, lowering both the mean and median number of steps per day.  

##Day of week analysis
The next study was done by adding a new column to the dataframe with the imputed missing values, *datanew*, which communicates whether the date is a weekday or weekend.

```r
datanew <- datanew %>% mutate(daytype = ifelse(weekdays(date)=="Saturday"|weekdays(date)=="Sunday","weekend","weekday"))
```
Then averaging over each time interval for that variable and plotting:

```r
timeintervalsteps <- datanew %>% 
                      group_by(interval) %>%
                      summarise(meanstepsweekday=mean(replacementsteps[daytype=="weekday"]),
                                meanstepsweekend=mean(replacementsteps[daytype=="weekend"]))
par(mfrow=c(1,2))
plot(timeintervalsteps$interval, timeintervalsteps$meanstepsweekday, type="l",
                 main="Mean Steps Over Time Weekdays", xlab="Time Interval",
                 ylab="Mean Steps Taken")
plot(timeintervalsteps$interval, timeintervalsteps$meanstepsweekend, type="l",
                 main="Mean Steps Over Time Weekends", xlab="Time Interval",
                 ylab="Mean Steps Taken")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png)
