**Reproducible Research: Peer Assessment 1**
========================================
By Djoko Soehartono

### **R Preparation and Loading the Necessary Packages**
In this document code will be represented to show how the results have been achieved. Set the default of echo to be true throughout the document and loading the necessary packages:

```r
library(knitr)
opts_chunk$set(echo = TRUE)

library(dplyr)
library(lubridate)
library(ggplot2)
```


### **Loading and preprocessing the data**
#### **Reading in the data**

```r
unzip(zipfile = "activity.zip")
data <- read.csv("activity.csv", header = TRUE, sep = ',', colClasses = 
                 c("numeric", "character", "integer"))
```
#### **Tidying the data**

```r
data$date <- ymd(data$date)
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : POSIXct, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
head(data)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

### **What is mean total number of steps taken per day?**
For this part of the assignment the missing values can be ignored.
The methodologies:
1. Calculate the total number of steps taken per day.
2. Make a histogram of the total number of steps taken each day.
3. Calculate and report the mean and median of the total number of steps taken per day.

**Methodology and Results**
1. Calculate the total number of steps per day using `dplyr` and group by date:

```r
total_steps <- data %>%
  filter(!is.na(steps)) %>%
  group_by(date) %>%
  summarize(steps = sum(steps)) %>%
  print
```

```
## Source: local data frame [53 x 2]
## 
##          date steps
##        (time) (dbl)
## 1  2012-10-02   126
## 2  2012-10-03 11352
## 3  2012-10-04 12116
## 4  2012-10-05 13294
## 5  2012-10-06 15420
## 6  2012-10-07 11015
## 7  2012-10-09 12811
## 8  2012-10-10  9900
## 9  2012-10-11 10304
## 10 2012-10-12 17382
## ..        ...   ...
```

2. Use `ggplot` for making the histogram:

```r
ggplot(total_steps, aes(x = steps)) +
  geom_histogram(fill = "steelblue", binwidth = 1000) +
  labs(title = "Histogram of Steps per day", 
       x = "Steps per day", y = "Frequency")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

3. Calculate the mean and median of the total number of steps taken per day:

```r
mean(total_steps$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(total_steps$steps, na.rm = TRUE)
```

```
## [1] 10765
```


### **What is the average daily activity pattern?**
1. Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

**Methodology and Result**

1. Calculate the average number of steps taken in each 5-minute interval per day using `dplyr` and group by `interval`:

```r
interval <- data %>%
  filter(!is.na(steps)) %>%
  group_by(interval) %>%
  summarize(steps = mean(steps))
```
Use 'ggplot' for making the time series of the 5-minute interval and average steps taken:

```r
ggplot(interval, aes(x=interval, y=steps)) +
  geom_line(color = "steelblue")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

2. Use `which.max()` to find out the maximum steps, on average, across all the days:

```r
interval[which.max(interval$steps),]
```

```
## Source: local data frame [1 x 2]
## 
##   interval    steps
##      (int)    (dbl)
## 1      835 206.1698
```

The interval 835 has the maximum average count of steps with 206 steps.


### **Imputing missing values**
Note that there are a number of days/intervals where there are missing values (coded as `NA`). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`s).
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
4. Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

**Methodology and Result**

1. Summarize all the missing values:

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

2. Taking the approach to fill in a missing `NA` with the average number of steps in the same 5-min interval.
3. Create a new dataset as the original and use tapply for filling in the missing values with the average number of steps per 5-minute interval:

```r
data_full <- data
nas <- is.na(data_full$steps)
avg_interval <- tapply(data_full$steps, data_full$interval, mean, na.rm=TRUE, simplify=TRUE)
data_full$steps[nas] <- avg_interval[as.character(data_full$interval[nas])]
```
Check if there is no missing values:

```r
sum(is.na(data_full$steps))
```

```
## [1] 0
```

4. Calculate the number of steps taken in each 5-minute interval per day using `dplyr` and group by `interval`. Use `ggplot` to make the histogram:

```r
steps_full <- data_full %>%
  filter(!is.na(steps)) %>%
  group_by(date) %>%
  summarize(steps = sum(steps)) %>%
  print
```

```
## Source: local data frame [61 x 2]
## 
##          date    steps
##        (time)    (dbl)
## 1  2012-10-01 10766.19
## 2  2012-10-02   126.00
## 3  2012-10-03 11352.00
## 4  2012-10-04 12116.00
## 5  2012-10-05 13294.00
## 6  2012-10-06 15420.00
## 7  2012-10-07 11015.00
## 8  2012-10-08 10766.19
## 9  2012-10-09 12811.00
## 10 2012-10-10  9900.00
## ..        ...      ...
```

```r
ggplot(steps_full, aes(x = steps)) +
  geom_histogram(fill = "steelblue", binwidth = 1000) +
  labs(title = "Histogram of Steps per day, including missing values", 
       x = "Steps per day", y = "Frequency")
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png) 

Calculate the mean and median steps with the filled in values:

```r
mean(steps_full$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(steps_full$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```
The impact of imputing missing data with the average number of steps in the same 5-min interval is that both the mean and the median are equal to `10766`


### **Are there differences in activity patterns between weekdays and weekends?**
1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
2. Make a panel plot containing a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

**Methodology and Result**

1. Use `dplyr` and `mutate` to create a new column, `weektype`, and apply whether the day is weekend or weekday:

```r
data_full <- mutate(data_full, weektype = ifelse(weekdays(data_full$date) == 
                    "Saturday" | weekdays(data_full$date) == "Sunday", 
                    "weekend", "weekday"))
data_full$weektype <- as.factor(data_full$weektype)
head(data_full)
```

```
##       steps       date interval weektype
## 1 1.7169811 2012-10-01        0  weekday
## 2 0.3396226 2012-10-01        5  weekday
## 3 0.1320755 2012-10-01       10  weekday
## 4 0.1509434 2012-10-01       15  weekday
## 5 0.0754717 2012-10-01       20  weekday
## 6 2.0943396 2012-10-01       25  weekday
```

2. Calculate the average steps in the 5-minute interval and use `ggplot` for making the time series of the 5-minute interval for weekday and weekend, and compare the average steps:

```r
interval_full <- data_full %>%
  group_by(interval, weektype) %>%
  summarise(steps = mean(steps))
g <- ggplot(interval_full, aes(x=interval, y=steps, color = weektype)) +
  geom_line() +
  facet_wrap(~weektype, ncol = 1, nrow=2)
print(g)
```

![plot of chunk unnamed-chunk-16](figure/unnamed-chunk-16-1.png) 

From the plots it seems the object is more active throughout the day during weekends than on weekdays.
