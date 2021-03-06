# Peer Assessment 1
Thursday, August 14, 2014  


## 1. Loading and processing the data
The data MUST be in the working directory and are gonna be loaded in to the "raw" table.
The NA values are gonna be removed using the complete.cases function and the new table will be named "data"


```r
raw <- read.csv("activity.csv", header = TRUE, sep = ",", stringsAsFactors = FALSE, na.strings = "NA")
data <- raw[complete.cases(raw),]
```

## 2. What is mean total number of steps taken per day?
First we are gonna calculate the sum of all the steps per day using the ddply function.


```r
d2 <- ddply(data,.(date), summarise, TotalSteps = sum(steps))
```

Then we make an histogram of the sum of all steps per day

```r
#png(file = "plot01.png", width = 480, height = 480)
hist(d2[,2], xlab = "Steps", col = "red", main = "Figure 1:\nHistogram of Steps")
```

![plot of chunk unnamed-chunk-4](./PA1_template_files/figure-html/unnamed-chunk-4.png) 

```r
#dev.off()
```

Now we are gonna calculate the mean a median of the total steps made per day.

```r
c(Mean = mean(d2[,2]), Median = median(d2[,2]))
```

```
##   Mean Median 
##  10766  10765
```


## 3. What is the average daily activity pattern?
First we are gonna calculate the mean of steps but aggregating by the inteveral

```r
d3 <- ddply(data,.(interval), summarise, MeanSteps = mean(steps))
```

Then we plot the time series

```r
#png(file = "plot02.png", width = 480, height = 480)
plot(d3, type = "l", col = "red", main = "Figure 2:\nMean of Steps by Intreval")
```

![plot of chunk unnamed-chunk-7](./PA1_template_files/figure-html/unnamed-chunk-7.png) 

```r
#dev.off()
```

Now we found which interval contains the maxium number of steps

```r
subset(d3, d3$MeanSteps==max(d3$MeanSteps))
```

```
##     interval MeanSteps
## 104      835     206.2
```


## 4. Imputing missing values
First we are gonna calculate the number of rows with missing values in the original data

```r
table(complete.cases(raw))["FALSE"]
```

```
## FALSE 
##  2304
```

Now we are gonna fill those missing values in the "steps" column with the mean value in the particular interval. We have calculate the mean for each interval and is already stored in the "d3" dataframe.
The first step is to subset the rows of the original data that has NA values.

```r
data.na <- raw[!complete.cases(raw),]
```

The second step is to make a join (plyr) of the data.na table with the one with the means by interval

```r
d3.na.means <- join(data.na, d3, by = "interval", type = "left", match = "all")
```

The third step is to update the "steps"" column in the dataframe with NA values (data.na) with the values merged in the d3.na.means dataframe.

```r
data.na$steps = d3.na.means$MeanSteps
```

The final step is to make a row bind of the new data.na dataframe with the one with no NA values (data) into a new one and make sort by data and interval

```r
data.complete <- rbind(data,data.na)
data.complete <- data.complete[order(data.complete$date, data.complete$interval),]
```

Note that now the dimension of the raw table is the same that this data.complete dataframe and that the last one has no NA values

```r
dim(raw) == dim(data.complete)
```

```
## [1] TRUE TRUE
```

```r
table(complete.cases(data.complete))
```

```
## 
##  TRUE 
## 17568
```

Lets compare the results of the first part (histogram of total steps by day, and their mean a median)

```r
d4 <- ddply(data.complete,.(date), summarise, TotalSteps = sum(steps))
#png(file = "plot03.png", width = 480, height = 480)
hist(d4[,2], xlab = "Steps", col = "red", main = "Figure 3:\nHistogram of Steps \n(filling NA values)")
```

![plot of chunk unnamed-chunk-15](./PA1_template_files/figure-html/unnamed-chunk-15.png) 

```r
#dev.off()
```

Because of the scale is not easy to compare the two histograms. Lets put them together. For this lets add a flag in d2 and d4 dataframes

```r
d2[,3] <- 'without-NA'
names(d2)[3]<-"Flag"
d4[,3] <- 'filling-NA'
names(d4)[3]<-"Flag"
d5 <- rbind(d2,d4)
#png(file = "plot04.png", width = 480, height = 480)
ggplot(d5, aes(TotalSteps, fill = Flag)) + geom_density(alpha = 0.2) + 
        labs(title = "Figure 4:\nComparision of total steps without NA values \n and filling NA values with the mean")
```

![plot of chunk unnamed-chunk-16](./PA1_template_files/figure-html/unnamed-chunk-16.png) 

```r
#dev.off()
```

Note that the difference between filling the NA values or eliminate them is that more values are concentrated in the mean.

Also note that the mean and median do not change that much. You can indeed appreciate it in the comparision plot

```r
c(Mean = mean(d4[,2]), Median = median(d4[,2]))
```

```
##   Mean Median 
##  10766  10766
```

## 5. Are there differences in activity patterns between weekdays and weekends?
Lets create a new dataframe from the one with the NA filled and add a new column named Weekend where TRUE is when is weekend.

```r
d6 <- cbind(data.complete,Weekend = as.factor(weekdays(as.Date(data.complete$date)) %in% c("Saturday", "Sunday")))
```

Now lets calculte the mean of the steps by interval but considering if it is weekend

```r
d7 <- ddply(d6,.(interval,Weekend), summarise, MeanSteps = mean(steps))
```

Now lets plot

```r
#png(file = "plot05.png", width = 960, height = 480)
ggplot(d7, aes(x=interval, y = MeanSteps, colour = Weekend)) + 
     geom_line() + 
     geom_smooth(method="lm") +
     facet_wrap(~Weekend) + 
     labs(title = "Figure 5:\nComparision of total steps between \n weekdays(FALSE) or weekends(TRUE)")
```

![plot of chunk unnamed-chunk-20](./PA1_template_files/figure-html/unnamed-chunk-20.png) 

```r
#dev.off()
```

There are a clear difference in the activity pattern between weekdays and weekend. In weekdays there are a pick in the early times of the day and then tend to stay stable while in weekdays there is a increasing trend in the activity. This could be explain because in weekdays most of the people works and move to their working places and once there they move less because they are working.

In the weekend people tend to sleep late and don't have activity in early hours. But they tend to increase activity while the time pass because they don't have to remain in just one place.
