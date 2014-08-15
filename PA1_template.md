Peer Assessment 1
========================================================

---
title: "Prueba"
author: "FChn"
date: "Thursday, August 14, 2014"
output: html_document
---
```{r, echo=FALSE}
setwd("C:/Users/VXL/Documents/Cursos Internet/Data Specialization/5_Reproducible Research/Peer1")
```

## 1. Loading and processing the data
The data must be in the working directory and are gonna be loaded in to the "raw" table.
The NA values are gonna be removed using the complete.cases function and the new table will be named "data"

```{r}
raw <- read.csv("activity.csv", header = TRUE, sep = ",", stringsAsFactors = FALSE, na.strings = "NA")
data <- raw[complete.cases(raw),1]
```

## 2. What is mean total number of steps taken per day?
First we are gonna calculate the sum of all the steps per day using the tapply function.

```{r}
d2 <- tapply(data[,1],data[,2],FUN = sum)
```


```{r}
hist(d2, xlab = "Steps", col = "red", main = "Histogram of Steps")
```