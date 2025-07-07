---
layout: post
title: "The Walking Skeleton: Breathing Life into Software"
date: 2025-07-07 09:00 +0000
comments: true
categories: [agile, devops]
---

I can't remember when I first learnt about walking skeletons, it was at least a decade ago, and it felt like common knowledge among the people I was working with. [Growing Object-Oriented Software, Guided by Tests](https://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627) was the I think the first time I saw it formalised.

Since then it would appear that this common knowledge has been lost to the anals of time. I still fully understand why I get so much pushback for what seems to me to be the blindingly obvious. Let's take another look at the metaphor!

A walking skeleton is a bare-bones but functional version of your system that touches every major part, like a skeleton brought to life. It's not strong yet, but it walks, just about, and can stand up. Then you add muscle, flesh, and structure. In terms of development, it's the minimal thing needed for the system you're building. 


Practical Example

Say you're building an api that serves up HTTP requests, it doesn't really matter what the product does, but we might know some things up front for what we're going to need. What we're trying to get to is a minimum viable process whereby a developer can make changes in an application and have them automatically deployed to production. What this means in practice will normally mean some automated workflow process that will check code, create some form of package and make that package available to a web server. For the majority of cases today, it means building a Docker image.


### Docker Build

Docker has been around for over a decade now, along with Kubernetes. In reality, it's the de facto deliverable for the majority of projects. My rules of thumb for building Docker images. Always build a Docker image for every commit to trunk/main/master. If it's slow, fix it! Tag images with the Git commit SHA. Push the image to a single container registry. Include the Git commit SHA within the application at build time.


```docker
FROM debian:bullseye-slim

WORKDIR /app
COPY entrypoint.sh .

ARG GITHUB_SHA
LABEL org.opencontainers.image.revision="$GITHUB_SHA"
ENV GITHUB_SHA="$GITHUB_SHA"

ENTRYPOINT ["/app/entrypoint.sh"]
```

This process ensures:

- Traceability from Docker image to source commit.
- Idempotent builds.
- Simplified versioning and environment consistency.

### Deploying the application

Deployments are then simply a matter of applying the desired version. In a k8s context, it's simply changing the image tag. Whilst I could advocate for having your entire process under source control and automated, there are diminishing returns on investment. You want to be in a situation where the tasks you perform frequently are automated. Whether or not you automate everything will depend on your scenario. 


The primary objective is to enable speed of deployment. Anything that hasn't been deployed and is in the hands of customers is considered programming inventory, which comes with associated costs. You should be aiming for minimum viable deployments. 

