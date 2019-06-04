---
layout: post
title: "HOWTO: Deploy mono web applications with capistrano-mono-deploy"
date: 2012-12-13 20:34
comments: true
redirect_from: '/blog/2012/12/13/howto-deploy-mono-web-applications-with-capistrano-mono-deploy/'
categories: [capistrano,mono]
---
I've recently been working on a capistrano gem to help with mono deployments. I've had a bit of experience trying to get capistrano to work with non-rails environments, basically it can be a bit of pain if you're not sure what you're doing. Also most of the beginners tutorials are aimed at people developing Ruby on Rails. So I though I'd create a gem and provide a little how to. The gem is called [capistrano-mono-deploy][1]. This is how you can use it to deploy a simple web application to a Linux machine. In this example I'll be using the excellent [ServiceStack ][2]for the web application and xsp4 to host the it. You can find all the code on [my github account][3].
## Mono 
I'm running Ubuntu 12.10 on my server and am using Mono 2.10.8.1 which was installed from the package repository using apt-get

    sudo apt-get install mono-xsp4

  
## Service Stack 
First I knocked a quick service stack application and created a status endpoint. This should work for any application that's mono compatible but I haven't tried with anything else. Once you've got an application you'll want to be deploying it.

## Capistrano 
You need to make sure you have capistrano and ruby gems installed first. I'm running this from my Ubuntu development machine and have not tried deploying from a windows machine to 'nix machine.

Install ruby and ruby gems   
    sudo apt-get install ruby1.9.3 rubygems
  
Install bundle 
    sudo gem install bundle capistrano

  
Add a Gemfile and insert the following
    source :rubygems
    gem "capistrano-mono-deploy"

Call bundle to update the gems
    bundle

Capify your project
    capify .

Edit your config/deploy.rb to be something like this:
    require "capistrano/mono-deploy"
    
    ssh_options[:keys] = %w('~/.ssh/*.pub')
    
    set :application, "service stack deployed with capistrano running on mono"
    set :deploy_to, "~/www" 
    
    role :app, "the.server.com"
    
Then from the root of your project call
    cap deploy
    
#### BOOM your application is deployed! 
Assuming you have your ssh keys setup for the server, either that or you had to enter a password. 
      
Now I wouldn't recommend using xsp as your weapon of choice when it comes to hosting, but it's easy to get started with and you can swap it out for other things once you get up and running.
    
You should take some time to read up about capistrano and how it works. It's a really powerful tool that has loads of options available for you to play with. The capistrano-mono-deploy gem currently deploys the first directory it finds with a web.config located in it. 
    
Feedback appreciated, particularly on my terrible ruby code!

 [1]: https://github.com/antonydenyer/capistrano-mono-deploy
 [2]: http://www.servicestack.net/
 [3]: https://github.com/antonydenyer/servicestack-mono-capistrano  