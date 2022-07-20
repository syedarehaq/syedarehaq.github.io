---
layout: post
title:  "Python pandas useful tricks (updated continuously)"
date:   2022-07-20 17:17:00
subtitle: 
img:
tags: [elastic,nlp] # add tag
published: true
---
### Filter the results using a text analyzed field

When limiting the search results, as in filtering the results using the `term` search, the field should be `TEXTFIELD.keyword`. This is the default behavior of the default analyzer. If we do not analyze the field, only in that case we can directly use `term` with the exact `TEXTFIELD`. Below, I am providing a correct example of how to do the filter.
```python
{
    "query":{
        "bool":{
            "filter":[
                {"term":{"TEXTFIELD.keyword":"matching term"}}
            ]
        }
    }
}
```
Source:[https://discuss.elastic.co/t/elastic-term-filter-not-working-when-term-contains/107301]{https://discuss.elastic.co/t/elastic-term-filter-not-working-when-term-contains/107301}