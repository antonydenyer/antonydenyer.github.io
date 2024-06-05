---
layout: post
title: "MVC and our interpretation at 7digital"
date: 2011-06-21 17:49
comments: true
redirect_from: '/blog/2011/06/21/mvc-and-our-interpretation-at-7digital/'
categories: mvc
---

Introduction
The following details concepts that have been adopted on the site I'm currently working on. This was born out of a discussion that took place among the devteam at the time. Differences in phraseology and meaning were ironed out to come up the following definitions and responsibilities.

Queries(GET)/Commands(POST)

* Contain all the information required to perform an action.  
* Data should be bound onto the query before it hits the controller action.  
* They are POCO and contain no logic. 

Controllers

* Marshall request to handlers and mappers  
* Creates the view model from the handler result 

Handler

* Process queries/commands  
* returns various types (Model/Entity/API DTO) 

View Models

* Contains data that you are using in the view, should not expose anything other than view models/value types. 

View

* Stuff you show the user, should only contain logic that is required to get the stuff displaying correctly. 

Wrappers

* Wrap calls to external api 

Services

* Do things that aren't simple, pull together bits and bobs and package them up. Take care of business logic that have cross cutting concerns. 

Mappers

* Mappers simply convert one object to another. They DO NOT perform any other logic. 

Charts Example from http://www.7digital.com/b/charts

ArtistChartController

* Takes the query from the web request (which has been bound in the pipeline)  
* Passes the query to a handler  
* Maps response from handler to view model using auto mapper 

ArtistChartQueryHandler

* Takes the query  
* Does some work to get some configuration options  
* Builds up the query further to execute against the API wrapper  
* Returns API DTO 

Api Wrapper

* Takes query  
* Executes against external API  
* Parses responses and maps into API DTO object  