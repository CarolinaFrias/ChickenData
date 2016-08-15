---
title: "ChickenData example"
author: "Andreia Carlos"
date: "12 August 2016"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Introduction

This is an exercise with the ChickenWeight data from R Datasets Package. 

The aim is to get data into a specific format to be used in a shinyApp. We want to present data by Time, Diet and the summary statistics for the chickens weight. For data processing and analysis we used [dplyr package](https://github.com/hadley/dplyr). 

## Data Processing 

**Steps:**

1. Load ```ChickWeight``` data
2. Produce a new dataset with summaries of chicken weight, including ```Number of cases (N)```, ```Mean```, ```Standard Deviation (SD)```, ```Median```, ```Minimum (Min)```and ```Maximum (Max)```values.
3. Make the dataset into a "longer" format by converting the columns with statistical calculations into a column ```Stats```and a column ```Values```. 
4. Move ```Diet``` groups from rows into columns.
5. Format data with *sprintf* function to present statistics into a consistent format, Mean as ```Mean(SD)``` and maximum and minimum values together as ```Min-Max```.


**1. Load ChickWeight data**
```{r ChickenWeight}
data("ChickWeight")
summary(ChickWeight)
```

Load required libraries
```{r load required libraries, message = FALSE}
library(dplyr) 
library(tidyr) #for 'unite' function
library(knitr)
```



**2. Produce a new dataset with summaries of chicken weight**
```{r Data Summaries}

DataStats <- ChickWeight %>%
        select(weight, Time, Diet) %>%
        group_by(Diet, Time) %>%
        summarise(
                N = n(),
                Mean = mean(weight),
                SD = sd(weight),
                Median = median(weight),
                Min = min(weight),
                Max = max(weight)
                ) %>%
        unite(Mean_SD, Mean, SD, sep = ".", remove = FALSE) %>% #Create new variable with MeanSD 
        unite(Range, Min, Max, sep = "-", remove = FALSE)  #Create new column with Min-Max into the same column 


kable(head(DataStats))
```

DataStats has 48 observations and 10 variables



**3. Make the dataset into a "longer" format by converting the columns with statistical calculations into a column**
```{r Long Data format with Statistics moving from columns to rows}

LongData <- DataStats  %>%
        gather("Statistics", "Value", 3:10) #selecting the columns that we want to rearrange 

kable(head(LongData))
```

LongData has 384 observations and 4 variables


**4. Move ```Diet``` groups from rows into columns. This time using numbers attached to the columns of statistics so that we can reorder them afterwards**
```{r Diets by Time}
TimesByDiet <- ChickWeight %>%
        select(weight, Time, Diet) %>%
        group_by(Diet, Time) %>%
        summarise( #Add numbers to the names of each column to order it afterwords
                N.1 = n(), 
                Mean.2 = mean(weight),
                SD.3 = sd(weight),
                Median.4 = median(weight),
                Min.5 = min(weight),
                Max.6 = max(weight)
                ) %>%
        unite(Mean_SD.8, Mean.2, SD.3, sep = ".", remove = FALSE) %>%
        unite(Range.9, Min.5, Max.6, sep = "-", remove = FALSE)  %>%
        select(Time, Diet, N.1, Mean_SD.8, Median.4, Range.9) %>%
        gather("Statistics", "Value", 3:6) %>%
        spread(Diet, Value, sep = ".") %>% #Move diet groups from rows to columns
        separate(Statistics, c("Stats", "Order"), sep = "\\.", remove = TRUE) %>% #Separate Stats column into two
        arrange(Time, Order) #reorder the dataset to have statistics by the appropriate order(N, Mean-SD, Median, Range)

kable(head(TimesByDiet))
```

This dataset has 48 observations and 7 variables


**5. Format data with *sprintf* function to present statistics into a consistent format**
```{r Prepare data for printing version}

#By using mutate and sprintf, there is no longer need to attach numbers to the variable names to put them in the right order, as they come in the order that we input in the code

Chick_print <- ChickWeight %>%
        select(weight, Time, Diet) %>%
        group_by(Diet, Time) %>%
        summarise( 
                N = n(),
                Mean = mean(weight),
                SD = sd(weight),
                Median = median(weight),
                Min = min(weight),
                Max = max(weight)
                ) %>%
        mutate( #Creating new variables merging the previous ones and giving it the right format
                pN = as.character(N), 
               pMeanSD = sprintf("%6.1f(%6.2f)", Mean, SD),
               pMedian = sprintf("%6.1f", Median),
               pMinMax = sprintf("%6.1f-%6.1f", Min, Max)
               ) %>%
        select(Diet, Time, N, pMeanSD, pMedian, pMinMax) %>% #select only the variables of interest
        gather("Statistics", "Value", 3:6) %>% #Get statistics into rows
        spread(Diet, Value, sep = ".") #Produce a dataset with diets in columns

kable(head(Chick_print))
```

This dataset has 48 observations and 6 variables

