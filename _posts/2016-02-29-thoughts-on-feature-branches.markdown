---
layout: post
title: "Thoughts on feature branches"
date: 2016-02-29 10:59:33 +0000
comments: true
redirect_from: '/blog/2016/02/29/thoughts-on-feature-branches/'
categories: ["Continuous Delivery","Feature Branches", "Continuous Integration"]
---

There seems to be two main schools of thought with regards to feature branches. Some argue that feature branches are an abomination and should not be considered as 'continuous' integration let alone delivery. Whilst others suggest that feature branches are a way of allowing each developer to work without disturbing everyone else.  

In my opinion the golden rule for doing continuous delivery (and therefore integration) is keeping the master branch pristine. By that means I mean every commit on master should be buildable, testable and deployable. If you commit something to master you should be happy for it to be deployed.  

Lets start by exploring some different mindsets and working practices that I've experience when not using feature branches.  

### Feature Toggles or Branch By Abstraction ###
I've seen this work with disciplined developers. They ensure that before every commit all tests are passing. They stop things from being available to users too early by using [Branch By Abstraction](http://martinfowler.com/bliki/BranchByAbstraction.html) or [Feature Toggles](http://martinfowler.com/bliki/FeatureToggle.html). They tend be happy with this arrangement and feel the benefits outweigh the overhead.  

### The 'other' way ###
Some developers don't agree with this, they think that the added work and overhead of [Branch By Abstraction](http://martinfowler.com/bliki/BranchByAbstraction.html) or [Feature Toggles](http://martinfowler.com/bliki/FeatureToggle.html) isn't worth the effort. They do, however, think that committing to master as much as possible is a good thing. The problems happen when they perceive that what they're working will take more than a few days and they **don't know how to break it down**. The developer will work on their local machine until everything is done. Along the way they make several commits locally but do not push them upstream.  

The question is what happens next, in my experience one of a few things can happen.  

#### Squash commits ####
The developer rebases their local version and squashes their work down to a single commit. They run all the tests locally and then push their changes. The problem with this is that you get one large commit at the end of the feature. However, you do get a single deployable commit that represents the feature that was worked on. It can be difficult to see what has happened but at least you can see when it happened. It falls down when people feel the need to work on a feature over an extended period of time. **They think that the problem can not be broken down.**  

#### Rebase commits ####
The developer sees the downside of squashing their commits. They want to maintain that incremental history so rebasing their work against master makes sense. They run all the tests locally and then push the many commits upstream. A common thing I see is developers prefixing every commit message with the feature they were working on. It can be used to make it clearer when work on a feature started and ended, but it's just convention. There are a couple of problems with this approach. You haven't run a deploy for every commit. The chances are your CI tool is just going to build the latest commit. And it's not as easy to revert as you have to revert many commits. You're also relying on the developer to clearly indicate when the feature started.  

#### Merge maniac ####
I've seen many developers who just don't understand git or its elegance. However, they still want to do continuous integration so they can put it on their CV. They pull from master all the time without rebasing and push their commits whenever they can, otherwise they might have to do a merge. The teams git history ends up looking like [Clapham Junction](https://en.wikipedia.org/wiki/Clapham_Junction_railway_station#The_station_today).  

#### Summary ####
Doing small single deployable commits with a combination of [Branch By Abstraction](http://martinfowler.com/bliki/BranchByAbstraction.html) and [Feature Toggles](http://martinfowler.com/bliki/FeatureToggle.html) has worked for me in the past. It does require buy-in, discipline and understanding from the whole team.  

## Feature branches - a simpler option? ##

I'm not a huge fan of feature branches. But they can push people towards better behaviour. To clarify, a feature branch is when someone takes a branch of master and works on it to implement a single feature.  

### The horror scenario ###
The worst case for feature branches is when it's a flaccid feature branch. It's used a means of isolating development from the rest of the team. When the developer is finished with their work they merge everything onto master. Again the team git history ends up looking like [Clapham Junction](https://en.wikipedia.org/wiki/Clapham_Junction_railway_station#The_station_today). The only benefit the developer has gained is that it makes it easy to pair with other people on the team.  

### The long lived scenario ###
Another bad case is when someone is working on a branch for a long period of time (anything over 1 day in my book). The developer doing the work probably wont have any problems merging or integrating their code the rest of the team will. The problem occurs after the developer has finished with the their branch and merges back onto master. Their team mates will try and integrate their code. The repository history may not look horrific but the experience of other developers on the team will be. This is the very opposite of what continuous integration is about.  

### The ideal scenario ###
For me if you decide to use feature branches you should do so in the following manner. Firstly you should ensure that your CI tool supports it and is setup to build each feature branch ([Teamcity](https://confluence.jetbrains.com/display/TCD8/Working+with+Feature+Branches), [Jenkins](https://wiki.jenkins-ci.org/display/JENKINS/Feature+Branch+Notifier+Plugin)). Every commit to the branch should be rebased against master and pushed to be built. Any build failures should be corrected by amending that commit and being force pushed against that feature branch. Every time you need to rebase against master each commit on your feature branch should be re-tested. When you are happy with the feature, merge with master using [--no-ff](http://stackoverflow.com/a/21717431/1155026). This gives you a nice indication of what was included in a feature. This way of working also provides pressure on the lifetime of the branch. The older it is the harder it is to rebase.  

## Conclusions ##
If you want to benefit from continuous integration and therefore continuous deployment you need to seriously think about how to do it. Your team needs to be disciplined, mature and highly collaborative. The problems people face with continuous integration are not the tooling, but how they breakdown work. Often you'll see people start working on a feature before they understand the problem and the goal. Personally I find pairing realy helps me out with this. Somehow it seems more justified spending time breaking down and understanding a problem when there's two of you. The act of making something clear to both pairs is normally enough to break the work down to the size of a single commit.  

There are times when this may not be possible, perhaps a large file reorganisation. I've found it's better for everyone to just stop working whilst the change happens or better yet try [mobbing](https://en.wikipedia.org/wiki/Mob_programming) the problem. Everyone in the team needs to understand these changes. Could there possibly be a better use case for it?





