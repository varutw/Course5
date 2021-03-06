---
title: "PA1_template"
output: 
  html_document:
    keep_md: true
---

1. Code for reading in the dataset and/or processing the data
```{r readdata, echo=TRUE}
setwd("D:/Google Drive/dataScientist")
data<-read.csv("activity.csv")
```

Here is the first 5 rows of data

```{r data, echo=TRUE}
head(data)
```

2. Histogram of the total number of steps taken each day

```{r histogram, echo=TRUE}
Sys.setlocale("LC_TIME", "C")
library(dplyr)
library(ggplot2)
data$date<- as.Date(data$date)
bin<-list()
for (i in unique(data$date)){
  if(i==unique(data$date)[1]){k<-1}
  cal<-subset(data,date==i)
  summation<-data.frame(sum(cal$steps,na.rm = TRUE))
  summation$date<-as.Date(i,origin="1970-01-01")
  bin[[k]]<-summation
  k<-k+1
}
dataComplete <-bind_rows(bin)
colnames(dataComplete)<-c("steps","date")

ggplot(dataComplete,aes(x=date,y=steps))+geom_histogram(stat = "identity",color="green")+ggtitle("Histogram of the total number of steps taken each day")
```

3. Mean and median number of steps taken each day

```{r meanandmedian, echo=TRUE}
bin<-list()
for (i in unique(data$date)){
  if(i==unique(data$date)[1]){k<-1}
  cal<-subset(data,date==i)
  dataNew<-data.frame(mean(cal$steps,na.rm=TRUE))
  dataNew$median<-median(cal$steps,na.rm=TRUE)
  dataNew$date<-as.Date(i,origin="1970-01-01")
  bin[[k]]<-dataNew
  k<-k+1
  }
dataComplete3 <-bind_rows(bin)
colnames(dataComplete3)<-c("mean","median","date")
```

Here is the first 5 rows of mean and median number of steps taken each day

```{r meanandmedianresult, echo=TRUE}
head(dataComplete3)
```

4. Time series plot of the average number of steps taken

```{r timeserires, echo=TRUE}
library(data.table)
library(lubridate)
dataNew4<-dcast(data,interval~date,value.var="steps")
dataNew4$mean<-rowMeans(dataNew4[,2:ncol(dataNew4)],na.rm = TRUE)
dataNew4$intervalNew <- sprintf("%04d",dataNew4$interval)
dataNew4$intervalNew <- strptime(dataNew4$intervalNew, format="%H%M")

plot(x=dataNew4$intervalNew,y=dataNew4$mean,type="l",
     main="Time series plot of the average number of steps taken",
     xlab="time",ylab="steps",col="blue",lwd=2)

```

5. The 5-minute interval that, on average, contains the maximum number of steps

```{r maxintervalintime, echo=TRUE}
maxInterval<-dataNew4$intervalNew[which.max(dataNew4$mean)]
maxInterval<-parse_date_time(maxInterval,order="YmdHMS")
paste0(hour(maxInterval),":",minute(maxInterval))
```

6. Code to describe and show a strategy for imputing missing data

First, let's look at how many NA data is by calculating the average
NA data of all columns.

```{r investigate, echo=TRUE}
paste("percentage of NA data in step column is",mean(is.na(data$steps))*100,"%")
paste("percentage of NA data in interval column is",mean(is.na(data$interval))*100,"%")
paste("percentage of NA data in date column is",mean(is.na(data$date))*100,"%")
```

It can be seen that there is no missing data in interval and date cloumns. However, there are 13.11% of missing values in step column which can be considered as significant.

Therefore, the best solution is to replace NA values in steps column with the mean of steps in each day (exclude NA value in calculation). However, when investigeting in the result, it

```{r impute, echo=TRUE}
dataNew6<-dcast(data,date~interval,value.var="steps")
dataNew6$mean<-rowMeans(dataNew6[,2:ncol(dataNew6)],na.rm = TRUE)

```
However, when investigeting in the result, it is seen that there are days that all the values of step is NA, Hence, alternative solution is to use the means of step in each 5-minutes interval that already calculated in problem 4. instead.

```{r impute2, echo=TRUE}
dataImputed<-data
for (i in unique(data$interval)){
  dataImputed$steps[dataImputed$interval==i&is.na(dataImputed$step)]<-dataNew4$mean[dataNew4$interval==i]
}

```
Here is the first 5 rows of imputed data
```{r dataimputed, echo=TRUE}
head(dataImputed)
```
7. Histogram of the total number of steps taken each day after missing values are imputed

Calculate the sum of steps taken each day from dataImputed.

```{r sum2, echo=TRUE}
dataNew7<-dcast(dataImputed,date~interval,value.var="steps")
dataNew7$total<-rowSums(dataNew7[,2:ncol(dataNew7)])
```

plot the histogram with ggplot2

```{r histogram2, echo=TRUE}
ggplot(dataNew7,aes(x=date,y=total))+geom_histogram(stat = "identity",color="green")+ggtitle("Histogram of the total number of steps taken each day after missing \n values are imputed")
```

8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

Split days of the week into weekday and weekend, and then use "weekdays" function to match the day
```{r data8, echo=TRUE}
weekDay<-c("Monday","Tuesday","Wednesday","Thursday","Friday")
weekEnd<-c("Saturday","Sunday")
dataImputed$dayOfWeek[weekdays(dataImputed$date) %in% weekDay]<-"weekday"
dataImputed$dayOfWeek[weekdays(dataImputed$date) %in% weekEnd]<-"weekend"
dataNew8<-dcast(dataImputed,interval~dayOfWeek,mean,value.var = "steps")
dataNew8$intervalNew <- sprintf("%04d",dataNew8$interval)
dataNew8$intervalNew <- strptime(dataNew8$intervalNew, format="%H%M")

``` 
plot the result with base plot

```{r plot8, echo=TRUE}
par(mfrow = c(2, 1))
plot(x=dataNew8$intervalNew,y=dataNew8$weekday,type="l",
     main="weekday",
     xlab="time",ylab="steps",col="blue",lwd=2)
plot(x=dataNew8$intervalNew,y=dataNew8$weekend,type="l",col="red",lwd=2,
     main="weekend",xlab="time",ylab="steps")
``` 