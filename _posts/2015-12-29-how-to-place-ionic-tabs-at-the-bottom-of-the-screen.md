---
layout: post
title: "How to place ionic tabs at the bottom of the screen"
categories:
- 
tags:
- 


---


http://stackoverflow.com/questions/27874855/how-to-place-ionic-tabs-at-the-bottom-of-the-screen

Since the beta 14 of Ionic (http://blog.ionic.io/the-final-beta/), there are some config variables that now default to their platform values, which means that they will try to behave according to the platform that they are built on. In the case of tabs, for iOS the default is to display on bottom, and Android on top.

You can easily "hard set" the location for all platforms by setting the tabs.position function in the $ionicConfigProvider, inside your config function like this:

	MyApp.config(['$ionicConfigProvider', function($ionicConfigProvider) {

	    $ionicConfigProvider.tabs.position('bottom'); // other values: top

	}]);

You can check documentation here: http://ionicframework.com/docs/api/provider/%24ionicConfigProvider/

Hope this is helpful