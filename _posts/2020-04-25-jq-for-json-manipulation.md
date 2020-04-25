---
layout: post
title:  "Using jq for manipulation json in command line (updated regularly)"
date:   2020-04-25 04:47:00
subtitle: 
img:
tags: [command line, data manipulation] # add tag
published: true
---

[jq][https://stedolan.github.io/jq/] is a really handy command line tool to manipulate json files on the fly in the command line.

In this post I will keep updating the useful tricks I learnt about jq.


### Counting number of items in a json file
```
jq length filename.json
```