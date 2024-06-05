---
layout: post
title: "OctoChef - Abusing Octopus Deploy for Windows Server configuration Management"
date: 2013-07-25 09:38
comments: true
redirect_from: '/blog/2013/07/25/OctoChef-abusing-octopus-deploy-for-Windows-Server-Configuration-Management/'
categories: [Octopus Deploy,DevOps]
---

Unix has some great tools like chef to help you declaratively describe the configuration of your server. It means that you can almost guarantee the state of your servers and there's no need to log on to each server and manually configure them, freeing up people to do more interesting things with their lives. 

[Where I currently work](http://www.fundapps.co/about-us/join-our-team/) we've been using Octopus Deploy for over a year for automating our site deployments. Essentially with octopus you need to install a client on each box you want to deploy to. This means you have a rather powerful entry point to do whatever you like with the machine. Namely run powershell scripts. 

Currently we have no DNS server for our systems (we only have about 6 servers) and so previously we used a manually edited host file and copy and paste it everywhere. Here's how I got some of my life back.

## nupkg 

Octopus deploy works by utilising nuget packages for it's deployment, the Octopus Tentacle (the client/agent running on the machine) will pull down packages from a nuget repository and installs them on the server. When the Tentacle installs the package it can also [run some powershell scripts](http://octopusdeploy.com/documentation/features/powershell). This is where we start to abuse Octopus :)

The [OctoChef.hosts](https://github.com/antonydenyer/OctoChef.hosts) package contains three files

1. OctoChef.hosts.nuspec
2. tools\hosts
3. tools\Deploy.ps1

```
$hostsPath = "$env:windir\System32\drivers\etc\hosts"
Copy-Item .\tools\hosts $hostsPath
```
The powershell script grabs the host file and splats it over the top of the existing one, it's that simeple.
We used this rather than powershell remoting as we could utilise Octopus to distribute it to all our servers and keep track of what versions had been deployed without yet more manual steps. 

## Configuring Octopus
Now all we need to do is set up Octopus to run the package on our desired servers. Personally I went for a project per machine type and used the environments feature as normal. Then I set up teamcity to deploy to test environments when I commit to any of the OctoChef repositories. So if I commit to the OctoChef.hosts it will be pushed to all our test environments. Why not push to all environment? Fear!

## Testing
You can use [PS Unit](https://psunit.codeplex.com/) to test the powershell scripts. This can be setup in teamcity to unit test everything before you push it out.

I also wrote a little powershell script that will basically download the package and run Deploy.ps1, it's a bit pointless as you could just run the Deploy.ps1 script yourself. It does, however, provide some reassurance that your paths are correct and that the thing will actually run!

{% gist 6087279 %}