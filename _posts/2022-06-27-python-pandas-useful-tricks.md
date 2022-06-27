---
layout: post
title:  "Python pandas useful tricks (updated continuously)"
date:   2022-06-27 05:46:00
subtitle: 
img:
tags: [python,pandas,dataframe] # add tag
published: true
---

### Flattening a dataframe after doing multiple aggregation on multiple columns

```python
import pandas as pd
import numpy as np

# Create an example dataframe
df = pd.DataFrame({"User": ["user1", "user2", "user2", "user3", "user2", "user1"],
                  "Amount": [10.0, 5.0, 8.0, 10.5, 7.5, 8.0],
                  "Value":[5,4,6,4,5,6]})

# First create the multiple aggregation on multiple column            
df_agg = df.groupby("User").agg({"Amount":[("total_amount","sum"),\
            ("average_amount","mean")],\
            "Value":[("total_value","sum"),\
                      ('value_std', lambda x: np.std(x))]})
df_agg
```

This is how the `df_agg` looks like:
```
            Amount                      Value          
      total_amount average_amount total_value value_std
User                                                   
user1         18.0       9.000000          11  0.500000
user2         20.5       6.833333          15  0.816497
user3         10.5      10.500000           4  0.000000
```
The `df_agg` dataframe is a grouped or aggregated dataframe and indexed by the column(s) it is grouped by. Before calling any `reset_index`, lets see how the `columns` attribute of the grouped dataframe looks like:
```python
df_agg.columns
```
```
MultiIndex([('Amount',   'total_amount'),
            ('Amount', 'average_amount'),
            ( 'Value',    'total_value'),
            ( 'Value',      'value_std')],
           )
```
You can see from the list above that the `User` is missing in the above list of columns. This is because it is not a column yet, it is an index. If we reset the index of this dataframe then we will find `User` as part of the columns.
```python
# We want to first reset the index
df_agg = df_agg.reset_index()
df_agg.columns
```
```
MultiIndex([(  'User',               ''),
            ('Amount',   'total_amount'),
            ('Amount', 'average_amount'),
            ( 'Value',    'total_value'),
            ( 'Value',      'value_std')],
           )
```
Now, we have already dropped the index, only left with hierarchical columns. Now we will get rid of these hierarchical columns with an easy trick. In the list above, the column names which are part of the grouping, have an empty string in the second element of the tuple. We will use this information to create list of column names, where the grouper columns come from the first element of the tuples and the aggregated columns names come from the second element of the tuple.
```python
df_agg.columns = [f"{a}" if b=="" else f"{b}" for a,b in df_agg.columns]
df_agg
```
```
    User  total_amount  average_amount  total_value  value_std
0  user1          18.0        9.000000           11   0.500000
1  user2          20.5        6.833333           15   0.816497
2  user3          10.5       10.500000            4   0.000000
```

Inspired by [this](https://stackoverflow.com/questions/39568965/how-to-reset-indexes-when-aggregating-multiple-columns-in-pandas) and [this](https://stackoverflow.com/questions/55817201/pandas-groupby-chaining-rename-multi-index-column-to-one-row-column) stackoverflow posts.
