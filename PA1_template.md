---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---



## Loading and preprocessing the data
First we need to unzip and clean the data so we end up with a tidy dataset,
storing it in a variable called dataset. The dates need to be transformed from
a factor variable too.

This is the code to achieve this:

```r
suppressWarnings(library(lubridate))
unzip("activity.zip")
dataset <- read.csv("activity.csv")
dataset <- transform(dataset, date = ymd(date))
```

## What is mean total number of steps taken per day?
For this portion we're told we can ignore missing values in the dataset. As I
see it there are 2 ways of achieving this: use **complete.cases** and completely
remove missing values or use **na.rm = TRUE** when summarizing the data.

I chose to use the latter option although it distorts summaries (eg. setting
quite a few days with missing data to 0 when doing the sum of the number of
steps) since while reading the entire assignment I realised the bias and
distortions should indeed be there and dealt with on the *Imputing missing
values* part of the assignment.

So, in order to build the histogram I created an array using tapply with the
variable **steps** as the factor and the **sum** function to summarize the 
number of steps. I then proceeded to plot it with an histogram as required:


```r
plotdata1 <- tapply(dataset$steps, dataset$date, sum, na.rm = TRUE)
hist(plotdata1,
     main = "Histogram of the total number of steps taken each day",
     xlab = "Number of steps")
```

![plot of chunk plot1](figure/plot1-1.png) 

As we can see from the plot, this looks like a normal distribution with 10,000
to 15,000 steps being taken most days. As a result of the decision described
previously of using na.rm = TRUE the number of days with 0 steps looks inflated.

In order to calculate the median and the mean I used the following code:


```r
mean1 <- mean(plotdata1)
median1 <- median(plotdata1)
```

From the code above, it resulted that the mean of steps taken per day is
9354.2295082 and the median of steps taken per day is 10395.

## What is the average daily activity pattern?

To find the average daily activity pattern I have a similar approach to the
previous question, except the factor will now be the variable **interval** and
the summarizing function will be **mean** in order to calculate the average
number of steps per interval. 

The caveat with this plot is that the integer sequence for interval is not a
linear sequence as it represents the actual time of day (eg. it jumps from 55
to 100, meaning it goes from 00:55 to 01:00). For this reason the x axis is a
sequence of 288 integers (the number of 5 minute intervals in 24 hours), where
I customized the labels for it to make sense.

This is the code to achieve this:


```r
plotdata2 <- tapply(dataset$steps, dataset$interval, mean, na.rm = TRUE)
plot(1:288, plotdata2, type="l", xaxt="n",
     main="Average number of steps per 5 minute interval",
     ylab="Number of steps",
     xlab="Time of day")
axis(1, at=c(1,37,73,109,145,181,217,253,288),
     labels=c("00:00","03:00","06:00","09:00",
              "12:00","15:00","18:00","21:00","23:55"))
```

![plot of chunk plot2](figure/plot2-1.png) 

In order to get the 5 minute interval with the highest average number of steps
this is the code that needs to be run:


```r
max_interval <- names(which.max(plotdata2))
```

The interval with the highest number of interval steps is 835.

## Imputing missing values

The total number of missing values in the dataset is
2304.

The strategy I chose to impute the missing data was to use the mean of the 5
minute interval. This is the code I used to create the new dataset without 
missing values:


```r
suppressWarnings(library(plyr))
impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))
fullDataset <- ddply(dataset, ~interval, transform, steps=impute.mean(steps))
fullDataset <- fullDataset[order(fullDataset$date, fullDataset$interval),]
```

Now in order to get the histogram and mean and median total number of steps
taken per day we just need to repeat the code in the first part of the
assignment using the new data set.


```r
plotdata3 <- tapply(fullDataset$steps, fullDataset$date, sum, na.rm = TRUE)
hist(plotdata3,
     main = "Histogram of the total number of steps taken each day",
     xlab = "Number of steps")
```

![plot of chunk plot3](figure/plot3-1.png) 

```r
mean3 <- format(mean(plotdata3), scientific=FALSE)
median3 <- format(median(plotdata3), scientific=FALSE)
```

The mean for the total number of steps per day is now 10766.19 and the median
for the total number of steps per day is now 10766.19. Imputing missing
data makes the histogram look more like a normal distribution as it removes the
frequency of 0 steps per day which occurs for days where there is no data.

## Are there differences in activity patterns between weekdays and weekends?

The first step is to create the factor variable and add it to the data set
without missing values. This is the code used:


```r
fullDataset <- transform(fullDataset, dayType = weekdays(fullDataset$date))
fullDataset$dayType <- sub("Saturday|Sunday","weekend",weekdays)
fullDataset$dayType <- sub("Monday|Tuesday|Wednesday|Thursday|Friday",
                           "weekday",weekdays)
fullDataset$dayType <- factor(fullDataset$dayType)
```

Now in order to create the plot required we need to calculate the average number
of steps taken of the 5-minute interval and day type and proceed to create the
plot. I used the lattice system and this is the code to get that plot (again, I
had to change the interval from an integer into a sequence so it is linear and
then manually set the correct labels):


```r
library(lattice)
plotdata4 <- aggregate(steps ~ interval + dayType,
                       data = fullDataset,
                       FUN="mean")
correctInterval <- c(1:288,1:288)
plotdata4 <- transform(plotdata4,interval = correctInterval)
xyplot(steps ~ interval | dayType,
       data=plotdata4,
       type = "l",
       layout = c(1,2),
       main="Average number of steps taken per 5 minute interval and day type",
       xlab="Interval",
       ylab="Number of steps",
       scales=list(
           x=list(
               at=c(1,37,73,109,145,181,217,253,288),
               labels=c("00:00","03:00","06:00","09:00",
                        "12:00","15:00","18:00","21:00","23:55"))))
```

![plot of chunk plot4_2](figure/plot4_2-1.png) 
