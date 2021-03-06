---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
    fig_caption: yes
---


## Loading and preprocessing the data

The below code is for loading and preprocessing the data

```r
if (!file.exists("./repdata-data-activity.zip")) {
  download.file(url="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", destfile="./repdata-data-activity.zip", method = "libcurl")
}
unzip("./repdata-data-activity.zip", overwrite = TRUE, exdir = ".")

DataSet<-read.csv("./activity.csv",header = TRUE, na.strings = "NA")
DataSet$date <- as.Date(DataSet$date,"%Y-%m-%d")
```

## What is mean total number of steps taken per day?

Step 1. Calculate total steps per date


```r
library(dplyr)
K<-aggregate(steps~date, DataSet, sum,na.rm=TRUE)
```

Step 2. Plot the Histogram


```r
hist(K$steps,main = "Histogram for total Steps daily", col="red",xlab="No. of Steps",ylab="Freq. of Steps")
```

<img src="figure/histogram-1.png" title="plot of chunk histogram" alt="plot of chunk histogram" style="display: block; margin: auto;" />

Step 3. Calculate the mean and median of the total number of steps taken per day

Code for calculating the mean:


```r
MeanResult<-mean(K$steps)
```

The mean is 10766

Code for calculating the median:


```r
MedianResult<-median(K$steps)
```

The median is 10765

## What is the average daily activity pattern?

Code for finding the daily pattern:


```r
DAP<-aggregate(steps~interval, DataSet,mean,na.rm=TRUE)
```

Code for plotting the time series plot for the daily activity


```r
plot(DAP$interval,DAP$steps, type="l", xlab="Interval", ylab="Avg. No. of Steps by Interval", main="Avg.No.of steps taken across all days",col='red')
```

<img src="figure/timeseries-1.png" title="plot of chunk timeseries" alt="plot of chunk timeseries" style="display: block; margin: auto;" />

The code for finding the interval with max. steps:


```r
MaxSteps<-DAP[DAP$steps==max(DAP$steps),1]
```

The result is 835

## Imputing missing values

Step1.Code for calculate the missing value:


```r
sum(is.na(DataSet))
```

The no. of missing value is 2304

In order to fill in the missing value, the avg. no. of steps based on each interval will be used to replace the missing value

Below are the code used to merge the original dataset and the dataset of avg. no. of steps by each interval and replace the missing value:


```r
CleanData<- merge(DataSet,DAP,by="interval")
CleanData[is.na(CleanData$steps.x),2]<-CleanData[is.na(CleanData$steps.x),4]
CleanData<- subset(CleanData, select =names(CleanData[1:3]))
names(CleanData)<-c("interval","steps","date")
```

After that, the no. of steps of each day will be re-calculate and plot out the histogram with data cleaned:


```r
CleanDataResult<-aggregate(steps~date, CleanData, sum)
hist(CleanDataResult$steps,main = "Histogram for daily Steps with data cleaned", col="red",xlab="No. of Steps",ylab="Freq. of Steps")
```

<img src="figure/histogram2-1.png" title="plot of chunk histogram2" alt="plot of chunk histogram2" style="display: block; margin: auto;" />

Code for calculating the mean of cleaned data:


```r
MeanResultClean<-mean(CleanDataResult$steps)
```

The mean of the cleaned data is 10766

Code for calculating the median:


```r
MedianResultClean<-median(CleanDataResult$steps)
```

The median of the cleaned data is 10766

The mean of the original dataset and the clean dataset is same. However, the median of the original dataset and the clean dataset is different. Filling the missing value to the dataset makes the mean and the median of the dataset to be equal.

## Are there differences in activity patterns between weekdays and weekends?

The code below help to identify the weekend and weekend of the date and create new variables in the dataset:


```r
library(chron)
CleanData2<-mutate(CleanData,dayofweek= weekdays(as.Date(date)))
CleanData2$period[is.weekend(as.Date(CleanData2$date))]<-"Weekend"
CleanData2$period[!is.weekend(as.Date(CleanData2$date))]<-"Weekday"
CleanData2$period<-as.factor(CleanData2$period)
```

Then calculate the mean based on interval and period (Weekend & Weekday):


```r
library(lattice)
CleanDataPeriod<-aggregate(steps~period+interval, CleanData2, mean)
xyplot(steps~interval| period, data=CleanDataPeriod, type = "l",xlab="Interval",  ylab="Number of Steps",layout=c(1,2))
```

<img src="figure/timeseries1-1.png" title="plot of chunk timeseries1" alt="plot of chunk timeseries1" style="display: block; margin: auto;" />
