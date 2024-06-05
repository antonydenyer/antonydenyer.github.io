---
layout: post
title: "Tight Coupling x Complexity"
date: 2020-03-04 08:00:00 +0000
comments: true
categories: ["developers"]
---

When building systems you will quite often have to make the tradeoff between having a system that is loosly coupled or one that is simple. When we think of tight coupling we tend to think of it as a bad thing. But that is not always the case, you need to understand what you are being tightly coupled to. A common example of tight coupling people give is database queries, most of the time people will advocate using ORM as it offers a loosly coupled way of connecting to a database. In theory you can swap out your database server with another one. The reality is usally somewhat different, the chances are that you've leaked database speicific concepts outside of your 'repository' classes. You might be able to switch between similar types of database (MySQL to PostgreSQL for example) without doing anything but are you really going to be able to swap out MongoDB for PostgreSQL? I doubt it. Further more, you are tightly coupled to the ORM.

# Why loose coupling?

The objective with having things loosley coupled is to enable or facilitate future changes and maintence of sofware. The idea being that loosly coupled software can be nurtred and changed over time. 

# When is tight coupling okay? 

You can tightly couple to things that don't change. Personally I think it's okay to be tightyly coupled to things that have standards. For instance, I'm okay with being tightly coupled to SMTP but not say AWS SES or Mandrill. You are tightly coupled to the way you are sending messages to people (SMTP) but not to the provider of that service. 