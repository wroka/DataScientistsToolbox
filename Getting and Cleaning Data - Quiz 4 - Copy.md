---
title: "Getting and Cleaning Data - Quiz 4"
author: "Bill Roka"
date: "March 14, 2016"
output: word_document
---

# Getting and Cleaning Data: Quiz 4

## QUESTION 1

The American Community Survey distributes downloadable data about United States communities. Download the 2006 microdata survey about housing for the state of Idaho using download.file() from here:
     
 <https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06hid.csv>
 
and load the data into R. The code book, describing the variable names is here:
     
 <https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FPUMSDataDict06.pdf>
 
Apply strsplit() to split all the names of the data frame on the characters "wgtp". What is the value of the 123 element of the resulting list?

```{r, eval=FALSE}
library(dplyr)
library(plyr)

download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06hid.csv", destfile = 'q4q1.csv')
quest1 <- read.csv('q4q1.csv', header = TRUE)

quest1a <- strsplit(colnames(quest1), 'wgtp')
quest1a[[123]]
# [1] ""   "15"

# alternate
strsplit(x = names(quest1), split = "wgtp")[[123]]
# [1] ""   "15"
```

## QUESTION 2

Load the Gross Domestic Product data for the 190 ranked countries in this data set:
     
 <https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv>
 
Remove the commas from the GDP numbers in millions of dollars and average them. What is the average?
 
Original data sources:
     
 <http://data.worldbank.org/data-catalog/GDP-ranking-table>

```{r, eval=FALSE}
install.packages("data.table")
library(data.table)

download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv", destfile = 'q2.csv')
gdp <- read.csv('q2.csv', skip = 4, nrows = 190)
gdpnocomma <- gsub(",", "", gdp$X.4) # removes the comma with gsub function. X.4 is the default column name for gdp.
gdpnocomma <- (as.numeric(gdpnocomma))
mean(gdpnocomma,na.rm = TRUE)
# [1] 377652.4

# alternate version: fread, with pipes. Note that with fread, we skip the read.csv step and bring in the url itself as "url"
url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv"
gdp = fread(url, skip="USA", nrows = 190, select = c(1, 2, 4, 5), col.names=c("CountryCode", "Rank", "Economy", "Total"))
gsub(",", "", gdp$Total) %>% # remove comma
  as.numeric() %>% # convert to numeric
    mean(na.rm = TRUE) # find the mean, removing NAs
# [1] 377652.4
```

## QUESTION 3

In the data set from Question 2 what is a regular expression that would allow you to count the number of countries 
whose name begins with "United"? Assume that the variable with the country names in it is named countryNames. 

How many countries begin with United?

```{r, eval=FALSE}
countrynames <- gdp$Economy
grep("^United",countrynames) # the "^" symbol selects values starting with "United". A "*" would select values that end with "United" ("*United").
length(grep("^United",countrynames))
# [1] 3
```

## QUESTION 4

Load the Gross Domestic Product data for the 190 ranked countries in this data set:
     
 <https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv>
 
Load the educational data from this data set:
     
 <https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FEDSTATS_Country.csv>
 
Match the data based on the country shortcode. Of the countries for which the end of the fiscal year is available, how many end in June?
 
Original data sources:
     
 <http://data.worldbank.org/data-catalog/GDP-ranking-table>
 
 <http://data.worldbank.org/data-catalog/ed-stats>

```{r, eval=FALSE}
library(dplyr)

download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FEDSTATS_Country.csv", destfile = 'edu.csv')
edu <- read.csv('edu.csv', header = TRUE)

download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv", destfile = 'q2.csv')
gdp <- read.csv('q2.csv', skip = 4, nrows = 190)

gdp <- rename(gdp, c("X" = "CountryCode", "X.1" = "Rank", "X.3"="CountryName", "X.4"="GDP"))
gdp <- gdp[,c("CountryCode","Rank","CountryName","GDP")]
combo <- left_join(gdp, edu, by = "CountryCode")

combojune <- grep("Fiscal year end: June",combo$Special.Notes)
length(combojune)
# [1] 13
```

## QUESTION 5

You can use the quantmod (http://www.quantmod.com/) package to get historical stock prices for publicly traded companies 
on the NASDAQ and NYSE. Use the following code to download data on Amazon's stock price and get the times 
the data was sampled.
 
 library(quantmod)
 amzn = getSymbols("AMZN",auto.assign=FALSE)
 sampleTimes = index(amzn)

How many values were collected in 2012? How many values were collected on Mondays in 2012?

```{r, eval=FALSE}
install.packages("quantmod")
library(quantmod)
install.packages("lubridate")
library(lubridate)

amzn = getSymbols("AMZN",auto.assign=FALSE)
sampleTimes = index(amzn)
# find how many true values there are for 2012 rows in the sampleTimes set
year <- which(grepl("2012-",sampleTimes))
length(year)
# [1] 250

# mondays 2012
year2012 <- grepl("2012-",sampleTimes)
year2012 <- subset(sampleTimes, year2012)
# note - may need to detatch data.table if running concurrently with lubridate for wday to work. 
monday2012 <- wday(year2012, label = TRUE) == "Mon"
monday2012 <- which(monday2012) # find how many rows of Mondays are "true"
length(monday2012)
# [1] 47
```
