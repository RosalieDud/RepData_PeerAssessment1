# Reproducible Data: Peer-graded assignment 1

### Data analysis from a personal activity monitory device

> I added the plots in a different file in this repository

Loading and preprocessing the data


```r
if(!file.exists("./PersonalActivity")){dir.create("./PersonalActivity")}
FileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(FileUrl, destfile = "./PersonalActivity.zip")
MyDataUnzip <- unzip("PersonalActivity.zip", exdir="PersonalActivity")
MyData <- read.csv(MyDataUnzip, header=TRUE, sep=",")
```

###What is mean total number of steps taken per day?

Calculate the total number of steps taken per day


```r
library(dplyr)
descSteps <-
    MyData %>%
    group_by(date) %>%
    summarize(count = n(), sumsteps = sum(steps))
```

Make a histogram of the total number of steps taken each day


```r
histsteps <- hist(descSteps$sumsteps, col = "green", xlab = "Total amount of steps", main = "Total number of steps taken each day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)

Calculate and report the mean and median of the total number of steps taken per day

```r
mean(descSteps$sumsteps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(descSteps$sumsteps, na.rm = TRUE)
```

```
## [1] 10765
```

### What is the average daily activity pattern?

Make a time series plot (i.e. type = l) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
library(ggplot2)
descSteps2 <-
    MyData %>%
    filter(!is.na(steps)) %>%
    group_by(interval) %>%
    summarize(count = n(), meansteps = mean(steps))
```


```r
plot(descSteps2$interval, descSteps2$meansteps, type = "l", col = "green",
     xlab = "interval", ylab = "Steps", main = "5-minute interval and the average number of steps taken")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
MyData %>% group_by(interval) %>% 
    summarize(meanByInterval = mean(steps, na.rm = TRUE)) %>%
    filter(meanByInterval == max(meanByInterval))
```

```
## # A tibble: 1 x 2
##   interval meanByInterval
##      <int>          <dbl>
## 1      835           206.
```

### Imputing missing values

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NA


```r
NaAll <- sum(is.na(MyData$steps))
NaAll
```

```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
MyDataFilledNa <- MyData
nas <- is.na(MyDataFilledNa$steps)
avg_interval <- tapply(MyDataFilledNa$steps, MyDataFilledNa$interval, mean, na.rm=TRUE, simplify=TRUE)
MyDataFilledNa$steps[nas] <- avg_interval[as.character(MyDataFilledNa$interval[nas])]
```

Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
descStepsWithNA <- MyDataFilledNa %>%
    filter(!is.na(steps)) %>%
    group_by(date) %>%
    summarize(steps = sum(steps)) %>%
    print
```

```
## # A tibble: 61 x 2
##    date        steps
##    <fct>       <dbl>
##  1 2012-10-01 10766.
##  2 2012-10-02   126 
##  3 2012-10-03 11352 
##  4 2012-10-04 12116 
##  5 2012-10-05 13294 
##  6 2012-10-06 15420 
##  7 2012-10-07 11015 
##  8 2012-10-08 10766.
##  9 2012-10-09 12811 
## 10 2012-10-10  9900 
## # ... with 51 more rows
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
histNA <- hist(descStepsWithNA$steps, col = "green", xlab = "Total amount of steps", 
               main = "Total number of steps taken each day - with filled in NA'S")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png)


```r
mean(descStepsWithNA$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(descStepsWithNA$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

### Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
MyDataFilledNaWeekdays2 <- MyDataFilledNa

df = data.frame(MyDataFilledNaWeekdays2) 
df$day <- weekdays(as.Date(df$date))
df$day[df$day == "zaterdag"] <- "Weekend"
df$day[df$day == "zondag"] <- "Weekend"
df$day[df$day == "maandag"] <- "Weekday"
df$day[df$day == "dinsdag"] <- "Weekday"
df$day[df$day == "woensdag"] <- "Weekday"
df$day[df$day == "donderdag"] <- "Weekday"
df$day[df$day == "vrijdag"] <- "Weekday"
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
lastplot <- df %>%
    group_by(interval, day) %>%
    summarise(steps = mean(steps))

series <- ggplot(lastplot, aes(x=interval, y=steps, color = day)) +
    geom_line() +
    facet_wrap(~day, ncol = 1, nrow=2)
print(series)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png)
