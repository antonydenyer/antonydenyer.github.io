---
layout: post
title: "ServiceStack the way I like it"
date: 2012-09-20 19:52
comments: true
redirect_from: '/blog/2012/09/20/servicestack-the-way-i-like-it/'
categories: ServiceStack
---

Over the last month we've started using ServiceStack for a couple of our api endpoints. Here's a breakdown of how I configured ServiceStack to work the way I like it. Hopefully you'll find this useful.   
##Overriding the defaults
Some of the defaults for ServiceStack are in my opinion not well suited to writing an api. This is probably down to the frameworks desire to be a complete web framework. Here's our current default implementation of AppHost: 
{% gist 3755430 %}
For me, the biggest annoyance was trying to find the DefaultContentType setting. I found some of the settings unintuitive to find, but it's not like you have to do it very often!   
##Timing requests with StatsD
As you can see, we've added a StatsD feature which was very easy to add. It basically times how long each request took and logs it to statsD. Here's how we did it:   
{% gist 3755475 %}
It would have been nicer if we could wrap the request handler but that kind of pipeline is foreign to the framework and as such you need to subscribe to the begin and end messages. There's probably a better way of recording the time spent but hey ho it works for us.   
One of my biggest bugbears with ServiceStack was the insistence on a separate request and response object, the presence of a special property and that you follow a naming convention all in the name of sending error and validation messages back to the client. It's explained at length on the [wiki][1] and Demis was good enough to answer our [question][2].   
##RestServiceBase 
The simple RestServiceBase that comes with provides an easy way of getting started, but there aren't many hooks you can use to manipulate how it works. It would be nice if you could inject your own error response creator. We ended up inheriting from RestServiceBase and overriding how it works: 
{% gist 3755906 %}
We basically chopped out the bits we're not using and changed the thing that creates the Error Response: This allows us to respond with an error no matter what the type is for the request or what response you are going to send back. It provides us with extra flexibility above what is provided out of the box. In a nutshell, if there's an exception in the code we will always get a stack trace in the response if debug is on.   
##Validation 
We had the same issue with the validation feature; if you don't follow the convention you don't get anything in the response body. So we followed the same practice and copied the ValidationFeature and tweaked it how we wanted it. 

 {% gist 3756013 %}
  
I like ServiceStack; it's really easy to get up and running and whilst it has it's own opinions on how you should work, what framework doesn't?

 [1]: https://github.com/ServiceStack/ServiceStack/wiki/Validation
 [2]: https://github.com/ServiceStack/ServiceStack/issues/248  


##Follow up from mythz

Hey Antony,

Nice Post and it's always good to see fellow ServiceStack on Mono deployments in the wild :)

I've just been deploying services on Linux with a custom build script, pulling from straight from git + building - but Capistrano sure sounds good so I'll be definitely be checking that out, thx! If you'd like to help out others wanting to do the same thing we'd love a wiki of your deployment strategy with Capistrano if you can find the time: https://github.com/ServiceStack/ServiceStack/wiki

I just want to clarify a few of the issues you had...

Most of these issues you ran into should now be resolved with our new API Design we just released:
https://github.com/ServiceStack/ServiceStack/wiki/New-API

It's pretty much superior in everyway to the old API design and I think you'll be right at home with the new API which should require a lot less changes to customize "ServiceStack the way you like it" :)

> This is probably down to the frameworks desire to be a complete web framework..

We initially had some restrictions with the old design to enable a typed, end-to-end message-based API since we believe it promotes the optimal design for remote services.

Although we've since lifted this restriction and you no longer need a Conventionally named Response DTO as we'll return a Generic ErrorResponse. This lets you return "Pure Collections" but it still retains structured exception handling on the client.


> For me, the biggest annoyance was trying to find the DefaultContentType setting. I found some of the settings unintuitive to find...

I'm not sure why this was so hard to find as there's only 1 place to set ServiceStack's Configuration as you've done in SetConfig() and the Field you need to set is called DefaultContentType. Also the first result when searching for it in Google: https://www.google.com/search?q=servicestack+default+content+type&sugexp=chrome,mod=13&sourceid=chrome&ie=UTF-8
brings up this gem: http://stackoverflow.com/questions/10317225/servicestack-default-format
Which IMO explains the situation pretty well.

> One of my biggest bugbears with ServiceStack was the insistence on a separate request and response object...

With the new API we've relaxed this restriction and you can now specify what each service returns (even void if you want :)

> We had the same issue with the validation feature; if you don't follow the convention you don't get anything in the response body

It's also much easier to override the default Exception handling which you can do just by providing your own IAppHost.ServiceExceptionHandler.

The StackTrace is populated if DebugMode is enabled (which is now automatically inferred based on Debug/Release build configuration of your AppHost project).

Your DebugMode() code looks a little long, you should check out ServiceStack's AppSettings which lets you do:

var appSettings = new AppSettings();
var debugMode = appSettings.Get("DebugMode", false); //return false if "DebugMode" appSetting doesn't exist.

You can even deserialize AppSetting configs in List's, POCO types etc. More info at: http://www.servicestack.net/mvc-powerpack/

Also since you've done your own StatsD impl, you may also want to check out what we've done in Request-logger plugin, which we've found invaluable for analyzing external client requests: https://github.com/ServiceStack/ServiceStack/wiki/Request-logger

With our new API design your StatsD impl would now just need to inherit a custom ServiceRunner and just override the hooks you need. This is explained in more detail in the wiki, but basically it's now decoupled from the service implementation which results in a more testable Service and ServiceRunner classes.