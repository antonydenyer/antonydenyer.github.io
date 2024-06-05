---
layout: post
title: "Developer environments and production parity"
date: 2020-03-23 08:00:00 +0000
comments: true
categories: ["devops", "docker", "docker-compose", "vargrant"]
---

When I first started writing software, a common practice was to use an emulator or a lightweight version of something for your development environment. The most common of these were in-memory databases, things like SQLite and H2. They are lightweight, easy to use and easy to segregate. You didn't need to worry about licensing or having a lot of resources for it to run. As machines have got more powerful and FOSS databases have become more mature, the reasons for not having a real database running for development are diminishing. The problem now seems to be more about orchestration.

# clone; build; run

When you had a website that just needed a database life was easy. You could clone the repo, build project and normally run without any issues. You might need to switch the language of the tool you were using but that was about it. Generally speaking tools like (nvm)[https://github.com/nvm-sh/nvm] and (direnv)[https://github.com/direnv/direnv] enabled you to switch easily. You could use an in-memory database and be up and running. The entire site was running from a single repo. Fast-forward ten years and you've got multiple repos with different databases, dependencies and languages.

You could clone each repo and run everything on your local machine, but that gets messy and painful very quickly. There are a couple of options available to you, but most of them assume that you are already using some kind of infrastructure management tool i.e. ansible/chef/docker.

## Shared environments

Sharing a database between your team can work, but it's less than ideal. You will end up with schema conflicts and inconsistencies. There's one scenario where it can work, and that is when the data is read-only. Maybe your front end code can get away with pointing at a shared test API? 

## Vagrant

Vagrant has been around for almost a decade now. Its main purpose was to provision a development box to address this problem. Note that the problem it was tackling at the time was one where a developers machine could have some custom software installed on it, things like drivers, licensed tools, database etc. The approach it took was to create a virtual machine with all the stuff installed that you needed to run your code. The problem is that there is a disconnect between your vagrant scripts and server setup. This inevitably leads you down the path of sharing your infrastructure setup scripts with your dev team. Which leads to another problem, where do you store them, and how do you share them? 

## Docker

Personally, I like running docker images. I love having an image that defines everything that my application needs to run. It could be a pdf library, a particular runtime environment, whatever; the great thing is that it's on me to get right. If you are running docker images in production, you'll most likely be using some kind of orchestration (k8s, mesos, etc.). To run those images, you need to have a container registry. It means that you can utilise that container registry for running up services locally. To this, I usually use docker-compose and define all my integration dependencies there.

# Summary

Whether you're using docker-compose or vagrant, you should be using something to ensure that your development environment is as consistent with production as possible. New developers can be productive more quickly and established developers can have more confidence that their code will work in production. You should be aiming for:

# clone; up; build; run
