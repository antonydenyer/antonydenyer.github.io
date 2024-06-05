---
layout: post
title: "running an image on kubernetes with an interactive shell"
date: 2019-02-26 15:12:09 +0000
comments: true
redirect_from: "/blog/2019/02/26/running-an-image-on-kubernetes-with-an-interactive-shell/"
categories: ["kubernetes", "k8s", "geth", "ethereum"]
---

If you have an existing kubernetes cluster that you are using you may need to get a shell running from inside that cluster to help you debug things. In my scenario access to an ethereum client was locked down to only allow access from the cluster IP address. 

### Run a geth client on kubernetes

``` bash
kubectl run --namespace default geth --rm --tty -i --image ethereum/client-go --command /bin/sh 
```

This will give you a command promt so that you can ```geth attach http://node-address.com```

### Run a mongo client on kubernetes

``` bash
kubectl run --namespace default mongo-client --rm --tty -i --image bitnami/mongodb --command -- \ 

mongo admin --host mongo.db.hostname --authenticationDatabase admin -u root -p password 
```


### Port forward from a kubernetes cluster to your local machine

Sometimes you want to run some local tools but connect to soemthing in the cluster, eg connect to a mongo db running in kubernetes.

``` bash
kubectl port-forward --namespace default svc/mongo-mongodb 27017:27017
```

[k8s Docs](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)