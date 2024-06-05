---
layout: post
title: "The Value of Functions"
date: 2017-02-18 21:49:07 +0000
comments: true
redirect_from: '/blog/2017/02/18/the-value-of-functions/'
categories: ["aws","FaaS", "lambda", "Azure functions", "serverless"]
---
With the emergence of FaaS entering the mainstream, we are starting to see some real world business value in using such an architecture. Many of the benefits of using FaaS are discussed elsewhere:  

- pay as you go
- no-ops
- limited provisioning
- performance scalability
- lower running costs

But some of the benefits are more subtle. The benefit I want to highlight is this:

# You can measure how much features cost to run. 

While this may not seem like a winning feature, let's think about it for a second.  
You have a feature on your website called 'price match'. What it does is matches prices against your competitor's prices. To do this it scrapes each of your competitor's sites to get their price of the product. If you ask the product owner how often they want the prices updated they'd probably say something like 'as much as possible' or 'real time please'. In the old world, we would probably hit it as often as we could and be done with.

Now with FaaS you have some additional information to help you make decisions, this feature will cost 0.001¢ to run per competitor per product. You can then extrapolate different price points for different frequencies and make a business case for each of them. Or better yet measure the value of having it run every minute vs. every hour.

# Summary

As an industry, we can genuinely begin looking at the running costs of an individual feature. This started at the product level with 'Lean Startup' and will, I believe, continue to a greater degree with individual features and functions.  

Features can be added and removed more quickly with the costs and benefits of them being more transparent to both developers and product owners. 
