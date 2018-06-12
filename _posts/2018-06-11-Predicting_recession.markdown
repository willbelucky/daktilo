---
layout: post
title:  "Predicting recession"
subtitle: "Using market-cap & capital in United State & Korea"
date:   2018-06-11 00:00:00
categories: [experiment]
---
# Can market-cap to asset or market-cap to capital predict recession?

PBR(market-cap to book-value) is a common factor to figure out whether a company is overpriced or not.
We can use same idea in the whole market. 

## Data
1970.1 ~ 2017.12 quarterly market-capital, capital data of United State from WRDS\\
https://www.dropbox.com/s/t8wbz8ioyc30ykv/2018-06-11-Predicting_recession-United_State_companies.csv?dl=0\\
2000.1 ~ 2017.12 inflation data of United State from WRDS\\
https://www.dropbox.com/s/ftmra7mdqgvjy5c/2018-06-11-Predicting_recession-United_State_inflation.csv?dl=0\\
2000.1 ~ 2017.12 quarterly market-capital, capital data of Republic of Korea from DataGuide\\
https://www.dropbox.com/s/2m03457ernb4lax/2018-06-11-Predicting_recession-Korea_companies.csv?dl=0\\
2000.1 ~ 2017.12 monthly inflation data of Republic of Korea from DataGuide\\
https://www.dropbox.com/s/7s9cm8cdgvhdzfs/2018-06-11-Predicting_recession-Korea_inflation.csv?dl=0
```r
rm(list=ls())

######################
## Import Libraries ##
######################

library(tidyverse)
library(timeDate)

###############
## Read Data ##
###############

# United State
united_state_companies <- read.csv("2018-06-11-Predicting_recession-United_State_companies.csv")
united_state_companies$date <- as.Date(as.factor(united_state_companies$date), "%Y-%m-%d")

monthly_united_state <- united_state_companies %>%
  group_by(date) %>%
  summarise(
    capital = sum(capital),
    market_capital = sum(market_capital)
  )

united_state_inflation <- read.csv("2018-06-11-Predicting_recession-United_State_inflation.csv")
united_state_inflation$date <- as.Date(as.factor(united_state_inflation$date), "%Y%m%d")
united_state_inflation$date <- as.Date(timeLastDayInMonth(united_state_inflation$date))
united_state_inflation$inflation <- cumprod(united_state_inflation$inflation + 1)
united_state_inflation$inflation <- united_state_inflation$inflation / united_state_inflation$inflation[1]

monthly_united_state <- monthly_united_state %>%
  inner_join(., united_state_inflation, by = c("date")) %>%
  mutate(
    capital = capital / inflation,
    market_capital = market_capital / inflation
  )

# Republic of Korea
korea_companies <- read.csv("2018-06-11-Predicting_recession-Korea_companies.csv")
korea_companies$date <- as.Date(korea_companies$date, "%Y.%m.%d")
korea_companies <- korea_companies %>%
  drop_na()

monthly_korea <- korea_companies %>%
  group_by(date) %>%
  summarise(
    capital = sum(capital),
    market_capital = sum(market_capital)
  )

korea_inflation <- read.csv("2018-06-11-Predicting_recession-Korea_inflation.csv")
korea_inflation$date <- as.Date(korea_inflation$date, "%Y.%m.%d")
korea_inflation$inflation <- korea_inflation$inflation / korea_inflation$inflation[1]

monthly_korea <- monthly_korea %>%
  inner_join(., korea_inflation, by = c("date")) %>%
  mutate(
    capital = capital / inflation / 1000,
    market_capital = market_capital / inflation
  )
```

## United State(1970.1 ~ 2017.12)
### Historical comparison of capital and market-cap
```r
ggplot(monthly_united_state, aes(x = date)) + 
geom_line(aes(y = capital, color='capital')) + 
geom_line(aes(y = market_capital, color='market_capital')) + 
scale_colour_manual(values=c(capital='red', market_capital='blue')) +
ylab('$1000') + xlab('Time') + ggtitle('United State: capital and market_capital, 1970.1~2017.12') + theme(plot.title=element_text(hjust=0.5))
```
![united_state_comparison](https://dl.dropboxusercontent.com/s/nqhmx0spci31eer/2018-06-11-Predicting_recession-United_State_comparison.png)
In United State, market-cap have more fluctuated than capital.
Capital have grown stably. On the other hand, market-cap have gone up and down repeatedly. 
Sometimes market-cap was far from capital, but it came back to capital.
In this graph, we can assume that the market-cap will come back to capital.
To see the gap between capital and market-cap, I used a linear regression method. 

### Regression of market-cap on capital
```r
capital_on_market_cap <- lm("market_capital ~ capital", data = monthly_united_state)
monthly_united_state$residual = resid(capital_on_market_cap) / monthly_united_state$inflation
summary(capital_on_market_cap)
```
![united_state_regression](https://dl.dropboxusercontent.com/s/vjxvujoa7mu1n8z/2018-06-11-Predicting_recession-United_State_regression.png)
Capital can explain market-cap well in United State. The explanatory power is 0.8738, and it is quite good.

### Residual
```r
ggplot(monthly_united_state, aes(x = date)) + 
geom_line(aes(y = residual, color='residual')) + 
scale_colour_manual(values=c(residual='red', market_capital='blue')) +
ylab('$1000') + xlab('Time') + ggtitle('United State: Resid of reg. of market-cap on capital, 1970.1~2017.12') + theme(plot.title=element_text(hjust=0.5))
```
![united_state_residual](https://dl.dropboxusercontent.com/s/t4t42acu44t8l2n/2018-06-11-Predicting_recession-United_State_residual.png)
The residual shows you how the market-cap is far from the capital.
If the residual is positive, you can see the market is overpriced.
If the residual is negative, the market is underpriced.
In the end of 2017, the market is slightly overpriced.

## Republic of Korea(2000.1 ~ 2017.12)
### Historical comparison of capital and market-cap
```r
ggplot(monthly_korea, aes(x = date)) + 
geom_line(aes(y = capital, color='capital')) + 
geom_line(aes(y = market_capital, color='market_capital')) + 
scale_colour_manual(values=c(capital='red', market_capital='blue')) +
ylab('₩1000') + xlab('Time') + ggtitle('United State: Capital and market_capital, 1970.1~2017.12') + theme(plot.title=element_text(hjust=0.5))
```
![korea_comparison](https://dl.dropboxusercontent.com/s/ra4255wm4o1hxwy/2018-06-11-Predicting_recession-Korea_comparison.png)
Korea has same tendency with United State.
The market was overpriced in the early 2000(Dot-com bubble) and 2008(2008 Financial Crisis).
And the market-cap came back to capital every time.

### Regression of market-cap on capital
```r
capital_on_market_cap <- lm("market_capital ~ capital", data = monthly_korea)
monthly_korea$residual = resid(capital_on_market_cap) / monthly_korea$inflation
summary(capital_on_market_cap)
```
![korea_regression](https://dl.dropboxusercontent.com/s/f2hn61riulpdyyn/2018-06-11-Predicting_recession-Korea_regression.png)
Capital can explain market-cap well in Korea too. The explanatory power is 0.9009.

### Residual
```r
ggplot(monthly_korea, aes(x = date)) + 
geom_line(aes(y = residual, color='residual')) + 
scale_colour_manual(values=c(residual='red')) +
ylab('₩1000') + xlab('Time') + ggtitle('Korea: Resid of reg. of market-cap on capital, 2000.1~2017.12') + theme(plot.title=element_text(hjust=0.5))
```
![korea_residual](https://dl.dropboxusercontent.com/s/7o7vnswb7owlrad/2018-06-11-Predicting_recession-Korea_residual.png)
In the end of 2017, the market of Korea is overpriced.
I do not recommend you to start stock investment now.
Maybe soon, the market-cap will come back to capital and it is also possible that market-cap goes down to capital.
I think that moment is the perfect time to start stock investment.
