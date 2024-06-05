---
layout: post
title: "angularjs performance tweaks"
date: 2014-12-30 10:48:57 +0000
comments: true
redirect_from: '/blog/2014/12/30/angularjs-performance-tweaks/'
categories: angularjs
---
# Performance tips for angularjs #
Two way data binding using AngularJS is pretty sweet, but it comes at a cost. When dealing with complex data structures or large lists things can get very slow very quickly. Here are some simple things you can check to give your site the performance boost it needs. 

## Watcher count ##

A good rule of thumb is to keep the number of watchers as low as possible. In a nutshell when you tell angular to watch something it will keep checking it to see if it has changed. 
Whenever you do an expression or use a directive that takes an expression (e.g. ng-src="expression") in your html template or a $scope.$watch in your controller code Angular adds a watcher to the [digest cycle](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$digest).

Personally I use [ng-stats](https://github.com/kentcdodds/ng-stats) to keep an eye on things. 

## ng-if vs ng-show ##
The difference between ng-if and ng-show is that ng-if actually **removes or recreates** the element where as ng-show will, obviously, just hide or show the element. The intersting thing about this is that if you have something that is hidden any watchers you have will still be updated even though you are not presenting the information to the user. 

So for example consider the following:

``` html
<div ng-if="showexpression">
    <span ng-bind="expression"></div>
</div>  

<div ng-show="showexpression">
    <span ng-bind="expression"></div>
</div>  
```

They will have the same amount of watchers if the showexpression evaluates to true. However, if the showexpression evaluates to false in the ng-if then there will be less watchers because it is not rendered onto the DOM.

## Prefer simple expressions ##

This is not usually a big issue but try and make sure all your expressions are properties rather than functions. So favor things like:

``` html
<div ng-if="model.show">
    <span ng-bind="expression"></div>
</div>  
```
Rather than:

``` html
<div ng-if="model.showFunction()">
    <span ng-bind="expression"></div>
</div>  
```
showFunction will be called every time there is a digest cycle which can cause performance pains.

## One time binding ##
There are many scenarios where all you are doing is displaying data and don't need two-way data binding. This is where [bind once](https://github.com/Pasvaz/bindonce) comes in. It allows you to specify what to bind to and when to watch for changes on the thing that you have bound to. 

For example:

``` html
<div bindonce="Person" bo-title="Person.title">
    <span bo-text="Person.firstname"></span>
    <span bo-text="Person.lastname"></span>
    <img bo-src="Person.picture" bo-alt="Person.title">
    <p bo-class="{'fancy':Person.isNice}" bo-html="Person.story"></p>
</div>
```
Basically there's only one watcher on Person. When that changes all the other bo-* directives will fire. This is in contrast to having a watcher on every single expression. 