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

This is the model I came up with.
We begin with the assumption that we have a poission process generating a word's usage rate $$f\_\tau$$, 
which we try to estimate, $$\bar{f}\_\tau$$. 
The true usage rate is constant, but as we increase the size of our sample $$\lambda\_\alpha$$, we expect the error of our estimate to decrease. The size of our sample is explicitly modeled as the counts of the anchor word in our sample, but this should approximately correspond to the number of tweets we need to sample for a given acceptable error level.   

%more model description

The first panel of Figure 1 shows the raw output of our model. Three examples are shown with true usage rates $$\mu = 0.01$$, $$\mu = 0.0001$$ and $$\mu = 0.000001$$. For a target word with true usage rate 0.01, if the anchor counts $$\lambda\_\alpha$$ are less than $$10^2$$, the measured usage rate often is zero.

This zero usage rate results in misleading plots on a log scale, were zero values are omitted, and the remaining values are inflated. In real ambient usage rate data, we often see a decline in usage during the early days of twitter when total tweet volume orders of magnitude lower than today. 
However, this appears to be an artifact of omitting zero days, and something a rolling average could potentially rectify.  
![Observed Usage Rate]({{ "/images/observed_ambient_usage_rate.png" | absolute_url }})
*Figure 1: Poission usage rate model. The top panel shows the increase in accuracy as we increase $$\lambda\_\alpha$$, approximately the number of tweets used to compute an ambient usage rate. The middle panel shows how failing to include the zero values in mean of the usage rate causes an increasing inflated estimated usage rate. The bottom panel shows how Mean Squared Error decreases with increased sample size, and how estimates with and without zero values converge.*

| Example N-Gram | Usage Rate | Rank | Minimum Sample Size |
|-------|--------|---------|---------|
| people | 0.0011 | 100 | $$10^3$$ |
| enjoy | 0.00011 | 800 |  $$10^4$$ |
| snap | 0.000013 | 5000 | $$10^5$$ |
![Sampled Rank Divergence]({{ "/images/Sampling_distribution.png" | absolute_url }})
*Figure 2: Convergence of word distributions as we increase the size of a random sample of tweets containinga given anchor word. We expect the rank divergence to converge to zero as the number of tweets increases. When this occurs depends on how many discrete tweets exist in our main collection from which we sample, and the usage rate of the given word.*
