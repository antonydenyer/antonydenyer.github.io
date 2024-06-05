---
layout: post
title: "Deploying ServiceStack to debian using capistrano hosted using nginx and fastcgi"
date: 2012-09-21 20:13
comments: true
redirect_from: '/blog/2012/09/20/servicestack-the-way-i-like-it/'
categories: [debian,fastcgi,mono,ServiceStack]
---
Over the last month we've started using ServiceStack for a couple of our api endpoints. We're hosting these projects on a debian squeeze vm using nginx and mono. We ran into various problems along the way. Here's a breakdown of what we found and how we solved the issues we ran into. Hopefully you'll find this useful.   
##Mono
We're using version 2.11.3. There are various bug fixes compared to the version that comes with [squeeze][1]. Namely, problems with min pool size specified in a connection string. Rule of thumb, if there's a bug in mono then get the latest stable!   
##Nginx
We're using nginx and fastcgi to host the application. This has made life easier for automated deployments as we can just specify the socket file based on the incoming host header. There's a discussion about what's the best way to host ServiceStack on Linux at [stackoverflow][2]. Our nginx config looks like this:
{% gist 3755395 %}  
When we deploy the application we nohup the monoservice from capistrano 
{% gist 3755449 %}
Where:  
fqdn is the fully qualified domain name, i.e. the url you are requesting from  
latest_release is the directory where the web.config is located.  
##Capistrano
To get the files onto the server using capistrano we followed the standard deploy recipe. In our set up we differ from the norm in that we have a separate deployment repository form the application. This allows us re-deploy without re-building the whole application. Currently we use Teamcity to package up the application. We then unzip the packaged application into a directory inside the deployment repository.   
We use deploy\_via copy to get everything on the server. This means you need to have some authorized\_keys setup on your server. We store our ssh keys in the git repository and pull all the keys into capistrano like this: 
    
    ssh_options[:keys] = Dir["./ssh/*id_rsa"]
###No downtime during deployments ... almost
Most people deal with deployment downtime using a load balancer. Something like take server out of rotation, update the server, bring it back in. The problem with this is it's slow and you need to wait for server to finish what it's doing. Also in our application we can have long running background tasks (up to 1000ms). So we didn't want to kill those of when we deployed. So we decided to take advantage of a rather nice feature of using sockets. When you start fastcgi using a socket it gets a handle to the file. This means that you can move the file! Meaning that you can have a long running process carry on whilst you move you new site into production, leaving that long running task running on the old code to finish. Amazing!   
This is what we have so far:   
{% gist 3760738 %}
There's room for improvement feedback appreciated.

 [1]: http://packages.debian.org/squeeze/mono-runtime
 [2]: http://stackoverflow.com/questions/12188356/what-is-the-best-way-to-run-servicestack-on-linux-mono/12298508#12298508  