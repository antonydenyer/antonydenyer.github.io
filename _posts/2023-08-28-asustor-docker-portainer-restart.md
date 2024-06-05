---
layout: post
title: "Problems with docker and portainer restarting on asustor nas"
date: 2023-08-28 09:00
comments: true
categories: [asustor, nas, docker, portainer]
---

My asustor NAS drive seems to suffer from a strange problem after a reboot. For whatever reason the docker network bindings don't work correctly and none of the exposed ports work any more.

I worked round the problem by restarting docker using the root users cron table.

```sh
@reboot /volume1/.@plugins/AppCentral/docker-ce/CONTROL/start-stop.sh stop
@reboot /bin/sleep 300; /volume1/.@plugins/AppCentral/docker-ce/CONTROL/start-stop.sh start

```