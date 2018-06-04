---
layout: post
title:  "Fleet sales pricing at Fjord Motor"
subtitle: "Robert Phillips"
date:   2018-05-24 00:00:00
categories: [paper_book]
---

# Case source
case centre
https://www.thecasecentre.org/educators/products/view?&id=132712


```python
import pandas as pd
import numpy as np
from scipy.optimize import minimize

UNIT_NUMBER = 'unit_number'
UNIT_PRICE = 'unit_price'
TOTAL_PRICE = 'total_price'
WIN = 'win'
DISCOUNT_RATE = 'discount_rate'
UNIT_MARGIN = 'unit_margin'
UNIT_SOLD_NUMBER = 'unit_sold_number'
TOTAL_MARGIN = 'total_margin'
POLICE = 'police'
P = 'P'
```

# Data reading


```python
# Set the PYTHONPATH as root folder, like PYTHONPATH=/Users/willbe/PycharmProjects/FleetSalesPricingAtFjordMotor
DATA_DIR = 'data/fleet_sales_pricing_at_fjord_motor.csv'


def get_all_data(MSRP):
    """
    Get data of all bids.

    :return data: (DataFrame) 8 columns * 4000 rows
        columns unit_number         (int) The number of vehicles.
                unit_price          (int) The price of a vehicle.
                total_price         (int) unit_number * unit_price
                win                 (int) If Fjord win a fleet bid, win is 1. Else, win is 0.
                discount_rate       (float) ( MSRP(the_manufacturer's_suggested_retail_price) - unit_price ) / MSRP
                unit_margin         (int) MSRP - cost
                unit_sold_number    (int) unit_number * win
                total_margin        (int) unit_margin * unit_sold_number
                p                   (float) unit_price / MSRP.
    """
    data = pd.read_csv(DATA_DIR)
    data[P] = data[UNIT_PRICE] / MSRP
    return data


def get_police_data(MSRP):
    """
    The bids 1 through 2000 were to police departments.

    :return data: (DataFrame) 8 columns * 2000 rows
        columns unit_number         (int) The number of vehicles.
                unit_price          (int) The price of a vehicle.
                total_price         (int) unit_number * unit_price
                win                 (int) If Fjord win a fleet bid, win is 1. Else, win is 0.
                discount_rate       (float) ( MSRP(the_manufacturer's_suggested_retail_price) - unit_price ) / MSRP
                unit_margin         (int) MSRP - cost
                unit_sold_number    (int) unit_number * win
                total_margin        (int) unit_margin * unit_sold_number
                p                   (float) unit_price / MSRP.
    """
    data = get_all_data(MSRP)
    data = data.iloc[:2000, :]
    return data


def get_corporate_buyer_data(MSRP):
    """
    The bids 2001 through 4000 were to corporate buyers.

    :return data: (DataFrame) 8 columns * 2000 rows
        columns unit_number         (int) The number of vehicles.
                unit_price          (int) The price of a vehicle.
                total_price         (int) unit_number * unit_price
                win                 (int) If Fjord win a fleet bid, win is 1. Else, win is 0.
                discount_rate       (float) ( MSRP(the_manufacturer's_suggested_retail_price) - unit_price ) / MSRP
                unit_margin         (int) MSRP - cost
                unit_sold_number    (int) unit_number * win
                total_margin        (int) unit_margin * unit_sold_number
                p                   (float) unit_price / MSRP.
    """
    data = get_all_data(MSRP)
    data = data.iloc[2000:, :]
    return data
```

# Assignment 1

$$\rho$$ is the probability of winning the bid.\\
$$a$$ and $$b$$ are the parameters to be estimated.\\
$$p$$ is the price that Fjord bid on a deal expressed as a fraction of the MSRP of $25,000 per units; that is, if Fjord id $20,000 per unit, p would e equal to 20,000/25,000 or 0.8.\\
A model is $$\rho(p) = \frac{1}{1 + exp(a + bp)}$$.

## What are the values of a and b that maximize the sum of log likelihoods?

1) Calculate winning probabilities using the model.


```python
def calculate_winning_prob(a, b, p):
    """
    :param a: (float) An intercept of the model.
    :param b: (float) A beta of the model.
    :param p: (pd.Series[float]) unit_price / MSRP.
    
    :return winning_prob: (pd.Series[float]) The expected winning probabilities of the model.
    """
    winning_prob = 1 / (1 + np.exp(a + b * p))
    return winning_prob
```

2) Make a function for calculating a sum of log likelihoods.

log_likelihood = $$log(\rho^y (1-\rho)^{1-y})$$


```python
def calculate_sum_of_log_likelihood(a, b, args):
    """
    :param a: (float) An intercept of the model.
    :param b: (float) A beta of the model.
    :param args: (tuple)
        p           (pd.Series[float]) unit_price / MSRP.
        observed_y  (pd.Series[int]) If Fjord win a fleet bid, win is 1. Else, win is 0.
        
    :return log_likelihood_sum: (float) The sum of log likelihood.
    """
    p, observed_y = args
    winning_prob = calculate_winning_prob(a, b, p)
    log_likelihood_sum = np.sum(np.log(np.power(winning_prob, observed_y) * np.power(1-winning_prob, 1-observed_y)))
    return log_likelihood_sum
```

3) Becasue numpy has only a minize optimizer, make a negative function of the function for calculating a sum of log likelihoods.


```python
def calculate_negative_sum_of_log_likelihood(param, *args):
    a, b = param
    negative_log_likelihood_sum = -calculate_sum_of_log_likelihood(a, b, args=args)
    return negative_log_likelihood_sum
```

4) Maximize the sum of log likelihood by minimizing the negative sum of log likelihoods.


```python
def maximize_log_likelihood(args):
    """
    : return optimal_a: (float) The optimal intercept of the model.
    : return optimal_b: (float) The optimal beta of the model.
    """
    # Make a list of initial parameter guesses (intercept, beta_1)
    initial_guess = np.array([0, 0])

    # Minimize a negative log likelihood for maximizing a log likelihood.
    results = minimize(calculate_negative_sum_of_log_likelihood, x0=initial_guess, args=args)

    optimal_a, optimal_b = results.x
    return optimal_a, optimal_b
```

5) Calculate the values of a and b.


```python
MSRP = 25000
all_data = get_all_data(MSRP)
args = (all_data[P], all_data[WIN])
optimal_a, optimal_b = maximize_log_likelihood(args)
print("a: {:.2f}, b: {:.2f}".format(optimal_a, optimal_b))
```

**a: -7.76, b: 9.16**


## What is the optimum price Fjord should offer, assuming it is going to offer a single price for each bid?

Based on a and b we calcuated, we need to find a single price maximizing sum of margin. For that,

1) Calculate the expected margin.

cost = 15000\\
expected_margin = (single_price - cost) * $$\rho(p)$$


```python
COST = 15000
def calculate_expected_margin(single_price, args):
    """
    :param single_price: (float)
    
    :return expected_margin: (float) The expected profits.
    """
    a, b = args
    p = single_price / MSRP
    winning_prob = calculate_winning_prob(a, b, p)
    expected_margin = (single_price - COST) * winning_prob
    return expected_margin
```

2) Make a negative function of the function for calculating the expected margin.


```python
def calculate_negative_expected_margin(param, *args):
    single_price = param
    negative_expected_margin = -calculate_expected_margin(single_price, args)
    return negative_expected_margin
```

3) Maximize the expected margin by minimizing the negative expected margin.


```python
def maxize_expected_margin(args):
    # Make a list of initial parameter guesses (single_price)
    initial_guess = np.array([0])

    # Minimize a negative log likelihood for maximizing a log likelihood.
    results = minimize(calculate_negative_expected_margin, x0=initial_guess, args=args)

    optimal_single_price = results.x[0]
    return optimal_single_price
```

4) Calculate the optimal single price maximizing margin.


```python
args = (optimal_a, optimal_b)
optimal_single_price = maxize_expected_margin(args)
print("optimal single price: ${:.2f}".format(optimal_single_price))
```

**optimal single price: $20818.70**


## What would the expected total contribution have been for the 4,000 bids?

ExpectedTotalContribution = $\sum_{i=1}^{4000}{[UnitNumber_i * (SinglePrice - Cost)]}$


```python
def calculate_expected_total_contribution(unit_numbers, optimal_price, cost, a, b):
    """
    :param unit_numbers: (pd.Series[int]) The numbers of unit of bids.
    :param optimal_price: (float) The optimal price for bids.
    :param cost: (float)
    :param a: (float)
    :param b: (float)
    
    :return expected_total_contribution: The sum of expected contributions(unit_number * expected_margin).
    """
    p = optimal_price / MSRP
    expected_total_contribution = np.sum(unit_numbers * (optimal_price - cost) * calculate_winning_prob(a, b, p))
    return expected_total_contribution
```


```python
expected_total_contribution = calculate_expected_total_contribution(all_data[UNIT_NUMBER], optimal_single_price, COST, optimal_a, optimal_b)
print("The expected total contribution: ${:.2f}".format(expected_total_contribution))
```

**The expected total contribution: $241083842.21**


## How does this compare to the contribution that Fjord actually received?

ActualTotalContribution = $\sum_{i=1}^{4000}{[Win_i * UnitNumber_i * (UnitPrice_i - Cost)]}$


```python
def calculate_actual_total_contribution(wins, unit_numbers, unit_prices):
    actual_total_contribution = np.sum(wins * unit_numbers * (unit_prices - COST))
    return actual_total_contribution
```


```python
actual_total_contribution = calculate_actual_total_contribution(all_data[WIN], all_data[UNIT_NUMBER], all_data[UNIT_PRICE])
print("The actual total contribution: ${}".format(actual_total_contribution))
print("Improvement: ${:.2f}, {:.2f}%".format(
    expected_total_contribution - actual_total_contribution,
    (expected_total_contribution / actual_total_contribution - 1) * 100
))
```

**The actual total contribution: $171829002**\\
**Improvement: $69254840.21, 40.30%**


# Assignment 2

Fjord discovers that bids 1 through 2,000 were to police departments, and the bids 2,001 through 4,000 were to corporate buyers.

## What are the corresponding values of a and b for each?

1) Calculate the values of a and b for police.


```python
police_data = get_police_data(MSRP)
args = (police_data[P], police_data[WIN])
police_optimal_a, police_optimal_b = maximize_log_likelihood(args)
print("Police a: {:.2f}, b: {:.2f}".format(police_optimal_a, police_optimal_b))
```

**Police a: -14.22, b: 20.01**


2) Calculate the values of a and b for corporate buyer.


```python
corporate_buyer_data = get_corporate_buyer_data(MSRP)
args = (corporate_buyer_data[P], corporate_buyer_data[WIN])
corporate_buyer_optimal_a, corporate_buyer_optimal_b = maximize_log_likelihood(args)
print("Corporate buyer a: {:.2f}, b: {:.2f}".format(corporate_buyer_optimal_a, corporate_buyer_optimal_b))
```

**Corporate buyer a: -27.88, b: 28.81**


## What are the optimum price Fjord should offer to the police?


```python
args = (police_optimal_a, police_optimal_b)
police_optimal_single_price = maxize_expected_margin(args)
print("Police optimal price: ${:.2f}".format(police_optimal_single_price))
```

**Police optimal price: $17638.54**


## To corporate buyers?


```python
args = (corporate_buyer_optimal_a, corporate_buyer_optimal_b)
corporate_buyer_optimal_single_price = maxize_expected_margin(args)
print("Corporate buyer optimal price: ${:.2f}".format(corporate_buyer_optimal_single_price))
```

**Corporate buyer optimal price: $22431.46**


## What would the expected contribution have been if Fjord had used the price in the 4,000 bids in the database?


```python
police_expected_total_contribution = calculate_expected_total_contribution(
    police_data[UNIT_NUMBER],
    police_optimal_single_price,
    COST,
    police_optimal_a,
    police_optimal_b
)
expected_total_contribution = calculate_expected_total_contribution(
    corporate_buyer_data[UNIT_NUMBER],
    corporate_buyer_optimal_single_price,
    COST,
    corporate_buyer_optimal_a,
    corporate_buyer_optimal_b
)
new_expected_total_contribution = police_expected_total_contribution + expected_total_contribution
print("The expected total contribution: ${:.2f}".format(new_expected_total_contribution))
```

**The expected total contribution: $308695819.88**


## What is the difference between the contribution actually received and the best that Fjord could do when it could not differentiate between the police and corporate buyers?


```python
print("Improvement from the contribution actually received: ${:.2f}, {:.2f}%".format(
    new_expected_total_contribution - actual_total_contribution,
    (new_expected_total_contribution / actual_total_contribution - 1) * 100
))
print("Improvement from the best that Fjord could do when it could not differentiate between the police and corporate buyers: ${:.2f}, {:.2f}%".format(
    new_expected_total_contribution - expected_total_contribution,
    (new_expected_total_contribution / expected_total_contribution - 1) * 100
))
```

**Improvement from the contribution actually received: $136866817.88, 79.65%**\\
**Improvement from the best that Fjord could do when it could not differentiate between the police and corporate buyers: $54579338.40, 21.48%**


# Assignment 3

$$c$$ is the size of the order.

The new model: $$ \rho(p) = 1 / (1 + exp(a + bp + cs))$$

## What is the resulting improvement in total log likelihood?

1) Calculate winning probabilities using the model.


```python
def calculate_2_factor_winning_prob(a, b, c, p, unit_number):
    """
    :param a: (float) An intercept of the model.
    :param b: (float) A coefficient for p of the bid.
    :param c: (float) A coefficient for the size of the bid.
    :param p: (pd.Series[float]) unit_price / MSRP.
    
    :return winning_prob: (pd.Series[float]) The expected winning probabilities of the model.
    """
    winning_prob = 1 / (1 + np.exp(a + b * p + c * unit_number))
    return winning_prob
```

2) Make a function for calculating a sum of log likelihoods.

log_likelihood = $$log(\rho^y (1-\rho)^{1-y})$$


```python
def calculate_2_factor_sum_of_log_likelihood(a, b, c, args):
    """
    :param a: (float) An intercept of the model.
    :param b: (float) A coefficient for p of the bid.
    :param c: (float) A coefficient for the size of the bid.
    :param args: (tuple)
        p           (pd.Series[float]) unit_price / MSRP.
        unit_number (pd.Series[int]) The size of the bid.
        observed_y  (pd.Series[int]) If Fjord win a fleet bid, win is 1. Else, win is 0.
        
    :return log_likelihood_sum: (float) The sum of log likelihood.
    """
    p, unit_number, observed_y = args
    winning_prob = calculate_2_factor_winning_prob(a, b, c, p, unit_number)
    log_likelihood_sum = np.sum(np.log(np.power(winning_prob, observed_y) * np.power(1-winning_prob, 1-observed_y)))
    return log_likelihood_sum
```

3) Becasue numpy has only a minize optimizer, make a negative function of the function for calculating a sum of log likelihoods.


```python
def calculate_2_factor_negative_sum_of_log_likelihood(param, *args):
    a, b, c = param
    negative_log_likelihood_sum = -calculate_2_factor_sum_of_log_likelihood(a, b, c, args=args)
    return negative_log_likelihood_sum
```

4) Maximize the sum of log likelihood by minimizing the negative sum of log likelihoods.


```python
def maximize_2_factor_log_likelihood(args):
    """
    :return optimal_a: (float) The optimal intercept of the model.
    :return optimal_b: (float) The optimal coefficient for p.
    :return optimal_c: (float) The optimal coefficient for the size of the bid.
    """
    # Make a list of initial parameter guesses (intercept, beta_1)
    initial_guess = np.array([0, 0, 0])

    # Minimize a negative log likelihood for maximizing a log likelihood.
    results = minimize(calculate_2_factor_negative_sum_of_log_likelihood, x0=initial_guess, args=args)

    optimal_a, optimal_b, optimal_c = results.x
    return optimal_a, optimal_b, optimal_c
```

5) Calculate the values of a and b.


```python
# Previous model
args = (all_data[P], all_data[WIN])
previous_log_likelihood_sum = calculate_sum_of_log_likelihood(optimal_a, optimal_b, args)

# New model
# Police
args = (police_data[P], police_data[UNIT_NUMBER], police_data[WIN])
police_2_factor_optimal_a, police_2_factor_optimal_b, police_2_factor_optimal_c = maximize_2_factor_log_likelihood(args)
new_police_log_likelihood_sum = calculate_2_factor_sum_of_log_likelihood(police_2_factor_optimal_a, police_2_factor_optimal_b, police_2_factor_optimal_c, args)

# Corporate buyer
args = (corporate_buyer_data[P], corporate_buyer_data[UNIT_NUMBER], corporate_buyer_data[WIN])
corporate_buyer_2_factor_optimal_a, corporate_buyer_2_factor_optimal_b, corporate_buyer_2_factor_optimal_c = maximize_2_factor_log_likelihood(args)
new_corporate_buyer_log_likelihood_sum = calculate_2_factor_sum_of_log_likelihood(corporate_buyer_2_factor_optimal_a, corporate_buyer_2_factor_optimal_b, corporate_buyer_2_factor_optimal_c, args)

# Total
new_total_log_likelihood_sum = new_police_log_likelihood_sum + new_corporate_buyer_log_likelihood_sum

# Comparison
print("Improvement: ${:.2f}, {:.2f}%".format(
    new_total_log_likelihood_sum - previous_log_likelihood_sum,
    np.abs((new_total_log_likelihood_sum / previous_log_likelihood_sum - 1) * 100)
))
```

**Improvement: $1241.95, 54.91%**


## How does this compare with the improvement from differentiating police and corporate sales?


```python
# Previous model
# Police
args = (police_data[P], police_data[WIN])
previous_police_log_likelihood_sum = calculate_sum_of_log_likelihood(police_optimal_a, police_optimal_b, args)

# Corporate buyer
args = (corporate_buyer_data[P], corporate_buyer_data[WIN])
previous_corporate_buyer_log_likelihood_sum = calculate_sum_of_log_likelihood(corporate_buyer_optimal_a, corporate_buyer_optimal_b, args)

# Total
previous_total_log_likelihood_sum = previous_police_log_likelihood_sum + previous_corporate_buyer_log_likelihood_sum

# Comparison
print("Improvement: ${:.2f}, {:.2f}%".format(
    new_total_log_likelihood_sum - previous_total_log_likelihood_sum,
    np.abs((new_total_log_likelihood_sum / previous_total_log_likelihood_sum - 1) * 100)
))
```

**Improvement: $10.27, 1.00%**


## What are the optimal prices Fjord should charge for orders of 20 cars and for orders of 40 cars to police departments and to corporate purchasers, respectively?

Based on a and b we calcuated, we need to find a single price maximizing sum of margin. For that,

1) Calculate the expected margin.

cost = 15000\\
expected_margin = (single_price - cost) * $$\rho(p)$$


```python
def calculate_2_factor_expected_margin(single_price, args):
    """
    :param single_price: (float)
    
    :return expected_margin: (float) The expected profits.
    """
    a, b, c, cost, unit_number = args
    p = single_price / MSRP
    winning_prob = calculate_2_factor_winning_prob(a, b, c, p, unit_number)
    expected_margin = (single_price - cost) * winning_prob
    return expected_margin
```

2) Make a negative function of the function for calculating the expected margin.


```python
def calculate_2_factor_negative_expected_margin(param, *args):
    single_price = param
    negative_expected_margin = -calculate_2_factor_expected_margin(single_price, args)
    return negative_expected_margin
```

3) Maximize the expected margin by minimizing the negative expected margin.


```python
def maximize_2_factor_expected_margin(args):
    # Make a list of initial parameter guesses (single_price)
    initial_guess = np.array([0])

    # Minimize a negative log likelihood for maximizing a log likelihood.
    results = minimize(calculate_2_factor_negative_expected_margin, x0=initial_guess, args=args)

    optimal_single_price = results.x[0]
    return optimal_single_price
```


```python
police_20_cars_args = (police_2_factor_optimal_a, police_2_factor_optimal_b, police_2_factor_optimal_c, COST, 20)
police_20_cars_optimal_price = maximize_2_factor_expected_margin(police_20_cars_args)
print("The optimal price of 20 cars to police departments: ${:.2f}".format(police_20_cars_optimal_price))

police_40_cars_args = (police_2_factor_optimal_a, police_2_factor_optimal_b, police_2_factor_optimal_c, COST, 40)
police_40_cars_optimal_price = maximize_2_factor_expected_margin(police_40_cars_args)
print("The optimal price of 40 cars to police departments: ${:.2f}".format(police_40_cars_optimal_price))

corporate_buyer_20_cars_args = (corporate_buyer_2_factor_optimal_a, corporate_buyer_2_factor_optimal_b, corporate_buyer_2_factor_optimal_c, COST, 20)
corporate_buyer_20_cars_optimal_price = maximize_2_factor_expected_margin(corporate_buyer_20_cars_args)
print("The optimal price of 20 cars to corporate purchasers: ${:.2f}".format(corporate_buyer_20_cars_optimal_price))

corporate_buyer_40_cars_args = (corporate_buyer_2_factor_optimal_a, corporate_buyer_2_factor_optimal_b, corporate_buyer_2_factor_optimal_c, COST, 40)
corporate_buyer_40_cars_optimal_price = maximize_2_factor_expected_margin(corporate_buyer_40_cars_args)
print("The optimal price of 40 cars to corporate purchasers: ${:.2f}".format(corporate_buyer_40_cars_optimal_price))
```

**The optimal price of 20 cars to police departments: $17616.11**\\
**The optimal price of 40 cars to police departments: $17214.42**\\
**The optimal price of 20 cars to corporate purchasers: $22441.15**\\
**The optimal price of 40 cars to corporate purchasers: $22525.99**


## Calculate optimal price for all order sizes from 10 through 60 vehicles for both police and corporate sales, and use these price to determine the total contribution margin Fjord would have received if it had used these prices in the 4,000 historic bids.


```python
police_optimal_prices = []
corporate_buyer_optimal_prices = []
for i in range(10, 61):
    # Police
    police_args = (police_2_factor_optimal_a, police_2_factor_optimal_b, police_2_factor_optimal_c, COST, i)
    police_optimal_price = maximize_2_factor_expected_margin(police_args)
    police_optimal_prices.append(police_optimal_price)
    
    # Corporate buyer
    corporate_buyer_args = (corporate_buyer_2_factor_optimal_a, corporate_buyer_2_factor_optimal_b, corporate_buyer_2_factor_optimal_c, COST, i)
    corporate_buyer_optimal_price = maximize_2_factor_expected_margin(corporate_buyer_args)
    corporate_buyer_optimal_prices.append(corporate_buyer_optimal_price)
```


```python
pd.DataFrame(data={'police': police_optimal_prices,
                   'corporate_buyer': corporate_buyer_optimal_prices},
             index=range(10, 61))
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>corporate_buyer</th>
      <th>police</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>10</th>
      <td>22398.832514</td>
      <td>17841.998337</td>
    </tr>
    <tr>
      <th>11</th>
      <td>22403.065253</td>
      <td>17818.654108</td>
    </tr>
    <tr>
      <th>12</th>
      <td>22407.282696</td>
      <td>17795.529174</td>
    </tr>
    <tr>
      <th>13</th>
      <td>22411.557702</td>
      <td>17772.606965</td>
    </tr>
    <tr>
      <th>14</th>
      <td>22415.723659</td>
      <td>17749.799144</td>
    </tr>
    <tr>
      <th>15</th>
      <td>22420.026251</td>
      <td>17727.132447</td>
    </tr>
    <tr>
      <th>16</th>
      <td>22424.271564</td>
      <td>17704.640133</td>
    </tr>
    <tr>
      <th>17</th>
      <td>22428.440539</td>
      <td>17682.229293</td>
    </tr>
    <tr>
      <th>18</th>
      <td>22432.690807</td>
      <td>17660.083406</td>
    </tr>
    <tr>
      <th>19</th>
      <td>22436.936602</td>
      <td>17638.008948</td>
    </tr>
    <tr>
      <th>20</th>
      <td>22441.151170</td>
      <td>17616.109653</td>
    </tr>
    <tr>
      <th>21</th>
      <td>22445.404314</td>
      <td>17594.442419</td>
    </tr>
    <tr>
      <th>22</th>
      <td>22449.643958</td>
      <td>17573.086999</td>
    </tr>
    <tr>
      <th>23</th>
      <td>22453.813047</td>
      <td>17551.673509</td>
    </tr>
    <tr>
      <th>24</th>
      <td>22458.074484</td>
      <td>17530.451576</td>
    </tr>
    <tr>
      <th>25</th>
      <td>22462.308636</td>
      <td>17509.227803</td>
    </tr>
    <tr>
      <th>26</th>
      <td>22466.543872</td>
      <td>17488.344587</td>
    </tr>
    <tr>
      <th>27</th>
      <td>22470.840222</td>
      <td>17467.668814</td>
    </tr>
    <tr>
      <th>28</th>
      <td>22475.101310</td>
      <td>17447.070235</td>
    </tr>
    <tr>
      <th>29</th>
      <td>22479.403692</td>
      <td>17426.694747</td>
    </tr>
    <tr>
      <th>30</th>
      <td>22483.609712</td>
      <td>17406.599145</td>
    </tr>
    <tr>
      <th>31</th>
      <td>22487.685601</td>
      <td>17386.574284</td>
    </tr>
    <tr>
      <th>32</th>
      <td>22492.068984</td>
      <td>17366.668712</td>
    </tr>
    <tr>
      <th>33</th>
      <td>22496.329942</td>
      <td>17347.109827</td>
    </tr>
    <tr>
      <th>34</th>
      <td>22500.563714</td>
      <td>17327.532611</td>
    </tr>
    <tr>
      <th>35</th>
      <td>22504.702582</td>
      <td>17308.250370</td>
    </tr>
    <tr>
      <th>36</th>
      <td>22509.019517</td>
      <td>17289.117823</td>
    </tr>
    <tr>
      <th>37</th>
      <td>22513.264192</td>
      <td>17270.029364</td>
    </tr>
    <tr>
      <th>38</th>
      <td>22517.513312</td>
      <td>17251.404789</td>
    </tr>
    <tr>
      <th>39</th>
      <td>22521.768850</td>
      <td>17232.877346</td>
    </tr>
    <tr>
      <th>40</th>
      <td>22525.991300</td>
      <td>17214.421578</td>
    </tr>
    <tr>
      <th>41</th>
      <td>22530.246494</td>
      <td>17196.339595</td>
    </tr>
    <tr>
      <th>42</th>
      <td>22534.506684</td>
      <td>17178.313565</td>
    </tr>
    <tr>
      <th>43</th>
      <td>22538.671058</td>
      <td>17160.555318</td>
    </tr>
    <tr>
      <th>44</th>
      <td>22543.002327</td>
      <td>17143.082284</td>
    </tr>
    <tr>
      <th>45</th>
      <td>22547.430786</td>
      <td>17125.527459</td>
    </tr>
    <tr>
      <th>46</th>
      <td>22551.519997</td>
      <td>17108.305135</td>
    </tr>
    <tr>
      <th>47</th>
      <td>22555.756078</td>
      <td>17091.336017</td>
    </tr>
    <tr>
      <th>48</th>
      <td>22560.008469</td>
      <td>17074.435613</td>
    </tr>
    <tr>
      <th>49</th>
      <td>22564.220936</td>
      <td>17057.839270</td>
    </tr>
    <tr>
      <th>50</th>
      <td>22568.482607</td>
      <td>17041.357755</td>
    </tr>
    <tr>
      <th>51</th>
      <td>22572.742502</td>
      <td>17025.194591</td>
    </tr>
    <tr>
      <th>52</th>
      <td>22576.952570</td>
      <td>17009.318247</td>
    </tr>
    <tr>
      <th>53</th>
      <td>22581.221811</td>
      <td>16993.411402</td>
    </tr>
    <tr>
      <th>54</th>
      <td>22585.494939</td>
      <td>16977.662457</td>
    </tr>
    <tr>
      <th>55</th>
      <td>22589.728859</td>
      <td>16962.222977</td>
    </tr>
    <tr>
      <th>56</th>
      <td>22593.887900</td>
      <td>16947.013351</td>
    </tr>
    <tr>
      <th>57</th>
      <td>22598.179947</td>
      <td>16931.969925</td>
    </tr>
    <tr>
      <th>58</th>
      <td>22602.505608</td>
      <td>16917.239199</td>
    </tr>
    <tr>
      <th>59</th>
      <td>22606.719729</td>
      <td>16902.577904</td>
    </tr>
    <tr>
      <th>60</th>
      <td>22610.975364</td>
      <td>16888.111587</td>
    </tr>
  </tbody>
</table>
</div>



# Assignment 4

The police version of the Cornet Elizabeth costs $16,000 to manufacture versus $15,000 for the corporate version.

## How would this change the optimal price charged to police departments for 20 vehicles? For 40?


```python
NEW_COST = 16000

police_20_cars_args = (police_2_factor_optimal_a, police_2_factor_optimal_b, police_2_factor_optimal_c, NEW_COST, 20)
police_20_cars_optimal_price = maximize_2_factor_expected_margin(police_20_cars_args)
print("The optimal price of 20 cars to police departments: ${:.2f}".format(police_20_cars_optimal_price))

police_40_cars_args = (police_2_factor_optimal_a, police_2_factor_optimal_b, police_2_factor_optimal_c, NEW_COST, 40)
police_40_cars_optimal_price = maximize_2_factor_expected_margin(police_40_cars_args)
print("The optimal price of 40 cars to police departments: ${:.2f}".format(police_40_cars_optimal_price))
```

**The optimal price of 20 cars to police departments: $18138.28**\\
**The optimal price of 40 cars to police departments: $17828.91**
