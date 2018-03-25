---
layout: post
title:  "Basic linear regression interpretation"
subtitle: "How to understand the result of regression"
date:   2018-01-30 13:47:00
categories: [statistics]
---
# Data source
## AustinApartmentRent
### from Thomas Sager at University of Texas at Austin
https://www.dropbox.com/s/g04jhka2u2ubrnm/AustinApartmentRent.xls?dl=0

# SAS regression code
{% highlight sas %}
libname BA "/home/jaekyoungkim0/BA";
PROC IMPORT DATAFILE="/home/jaekyoungkim0/BA/AustinApartmentRent.xls"
		    OUT=BA.apts
		    DBMS=XLS
		    REPLACE;
RUN;

* Get summary statistics on RENT in WORK.APTS;
proc means data=apts;
	var rent;
RUN;

* Test normality of RENT in BA.APTS;
proc univariate data=BA.apts normal;
	var rent;
RUN;

* Regress RENT (Y) on AREA (X) in WORK.APTS;
proc reg data=WORK.apts;
	model rent = area;
RUN;

* Regress RENT (Y) on AREA (X) in BA.APTS, add predicted and residual values to a WORK output dataset,
and test normality of the residuals;
proc reg data=BA.apts;
	model rent = area;
	output out=apts2 p=pRent r=residRent;
RUN;

proc univariate data=apts2 normal;
	var residRent;
RUN;

* To get the exact standard error for estimate of rent and mean rent of 1000 sq ft apt,
add a 61st apt with missing value for its rent, and area = 1000;
data BA.apts2; set BA.apts;
	/* Bring in data lines from BA.APTS */
	output;
	* Process the following code when reach the 60th observation;
	if _n_ = 60 then do; area=1000; bathrooms=2; rent = .;
		/* period is the missing value code */
		output;
	end;
RUN;

* Regress RENT (Y) on AREA (X) in BA.APTS2, calculate plug-in estimates of Y
and mean Y for every AREA, and print stdevs for mean Y and confidence intervals for individual
and mean Y for every area;
* The 61st apt is not used in the regression, but confidence intervals are calculated;
* CLI=Confidence Limits for Individual apts; * CLM=Confidence Limits for Means of apts;
* Default confidence percentage is 95%;
proc reg data=BA.apts2;
	model rent = area / CLI CLM;
RUN;
{% endhighlight %}

# Regression result
![SimpleLinearRegression](https://dl.dropboxusercontent.com/s/d7qg0uksb56euez/2018-01-30-Basic_regression_interpretation-simple_linear_regression.png)

## Interpretation
- The number of sample
  - N = 60
- The rent of an Austin apartment that will be randomly selected from the population of all Austin apartments
  - mean(평균) = $572.27
- An expected margin of error
  - standard_deviation(표준편차) = $140.52, but we cannot quantify the % confidence. The Central Limit Theorem (CLT) does not apply to individuals drawn from a distribution.
- The mean rent of all Austin apartments
  - mean = $572.27
- An expected margin of error
  - standard_deviation / sqrt(N) = $140.52 / sqrt(60) = $18.14, and the probability that the sample mean is found within one standard error of the true mean is about 68%(by CLT).
- The meaning of intercept_parameter_estimate
  - The average fixed cost
- An expected margin of error
  - intercept_standard_error = 31.36081, and the probability that the sample mean is found within one standard error of the true mean is about 68%(by CLT).
- The meaning of area_parameter_estimate
  - The worth of additional 1 square foot.
- An expected margin of error
  - area_standard_error = 0.03685, and the probability that the sample mean is found within one standard error of the true mean is about 68%(by CLT).
- The rent of an Austin apartment that will be randomly selected from the population of all Austin apartments that have 1000 square feet of area
  - intercept_parameter_estimate + area_parameter_estimate * area = $160.19 + $0.505 * 1000 = $665.19
- An expected margin of error
  - root_MSE = $68.86, but we cannot quantify the % confidence. The Central Limit Theorem (CLT) does not apply to individuals drawn from a distribution.
- The mean rent of all Austin apartments that have 1000 square feet of area
  - intercept_parameter_estimate + area_parameter_estimate * area = $160.19 + $0.505 * 1000 = $665.19
- An expected margin of error
  - root_MSE / sqrt(N) = $68.86 / sqrt(60) = 8.89, and the probability that the sample mean is found within one standard error of the true mean is about 68%(by CLT).
- The proportion of variation of rent that can be attributed to the effect of area
  - R-Square = 0.7640