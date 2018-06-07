---
layout: post
title:  "Investment considering inflation"
subtitle: "Stock vs. House vs. Saving, 2000.1 ~ 2018.5"
date:   2018-06-07 00:00:00
categories: [experiment]
---
# Are savings risk free investments?
Savings are often considered risk free investments. But actually savings have an inflation risk.
For example, if a bank offer you a 5% interest rate installment saving, it looks good.
However, you need to consider inflation. If the inflation will be higher than 5%, actually you will loose your money.
Even if there is no inflation risk, you have to consider inflation to measure the exact effect of investments.
Although you made your money double for 10 years, it is meaningless if all prices of what you want became double.

Let's see a historical data from 2000.1 to 2018.5.\\
Source data:\\
https://www.dropbox.com/s/h7r3q630o99g4pk/2018-06-07-Investment_considering_inflation-data.csv?dl=0

```r
rm(list=ls())

######################
## Import Libraries ##
######################

library(tidyverse)

###############
## Read Data ##
###############
data <- read.csv("2018-06-07-Investment_considering_inflation-data.csv")
data$date <- as.Date(data$date, "%Y-%m-%d")

consumer_price_not_considered <- data %>%
  mutate(
    KOSPI = KOSPI / KOSPI[1] - 1,
    consumer_price_index = consumer_price_index / consumer_price_index[1] - 1,
    # Calculate monthly returns of CD 91.
    CD_91 = cumprod(((CD_91 / 100) + 1) ^ (1/12)) - 1,
    Seoul_house_price_index = Seoul_house_price_index / Seoul_house_price_index[1] - 1
  )

consumer_price_considered <- data %>%
  mutate(
    KOSPI = KOSPI / KOSPI[1],
    consumer_price_index = consumer_price_index / consumer_price_index[1],
    # Calculate monthly returns of CD 91.
    CD_91 = cumprod(((CD_91 / 100) + 1) ^ (1/12)),
    Seoul_house_price_index = Seoul_house_price_index / Seoul_house_price_index[1]
  ) %>%
  mutate(
    KOSPI = KOSPI / consumer_price_index - 1,
    CD_91 = CD_91 / consumer_price_index - 1,
    Seoul_house_price_index = Seoul_house_price_index / consumer_price_index - 1
  ) %>%
  select(c(date, KOSPI, CD_91, Seoul_house_price_index))
```

## Without considering inflation
```r
ggplot(consumer_price_not_considered, aes(x = date)) + 
geom_line(aes(y = KOSPI, color='Stock')) + 
geom_line(aes(y = CD_91, color='Saving')) + 
geom_line(aes(y = Seoul_house_price_index, color='House')) + 
geom_line(aes(y = consumer_price_index, color='consumer_price_index')) + 
scale_colour_manual(values=c(Stock='red', Saving='blue', House='green', consumer_price_index='gray')) +
ylab('Returns') + xlab('Time') + ggtitle('Stock vs. House vs. Saving, 2000.1~2018.5') + theme(plot.title=element_text(hjust=0.5))
```
![without_inflation](https://dl.dropboxusercontent.com/s/w6q2bwxjm0zama4/2018-06-07-Investment_considering_inflation-without_inflation.png)
Without considering inflation, saving looks good enough. The return of saving is more than halves of the returns of stock and house.

## Without considering inflation
```r
ggplot(consumer_price_considered, aes(x = date)) + 
geom_line(aes(y = KOSPI, color='Stock')) + 
geom_line(aes(y = CD_91, color='Saving')) + 
geom_line(aes(y = Seoul_house_price_index, color='House')) + 
scale_colour_manual(values=c(Stock='red', Saving='blue', House='green')) +
ylab('Returns') + xlab('Time') + ggtitle('Stock vs. House vs. Saving, 2000.1~2018.5') + theme(plot.title=element_text(hjust=0.5))
```
![with_inflation](https://dl.dropboxusercontent.com/s/glu3f7exb9nfhve/2018-06-07-Investment_considering_inflation-with_inflation.png)
With considering inflation, saving is not attractive anymore. The return of saving is 30% of the return of stock and 40% of the return of house.
Of course, saving is the most stable investment. The final decision is up to you.
I just want to show you the power of inflation in long-term investments.