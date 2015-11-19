---
layout: post
title: "Ionic requests return 404 only on android, in Chrome it works fine"
categories:
tags:


---
 The thing is that there were some major changes in Cordova 4.0.0:

Major Changes [...] - Whitelist functionality is now provided via plugin (CB-7747) The whitelist has been enhanced to be more secure and configurable Setting of Content-Security-Policy is now supported by the framework (see details in plugin readme) You will need to add the new cordova-plugin-whitelist plugin Legacy whitelist behaviour is still available via plugin (although not recommended).
So I installed the Cordova Whitelist plugin. And added

<allow-navigation href="http://*/*" />
in my config.xml file.

  http://stackoverflow.com/questions/29826971/ionic-requests-return-404-only-on-android-in-chrome-it-works-fine

ionic plugin add cordova-plugin-whitelist

  <plugin name="cordova-plugin-whitelist" version="1" />
  <allow-intent href="http://192.168.1.108:3000/*" />
  <meta http-equiv="Content-Security-Policy" content="default-src *; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' 'unsafe-eval'"/>

