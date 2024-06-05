---
layout: post
title: "nuget Update-Package with a filter"
date: 2013-09-10 11:29
comments: true
redirect_from: '/blog/2013/07/25/OctoChef-abusing-octopus-deploy-for-Windows-Server-Configuration-Management/'
categories: [powershell, nuget]
---

One of my biggest annoyances about nuget and powershell in general is the lack of universal support for unix like features such as pipe and grep. 
Lets say you have a solution with multiple projects in and you want to update to the latest versions of your internal dependencies, you'd proabably do something like.

```
Update-Package CompanyFoo.Package1
Update-Package CompanyFoo.Package2
Update-Package CompanyFoo.Package3

```

In the gems world you could do something like

```
gem update `gem list | grep CompanyFoo | cut -d ' ' -f 1`

```

What I'd like to be able to do is something like

```
Update-Package -like CompanyFoo

```
Or even 

```
Update-Package | List-Package -like CompanyFoo
```
Sadly the closest I got was going though each packages.config file and filtering down the list

```
Get-ChildItem -path '.' -Recurse -Include 'packages.config' |
 Select-Xml -xpath '//package/@id' |
 Select-Object -ExpandProperty Node |
 Select-Object -ExpandProperty value |
 Sort-Object -Unique |
 Where-Object {$_ -like 'CompanyPackages*'} |
 ForEach-Object { Update-Package $_ }
     
 
```

Update
======

I take it all back, [@neilbarnwell](https://twitter.com/neilbarnwell) pointed out this lovely one liner

```
get-project -all | get-package | ?{ $_.Id -like 'Stripe*'} | Update-Package

```