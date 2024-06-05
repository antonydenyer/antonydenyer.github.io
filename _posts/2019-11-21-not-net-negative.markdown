---
layout: post
title: "Not Net Negative Developers"
date: 2019-09-26 08:00:00 +0000
comments: true
categories: ["developers"]
---

I've worked with many developers over the years. More recently the majority of them have been junior developers. When working with someone junior the best you can hope for is not net negative [NNPP](http://www.pyxisinc.com/NNPP_Article.pdf). By that, I mean that the developer doesn't have any impact on the project.

There will be a period where the vast majority of what they are doing is whatever you tell them to do. They will most likely not do it how you want it to be done. It will also probably take you longer to explain what needs doing than it would be to do it yourself. This is normal and should be expected for the first 3 months with a junior developer. This is going to have a negative impact on your project as you are spending time that could otherwise be used for other things. 


# Getting to Not Net Negative
The first thing you need to do is get your developer to a not net negative state. There are a few things that can help you and your developer get there.

1. Acceptance Criteria - generally speaking, you should already have for some acceptance criteria for a feature that you are working on. If this is not explicitly defined, you should do so know. The next step will be to break the task down and explicitly define acceptance criteria for each chunk. This will help you both break down the problem and ensure that you're both on the same page.

2. Framework - ensure that the area they will be working on already has some kind of framework in place. They should be able to work from what is already there. Let's say you're adding a sort parameter to a query; there should already be a place in the system that does this already. They shouldn't be adding brand new features to a system, only extending existing ones. 

3. Tests - make sure you already have a test suite in place. Your new developer should be extending existing tests as well as existing functionality. You want to make it easy for them to succeed.

4. Pipeline - ensure that you have in place a build pipeline that will protect them from making stupid mistakes. This means having an automated test pipeline with unit and acceptance tests. You want to make sure that they don't break anything right! 

## Summary
In general, it's about providing a safe environment for a new developer to be able to succeed in. That way you won't have to spend as much time manually checking work. You can focus on more higher-level priorities. If you get it right for a junior developer when you onboard more senior developers, they should be able to get up to speed more quickly. 