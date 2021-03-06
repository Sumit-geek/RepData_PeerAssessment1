# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

1. **Remove all variables from the environment to setup a clear environment**

```r
#clear environment vars
rm(list=ls(all=TRUE)) 
```

2. **Read the data into dataframe**

```r
activity <- read.csv('activity.csv', header = T)
```

## What is mean total number of steps taken per day?

Summarize the data by day:


```r
library(data.table)
activity_tbl <- data.table(activity)
activity_tbl_summary <- activity_tbl[, list(total_steps = sum(steps, na.rm = T)), 
                          by = date]
```

**Create histogram and report on mean and median:**

Following items will be addressed in below histogram:

*  Make a histogram of the total number of steps taken each day
*  Calculate and report the mean and median total number of steps taken per day



```r
library(ggplot2)
# Single function to create histogram and add lines and legend for mean and median
plot_hist <- function(x, title){
        hist(x, 
             breaks = 20,
             main = title,
             xlab = 'Number of steps taken', col = 'light grey',
             cex.main = .9)
        
        #caluclate mean and median
        meanVal = round(mean(x), 1)
        medianVal = round(median(x), 1)
        
        #draw lines for mean and median on histogram
        abline(v=meanVal, lwd = 3, col = 'red')
        abline(v=medianVal, lwd = 3, col = 'blue')
        
        #legend
        legend('topright', lty = 1, lwd = 3, col = c("red", "blue"),
               cex = .8, 
               legend = c(paste('Median: ', medianVal),
                          paste('Mean: ', meanVal))
               )
}

plot_hist(activity_tbl_summary$total_steps, 'Number of steps taken each day')
```

![](figure/unnamed-chunk-3-1.png)<!-- -->

## What is the average daily activity pattern?

Following items will be addressed in below plot:

*  Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
*  Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?



```r
#summarize dataset by interval
activity_tbl_summary_intv = activity_tbl[, list(avg_steps = mean(steps, na.rm = T)), 
                          by = interval]
#plot the time series
with(activity_tbl_summary_intv, {
        plot(interval, avg_steps, type = 'l',
             main = 'Average steps by 5-minute interval',
             xlab = '5-minute interval',
             ylab = 'Average number of steps taken')
        })
#find Interval with maximum avg steps
max_steps = activity_tbl_summary_intv[which.max(avg_steps), ]

#legend label String
max_lab = paste('Maximum steps taken =', round(max_steps$avg_steps, 1), '\n @Time interval=', max_steps$interval, sep = '')

#get cooridinates of max Interval
points(max_steps$interval,  max_steps$avg_steps, col = 'blue', lwd = 3, pch = 19)

#Add legend for maximum # steps and interval
legend("topright",
       legend = max_lab,
       text.col = 'blue',
       bty = 'n'
       )
```

![](figure/unnamed-chunk-4-1.png)<!-- -->


## Imputing missing values
1. Calculate & report number of missing values

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

* Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

* Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
#join the dataframe return(y)average number of steps per interval to the original dataset
setkey(activity_tbl, interval)
setkey(activity_tbl_summary_intv, interval)

#create new dataset having average values in place of NAs 
activity_tbl_miss = activity_tbl[activity_tbl_summary_intv]
activity_tbl_miss$new_steps = mapply(function(val1,val2){if(is.na(val1)) return(val2) else return(val1) },activity_tbl_miss$steps, activity_tbl_miss$avg_steps)

#summaryize new dataset by day
activity_tbl_summary_miss = activity_tbl_miss[, list(new_steps = sum(new_steps, na.rm = T)), 
                          by = date]
```

2.  Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?



```r
plot_hist(activity_tbl_summary$total_steps, 'Without missing values')
```

![](figure/unnamed-chunk-7-1.png)<!-- -->

```r
# As mean and median will be same in this case so median will be overlapped by mean line
plot_hist(activity_tbl_summary_miss$new_steps, 'NA values replaced with \n mean of intervals')
```

![](figure/unnamed-chunk-7-2.png)<!-- -->


## Are there differences in activity patterns between weekdays and weekends?

*  Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
#Add name of the day
activity_tbl_miss$dayname = weekdays(as.Date(activity_tbl_miss$date))

#Add a factor variable to seperate weekday and weekend
activity_tbl_miss$daytype = as.factor(apply(as.matrix(activity_tbl_miss$dayname), 1, function(day){if(day %in% c('Saturday', 'Sunday')) return('Weekend') else return('Weekday')}))

#Summarize the dataset: Mean grouped by interval and daytype
activity_tbl_summary_miss = activity_tbl_miss[, list(avg_steps = mean(new_steps, na.rm = T)), 
                          by = list(interval, daytype)]
```

Panel plot:

```r
library(lattice)
xyplot(avg_steps~interval | daytype, data = activity_tbl_summary_miss,
      type = 'l',
      xlab = 'Interval',
      ylab = 'Number of steps',
      layout = c(1,2))
```

![](figure/unnamed-chunk-9-1.png)<!-- -->
