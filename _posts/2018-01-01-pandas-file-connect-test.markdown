---
layout: post
title:  "Let's test pandas file connecting methods."
subtitle: "excel vs. csv vs. hdf"
date:   2018-01-01 14:37:00
categories: [programming]
---
# CODE
{% highlight python3 %}
import pandas as pd

# 46.8MB as a csv file.
stock_price = pd.read_csv('stock_price.csv', low_memory=False)

# to_excel vs. to_csv vs. to_hdf
writer = pd.ExcelWriter('stock_price.xlsx')
%timeit stock_price.to_excel(writer)
writer.save()
%timeit stock_price.to_csv('stock_price.csv')
%timeit stock_price.to_hdf('stock_price.h5', 'table')

# read_excel vs. read_csv vs. read_hdf
%timeit pd.read_excel('stock_price.xlsx')
%timeit pd.read_csv('stock_price.csv', low_memory=False)
%timeit pd.read_hdf('stock_price.h5', 'table')
{% endhighlight %}

# Result
![Result]({{ "/assets/images/2018-01-01-pandas-file-connect-test.png" | absolute_url }})