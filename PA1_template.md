# Reproducible Research: Peer Assessment 1



## Loading and preprocessing the data


### Global Options and Initial Setup
First of all, I'll configure the global settings to use **echo = TRUE** on every single code chunk. This is to ensure all the code used to generate the report is shown.


```r
library(knitr)
opts_chunk$set(echo = TRUE)
```

**Important:** don't forget to set your working directory to the location of this file.   
Use the following code chunk as a placeholder and enter your own path.  
Or leave it as is if you have already took care of it.


```r
setwd("C:\\santiago.silvestrini\\Training\\Coursera\\JH DS 05\\PA1\\RepData_PeerAssessment1")
```

### Extract and Load the Data
The Data used on this assignment can be downloaded from [here](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip).  
But as the GitHub repository is already containing it there is no need to download it from the web.  
If you have forked/cloned the repository correctly as well as configured your working directory as instructed above, then the zip file will be located in the current folder.  
The following piece of code will extract the .csv file from it:


```r
unzip("activity.zip")
```

Next step will be to load the .csv file into memory:

```r
data <- read.table( file = "activity.csv"
                    , header = TRUE
                    , sep = ","
                    , na.strings = "NA"
                    , stringsAsFactors = FALSE)
```

### Quick look at the data

According to the assignment, the variables included in the dataset are:

* **steps:** Number of steps taking in a 5-minute interval (missing values are coded as *NA*)
* **date:** The date on which the measurement was taken in YYYY-MM-DD format
* **interval:** Identifier for the 5-minute interval in which measurement was taken
The dataset is stored in a comma-separated-value (CSV) file and there are a total of **17,568** observations in this dataset.

So, to ensure it was loaded correctly I'm going to explore it a litle bit:


```r
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
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

Will be useful to convert the **date** field to a date format instead of char.  


```r
data$date <- as.Date(data$date)
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

## What is mean total number of steps taken per day?

### 1. Calculate the total number of steps taken per day.  

For this step we I'll make use of the [sqldf package](https://cran.r-project.org/web/packages/sqldf/sqldf.pdf)


```r
suppressMessages(library(sqldf))
stepsperday <- sqldf("SELECT date
                            , SUM(steps) TotalSteps 
                     FROM data 
                     WHERE steps != 'NA' 
                     GROUP BY date ORDER BY date")
```

```
## Loading required package: tcltk
```

```r
head(stepsperday)
```

```
##         date TotalSteps
## 1 2012-10-02        126
## 2 2012-10-03      11352
## 3 2012-10-04      12116
## 4 2012-10-05      13294
## 5 2012-10-06      15420
## 6 2012-10-07      11015
```

```r
tail(stepsperday)
```

```
##          date TotalSteps
## 48 2012-11-24      14478
## 49 2012-11-25      11834
## 50 2012-11-26      11162
## 51 2012-11-27      13646
## 52 2012-11-28      10183
## 53 2012-11-29       7047
```

### 2. Histogram of the total number of steps taken each day
If you do not understand the [difference between a histogram and a barplot](http://stattrek.com/statistics/charts/histogram.aspx?Tutorial=AP), research the difference between them. Make a histogram of the total number of steps taken each day


```r
library(ggplot2)
his <- ggplot(stepsperday, aes(x = TotalSteps)) +
    geom_histogram(binwidth = 3000, alpha = .7, aes(fill = ..count..)) +
    scale_fill_gradient("Count", low = "green", high = "red") +
    ggtitle("Histogram of the total number of steps \n taken each day") +
    labs(x = "Total Steps per Day", y = "Frequency") +
    theme(plot.title = element_text(color = "#666666", face = "bold", size = 14, hjust = 0.5)) +
    theme(axis.title = element_text(color = "#666666", size = 12))
print(his)
```

![](PA1_template_files/figure-html/Histogram Total Number of Steps-1.png) 

### 3. Calculate and report the mean and median of the total number of steps taken per day


```r
## Compute the Mean excluding NAs
exmean <- round(mean(stepsperday$TotalSteps, na.rm = TRUE),2)

## Compute the Median excluding NAs
exmedian <- round(median(stepsperday$TotalSteps, na.rm = TRUE),2)

## Plot the Median and Mean in the Histogram (2)
hismm <- ggplot(stepsperday, aes(x = TotalSteps))
hismm + geom_histogram(binwidth = 3000, alpha = .7, aes(fill = ..count..)) +
    scale_fill_gradient("Count", low = "green", high = "red") +
    ggtitle("Histogram of the total number of steps \n taken each day") +
    labs(x = "Total Steps per Day", y = "Frequency") +
    theme(plot.title = element_text(color = "#666666", face = "bold", size = 14, hjust = 0.5)) +
    theme(axis.title = element_text(color = "#666666", size = 12)) +
    geom_vline(aes(xintercept = mean(TotalSteps, na.rm = TRUE))
                , color = "blue", linetype = "dashed", alpha = .7, size = 0.5) +
    geom_vline(aes(xintercept = median(TotalSteps, na.rm = TRUE))
               , color = "white", linetype = "dashed", alpha = .7, size = 0.5)
```

![](PA1_template_files/figure-html/Mean and Median-1.png) 

The **Median** of the total number of steps taken per day is 10765 while the **Mean** has a value of 10766.19 (excluding NAs).

*Although there is no need to filter out the NAs because we already did it in a previous step (check* **Get and Group Data chunk***), I just want to make it explicit in this section of the code to avoid any confusion and also in case I decide to change that later*  

*Also please note that it would be hard to appreciate both median and mean in the chart as are pretty close to each other*

## What is the average daily activity pattern?
### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
## Group the data by intervals and compute the avg of steps
stepsbyinterval <- sqldf("SELECT 
                            interval
                            , AVG(steps) AvgSteps 
                         FROM data 
                         WHERE steps != 'NA' 
                         GROUP BY interval 
                         ORDER BY interval")

## A quick look at the resultant data
head(stepsbyinterval)
```

```
##   interval  AvgSteps
## 1        0 1.7169811
## 2        5 0.3396226
## 3       10 0.1320755
## 4       15 0.1509434
## 5       20 0.0754717
## 6       25 2.0943396
```

```r
tail(stepsbyinterval)
```

```
##     interval  AvgSteps
## 283     2330 2.6037736
## 284     2335 4.6981132
## 285     2340 3.3018868
## 286     2345 0.6415094
## 287     2350 0.2264151
## 288     2355 1.0754717
```

```r
## Plot the time Series chart
ts <- ggplot(stepsbyinterval, aes(x = interval, y = AvgSteps)) 
ts + geom_line(alpha = .5) +
    ggtitle("Time Series of the 5-minute interval \n and avg steps taken across all days") +
    labs(x = "Interval", y = "Average Steps across days") +
    theme(plot.title = element_text(color = "#666666", face = "bold", size = 14, hjust = 0.5)) +
    theme(axis.title = element_text(color = "#666666", size = 12))
```

![](PA1_template_files/figure-html/Time Series-1.png) 

### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
## Show the interval with max amount of avg steps
max <- stepsbyinterval[which.max(stepsbyinterval$AvgSteps),]

## We will use this df for plotting purposes
auxdf <- data.frame(x = max[,1]
                    , y = max[,2]
                    , label = paste('(',max[,1],',',round(max[,2],0),')', sep = ""))

## Plot the time Series chart agin highlighting the Max value
ts <- ggplot(stepsbyinterval, aes(x = interval, y = AvgSteps)) 
ts + geom_line(alpha = .5) +
    ggtitle("Time Series of the 5-minute interval \n and avg steps taken across all days") +
    labs(x = "Interval", y = "Average Steps across days") +
    theme(plot.title = element_text(color = "#666666", face = "bold", size = 14, hjust = 0.5)) +
    theme(axis.title = element_text(color = "#666666", size = 12)) +
    geom_point(data = auxdf, aes(x = x, y = y, color = 'red')) +
    geom_text(data = auxdf, aes(label = label, x = x , y = y, color = 'red', size = 8), hjust = 0, vjust = 0) +
    theme(legend.position = "none")
```

![](PA1_template_files/figure-html/Max Interval-1.png) 

The interval **835** is the one with more steps in average: **206.2** 

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as *NA*). The presence of missing days may introduce bias into some calculations or summaries of the data.

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with *NAs*)


```r
## Count of intervals with missing steps by date
missingdays <- sqldf("SELECT date
                            , COUNT(interval) intervals  
                    FROM data 
                    WHERE steps IS NULL
                    GROUP BY date")
print(missingdays)
```

```
##         date intervals
## 1 2012-10-01       288
## 2 2012-10-08       288
## 3 2012-11-01       288
## 4 2012-11-04       288
## 5 2012-11-09       288
## 6 2012-11-10       288
## 7 2012-11-14       288
## 8 2012-11-30       288
```
There are **2304** intervals with NAs values, which represents a **7.6%** of the total.

### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

I've chosen to go with the approach of replacing the NAs values by the mean for the 5-minute interval.

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
## Copy the data to an auxiliary variable
datacompleted <- data

## stepsbyinterval has the avg step for each interval. We'll use it to replace the missing ones.
for (i in 1:nrow(data)){
    if (is.na(data$steps[i])) {
        datacompleted$steps[i] <-  stepsbyinterval[stepsbyinterval$interval == datacompleted$interval[i],2]
    } 
}

## Check if there is still any missing data
nrow(datacompleted[!complete.cases(datacompleted),])
```

```
## [1] 0
```

### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?



```r
## group the data
newstepsperday <- sqldf("SELECT date, SUM(steps) TotalSteps FROM datacompleted GROUP BY date ORDER BY date")

## Compute the Mean excluding NAs
newmean <- mean(newstepsperday$TotalSteps)

## Compute the Median excluding NAs
newmedian <- median(newstepsperday$TotalSteps)
```



```r
library(gridExtra)

## Plot the Median and Mean in the Histogram (2)
newhis <- ggplot(newstepsperday, aes(x = TotalSteps)) +
    geom_histogram(binwidth = 3000, alpha = .7, aes(fill = ..count..)) +
    scale_fill_gradient("Count", low = "green", high = "red") +
    ggtitle("Histogram of the total number of steps \n taken each day (Completing NAs with Mean)") +
    labs(x = "Total Steps per Day", y = "Frequency") +
    theme(plot.title = element_text(color = "#666666", face = "bold", size = 10, hjust = 0.5)) +
    theme(axis.title = element_text(color = "#666666", size = 8)) +
    coord_cartesian(ylim = c(0, 27)) 

grid.arrange(his + ggtitle("Histogram of the total number of steps \n taken each day (Excluding NAs)") +
                    coord_cartesian(ylim = c(0, 27)) +
                    theme(plot.title = element_text(color = "#666666", face = "bold", size = 10, hjust = 0.5)) +
                    theme(axis.title = element_text(color = "#666666", size = 8))
             ,newhis , ncol = 2)
```

![](PA1_template_files/figure-html/Plot Both Histograms-1.png) 

The new calculated **Median** is 10766.19 while the one excluding NAs was 10765

The new calculated **Mean** is 10766.19 while the one excluding NAs was 10766.19

## Are there differences in activity patterns between weekdays and weekends?

For this part the ***weekdays()*** function may be of some help here. Use the dataset with the filled-in missing values for this part.

### 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data


```r
data$dayname <- weekdays(data$date)
data$daytype <- factor(ifelse( (data$dayname == "Saturday" | data$dayname == "Sunday"), "weekend", "weekday" ))

stepsbyinterval2 <- sqldf("SELECT 
                            interval
                            , daytype
                            , AVG(steps) AvgSteps 
                         FROM data 
                         WHERE steps != 'NA' 
                         GROUP BY interval, daytype
                         ORDER BY interval")

wts <- ggplot(stepsbyinterval2, aes(x = interval, y = AvgSteps, colour = daytype)) 
wts + geom_line(alpha = .5) +
    ggtitle("Comparison of the average activity during Weekends and Weekdays") +
    labs(x = "Interval", y = "Average Steps across days") +
    theme(plot.title = element_text(color = "#666666", face = "bold", size = 12, hjust = 0.5)) +
    theme(axis.title = element_text(color = "#666666", size = 10)) +
    facet_wrap(~daytype, nrow = 2) +
    theme(legend.position = "none")
```

![](PA1_template_files/figure-html/Weekdays vs Weekends-1.png) 

We can see from the above chart that there is more activity on the early hours on weeekdays while on weekends we can see similar spikes during the entire day. Interesting to highlight that on the late hours of the day the subject is more active during weekends than on weekdays. Definitely is not a programmer.
