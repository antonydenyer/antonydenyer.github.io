---
layout: post
title: "Continuous Delivery - Chaos Build/Release Monkey"
date: 2014-08-28 09:16:25 +0100
comments: true
redirect_from: '/blog/2014/08/28/continuous-delivery-release-monkey/'
categories: "Continuous Delivery"
---

I think some people are missing the benefits of continuous deployment and are focusing on infrastructure and tools rather than process. The initial incarnation of this chain of thought was continuous integration, where by whenever you check in your code it's built by a central server. Ensuring that your code builds with other people’s code. Then it was taken a step further by having an automated deployment process. Your code would be automatically deployed after your build & unit test has successfully run. Then people took it further and started declaratively building the infrastructure that was required to run your application. 

### I think we've missed a step.

I think we've missed the continuous part of the puzzle. When you're actively working on a project this isn't a problem, code is being deployed on a regular basis, your tests are running and everything is being deployed. Most software projects are not being actively developed, they're the legacy applications that's running your cms system. In my opinion to truely benefit from continuous deployment you need to be building and deploying your code continuously, even if there's no new code. This helps keep everything upto date. Some automatic updates may break your deployment process and you won't know until you try and deploy that urgent bug fix. If your CI server is updating external dependencies you'll know when something gets deprecated straight away rather than in six months. 

## This is why we need a release chaos monkey

The Netflix [Chaos Monkey](http://techblog.netflix.com/2010/12/5-lessons-weve-learned-using-aws.html) brings about the idea of randomly failing infrastructure to push you into a state where your code and infrastructure becomes anti-fragile. Your builds and deployments should be the same, you should be able to deploy at a moment’s notice. The only way to, in my opinion, is to be randomly deploying your code. The tangential benefits are numerous. Your monitoring will improve as you won't be sitting there watching a deploy go out to production. The cost of code maintenance will be moved to the forefront as things break sooner. Developers will be less scared to work on legacy code as they know it's in a deployable state. 

In summary ....


![Release all the things](http://cdn.meme.li/instances/500x/53837120.jpg)