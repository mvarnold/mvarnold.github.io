---
layout: post
title:  "n-gram Usage Rate Model"
date:   2020-05-29 08:46:30 -0500
categories: jekyll update
---
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

## Introduction

Interested in building instruments to measure large scale language use? 
Want to do sentiment polling around topics close to your heart? 
With access to Twitter data, these objectives seem within reach. But just how reliable are the results? 

One of the most fundimental tasks we might be interested in is estimating a word or $n$-gram's usage rate over time.
How accurate should we expect usage rates to be, is a vital question, especially for ambient usage rates, which might be computed from less than $$10^3$$ tweets or from more than $$10^6$$ tweets on a single day. Understanding the variation in usage rate espected for a given sample size of tweets is the main objective here.
![Observed Usage Rate]({{ "/images/observed_ambient_usage_rate.png" | absolute_url }})


