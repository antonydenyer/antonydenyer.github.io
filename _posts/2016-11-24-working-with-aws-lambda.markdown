---
layout: post
title: "Working with AWS Lambda"
date: 2016-11-24 15:23:59 +0000
comments: true
redirect_from: '/blog/2016/11/24/working-with-aws-lambda/'
categories: ["AWS", "AWS Lambda", "FaaS", "Serverless"]
---

Here is a collection of tips for using AWS lambdas. These tips are based on our experiences of using lambdas to ingest a legacy database into a new elasticsearch cluster.

Some tips are FaaS specific whilst others are more generic.

## Separate your concerns and be loosely coupled ##
For some reason, people seem to forget best practices when working with FaaS and to a certain extent microservices. The late Jim Weirich did an excellent talk on [decoupling from Rails](https://teamgaslight.com/blog/jim-weirich-on-decoupling-from-rails). It's a common (mis)practice with Rails developers to tightly coupling themselves to the framework. With FaaS no matter if you're using a framework or the service provider directly you should aim to abstract yourself away, put clear water between your code and integration code. It can be difficult as most examples are glib and to a degree they want you to be tightly coupled. Create and test libraries in isolation by defining your inputs and outputs, then wire it up to your framework of choice. Separate your concerns - framework integration and business logic.

## try/catch all the things ##
With AWS Lambda it appears that if an exception is thrown in your code, the function will exit without completing. A simple solution is to wrap your entry point code in a massive "try catch".

``` js
module.exports.handler = (event, context, callback) => {
  try {
    const response = myLibrary.handle(event)
    callback(null, {statusCode: 200, message: response})
  } catch (error) {
    context.fail({statusCode: 500, message: JSON.stringify(error)})
  }
}
```

## use promises ##
Personally, I prefer using promises. I think they're aesthetically more elegant and it allows you to bubble errors to their most appropriate level. In this case the lambda response.

``` js
module.exports.handler = (event, context, callback) => {
  myLibrary
    .handle(event)
    .then(response => {
      callback(null, {statusCode: 200, message: response})
    })
    .catch (error => {
      context.fail({statusCode: 500, message: JSON.stringify(error)})
    })
}
```

However, there's a catch (*rimshot*), if any of your code is doing anything synchronously, i.e. in this case not in a promise you still need that big "try catch" again. It has caught us out with the most mundane of errors. When you're trying to debug and understand what's going on it's a life saver.

So our code becomes:

``` js
module.exports.handler = (event, context, callback) => {
  try {
    myLibrary
      .handle(event)
      .then(response => {
        callback(null, {statusCode: 200, message: response})
      })
      .catch (error => {
        context.fail({statusCode: 500, message: JSON.stringify(error)})
      })
  } catch (error) {
    context.fail({statusCode: 500, message: JSON.stringify(error)})
  }
}
```

## log it out ##
Simply put you've got this black box that you deploy to, and that's it. No debugging, no helpful information, nothing. We log just about everything so that when we get problems (and you will get them), we can reproduce them locally. We use [winstonjs](https://github.com/winstonjs/winston) but you can just ```console.log``` if you wish. 

Now that we want to log things out our handler code becomes:

``` js
module.exports.handler = (event, context, callback) => {
  try {
    logger.info('Handling:${context.awsRequestId}', event)
    myLibrary
      .handle(event)
      .then(response => {
        logger.info('Finished:${context.awsRequestId}', response)
        callback(null, {statusCode: 200, message: response})
      })
      .catch (error => {
        logger.error('Failed:${context.awsRequestId}', error)
        context.fail({statusCode: 500, message: JSON.stringify(error)})
      })
  } catch (error) {
    logger.error('Failed:${context.awsRequestId}', error)
    context.fail({statusCode: 500, message: JSON.stringify(error)})
  }
}
```

## Keep it warm ##
We've noticed that the larger your package gets, the longer it takes to warm up. You can keep your lambdas warm by having a status check call the lambda periodically. As your load increases, you will cross a threshold whereby you will not need to do this. However, given that you're keeping a function warm as opposed to a service the chances are you'll have functions that don't get called very often. 

## Conclusions ##
FaaS pushes you to think about software in a different way. It makes it easier for you to think about messages and separating concerns. 
