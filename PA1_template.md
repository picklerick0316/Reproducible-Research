Activity monitoring data
================

-1. Code for reading in the dataset and processing the data

``` r
file_url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
AMD_zip <- "./AMD.zip"
AMD_file <- "./Activity monitoring data"
if (file.exists(AMD_zip) == FALSE) {
  download.file(file_url, 
                destfile = AMD_zip)
}
if (file.exists(AMD_file) == FALSE) {
  unzip(AMD_zip)
}
ADM <- read.csv("activity.csv")
ADM$date <- as.Date(ADM$date)
ADM_complte <- na.omit(ADM)
```

-2. Histogram of the total number of steps taken each day

``` r
ADM_perday <- aggregate(steps~date, 
                        ADM_complte, 
                        sum)
hist(ADM_perday$steps,
     main = "Total number of steps taken per day", 
     xlab = "Steps taken per day",
     breaks = 53) 
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-2-1.png)

-3. Mean and median number of steps taken each day

``` r
mean(ADM_perday$steps)
```

    ## [1] 10766.19

``` r
median(ADM_perday$steps)
```

    ## [1] 10765

-4 . Time series plot of the average number of steps taken

``` r
ADM_interval <- aggregate(steps ~ interval, 
                          ADM_complte, 
                          mean)
plot(ADM_interval$interval, 
     ADM_interval$steps, 
     type='l', 
     main="Average daily activity pattern", 
     xlab="interval", 
     ylab="number of steps")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-4-1.png)

-5. The 5-minute interval that, on average, contains the maximum number of steps

``` r
ADM_max <- which.max(ADM_interval$steps)
ADM_interval[ADM_max, ]
```

    ##     interval    steps
    ## 104      835 206.1698

-6. Code to describe and show a strategy for imputing missing data

``` r
data_NA <- ADM[!complete.cases(ADM),]
missing_data <- nrow(data_NA)
for (i in 1:nrow(ADM)) {
  if(is.na(ADM$steps[i])) {
    value <- ADM_interval$steps[which(ADM_interval$interval == ADM$interval[i])]
    ADM$steps[i] <- value 
  }
}
ADM_imputing <- aggregate(steps ~ date, 
                          ADM, 
                          sum)
```

-7. Histogram of the total number of steps taken each day after missing values are imputed

``` r
ADM_perday2 <- aggregate(steps~date, 
                         ADM_imputing, 
                         sum)
hist(ADM_perday2$steps,  
     main = "Total number of steps taken per day", 
     xlab = "Steps taken per day",
     breaks = 61) 
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-7-1.png)

``` r
mean(ADM_perday2$steps)
```

    ## [1] 10766.19

``` r
median(ADM_perday2$steps)
```

    ## [1] 10766.19

-8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

``` r
week <- function(dates) {
  weekly <- weekdays(as.Date(dates, 
                             '%Y-%m-%d'))
  if  (!(weekly == "s攼㸱bado" || weekly == "domingo")) {
    x <- "Weekday"
  } else {
    x <- "Weekend"
  }
  x
}
ADM$day_type <- as.factor(sapply(ADM$date, 
                                 week))
ADM_week <- aggregate(steps ~ interval + day_type, 
                      ADM, 
                      mean)

library(ggplot2)

ggplot(ADM_week, 
       aes(interval, 
           steps)) + 
  geom_line() + 
  facet_grid(rows = vars(day_type)) +
  labs(x="Interval", 
       y=expression("Steps")) +
  ggtitle("Average number of steps taken") +
  theme(plot.title = element_text(hjust = 0.5))
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-9-1.png)
