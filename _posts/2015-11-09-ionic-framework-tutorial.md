---
layout: post
title: "Ionic Framework Tutorial"
categories: 手机开发
tags: ionic


---



1.  Install Ionic
2.  Starting the Node Server
3.  Creating an Ionic Application
4.  Creating the Session Service
5.  Creating the Session Controllers
6.  Creating Templates
7.  Routing

Module 1: Install Ionic
================ 

In this module, you install the [Ionic Framework](http://ionicframework.com/) and [Cordova](http://cordova.apache.org/) using npm (the Node Package Manager).

Steps
================ 

Make sure you have an up-to-date version of Node.js installed on your system. If you don't have Node.js installed, you can install it from [here](http://nodejs.org/).

Open a terminal window (Mac) or a command window (Windows), and install Cordova and Ionic:

	npm install -g cordova ionic

On a Mac, you may have to use sudo depending on your system configuration:

	sudo npm install -g cordova ionic

If you already have Cordova and Ionic installed on your computer, make sure you update to the latest version:

	npm update -g cordova ionic

or

	sudo npm update -g cordova ionic

Module 2: Starting the Node Server
================ 

In this module, you install and start a Node.js server that exposes the conference data (sessions and speakers) through a set of REST services.

Steps
================ 

Download the supporting files for this tutorial here, or clone the repository:

	git clone https://github.com/ccoenraets/ionic-tutorial

If you downloaded the zip file, unzip it anywhere on your file system.
Open a terminal window (Mac) or a command window (Windows), and navigate (cd) to the ionic-tutorial/server directory

Install the server dependencies:

	npm install

Start the server:

node server
If you get an error, make sure you don't have another server listening on port 5000.
Test the REST services. Open a browser and access the following URLs:

http://localhost:5000/sessions (for a list of conference sessions returned as a JSON document)

http://localhost:5000/sessions/1 (for information about a specific session )

Module 3: Creating an Ionic Application
================ 

In this module, you use the Ionic CLI (command line interface) to create a new application based on the sidemenu starter app.

Steps
================ 

Open a new terminal window (Mac) or a command window (Windows), and navigate (cd) to the ionic-tutorial directory

Using the Ionic CLI, create an application named conference based on the sidemenu starter app:

	ionic start conference sidemenu

Navigate to the conference folder

	cd conference

Start the application in a browser using ionic serve.

	ionic serve

NOTE: Because of cross domain policy issues (specifically when loading templates), you have to load the application from a server (using the http protocol and not the file protocol). ionic serve is a lightweight local web server with live reload.
In the application, open the side menu ("hamburger" icon in the upper left corner) and select Playlists. Select a playlist in the list to see the details view (not much to see at this point).

In the next modules, you will replace the playlists with a list of conference sessions retrieved from the server using the REST services you experimented with in the previous module.

Open the side menu again and select Login. Click the Login button to close the window (Login is not implemented in the starter app).

In the last module of this tutorial you will implement Login using Facebook.

Module 4: Creating the Session Service
================ 

In the sidemenu starter app, the playlists are hardcoded in controllers.js. In this module, you create a Session service that uses the Angular resource module (ngResource) to retrieve the conference sessions using REST services.

Steps
================ 

In the conference/www/js directory, create a file named services.js

In services.js, define a module named starter.services with a dependency on ngResource:

	angular.module('starter.services', ['ngResource'])

In that module, define a service named Session that uses the Angular resource module to provide access to the REST services at the specified endpoint:

	angular.module('starter.services', ['ngResource'])

	.factory('Session', function ($resource) {
    	return $resource('http://localhost:5000/sessions/:sessionId');
	});

In a real-life application, you would typically externalize the server parameters in a config module.
The starter.services module you just created has a dependency on the Angular resource module which is not included by default. Open index.html and add a script tag to include angular-resource.min.js (right after ionic-bundle.js):

	<script src="lib/ionic/js/angular/angular-resource.min.js"></script>

Add a script tag to include the services.js file you just created (right after app.js):

	<script src="js/services.js"></script>

Module 5: Creating the Session Controllers
================ 

AngularJS controllers act as the glue between views and services. A controller often invokes a method in a service to get data that it stores in a scope variable so that it can be displayed by the view. In this module, you create two controllers: SessionsCtrl manages the session list view, and SessionCtrl manages the session details view.

Step 1: Declare starter.services as a Dependency
================ 

The two controllers you create in this module use the Session service defined in the starter.services module. To add starter.services as a dependency to the starter.controller module:

Open conference/www/js/controllers.js

Add starter.services as a dependency to make the Session service available to the controllers:

	angular.module('starter.controllers', ['starter.services'])

Step 2: Implement the Session List Controller
================ 

In controllers.js, delete PlayListsCtrl (plural)

Replace it with a controller named SessionsCtrl that retrieves the list of conference sessions using the Session service and stores it in a scope variable named sessions:

	.controller('SessionsCtrl', function($scope, Session) {
	    $scope.sessions = Session.query();
	})

Step 3: Implement the Session Details Controller
================ 

In controllers.js, delete PlayListCtrl (singular)

Replace it with a controller named SessionCtrl that retrieves a specific session using the Session service and stores it in a scope variable named session:

	.controller('SessionCtrl', function($scope, $stateParams, Session) {
	    $scope.session = Session.get({sessionId: $stateParams.sessionId});
	});

Module 6: Creating Templates
================ 

In this module, you create two templates: sessions.html to display a list of sessions, and session.html to display the details of a particular session.

Step 1: Create the sessions template
================ 

In the conference/www/templates directory, rename playlists.html (plural) to sessions.html

Implement the template as follows:

	<ion-view view-title="Sessions">
	  <ion-content>
	    <ion-list>
	      <ion-item ng-repeat="session in sessions"
	                href="#/app/sessions/{{session.id}}">{{session.title}}</ion-item>
	    </ion-list>
	  </ion-content>
	</ion-view>

Notice the use of the ng-repeat directive to display the list of sessions

Step 2: Create the session template
================ 

Rename playlist.html (singular) to session.html

Implement the template as follows:

	<ion-view view-title="Session">
	  <ion-content>
	    <div class="list card">
	      <div class="item">
	        <h3>{{session.time}}</h3>
	        <h2>{{session.title}}</h2>
	        <p>{{session.speaker}}</p>
	      </div>
	      <div class="item item-body">
	        <p>{{session.description}}</p>
	      </div>
	      <div class="item tabs tabs-secondary tabs-icon-left">
	        <a class="tab-item">
	          <i class="icon ion-thumbsup"></i>
	          Like
	        </a>
	        <a class="tab-item">
	          <i class="icon ion-chatbox"></i>
	          Comment
	        </a>
	        <a class="tab-item">
	          <i class="icon ion-share"></i>
	          Share
	        </a>
	      </div>
	    </div>
	  </ion-content>
	</ion-view>


Module 7: Implementing Routing
================ 

In this module, you add two new routes (states) to the application: app.sessions loads the session list view, and app.session loads the session details view.

Step 1: Define the app.sessions route
================ 

Open app.js in conference/www/js

Delete the app.playlists state

Replace it with an app.sessions state defined as follows:

	.state('app.sessions', {
	  url: "/sessions",
	  views: {
	      'menuContent': {
	          templateUrl: "templates/sessions.html",
	          controller: 'SessionsCtrl'
	      }
	  }
	})

Step 2: Define the app.session route
================ 

Delete the app.single state

Replace it with an app.session state defined as follows:

	.state('app.session', {
	    url: "/sessions/:sessionId",
	    views: {
	        'menuContent': {
	          templateUrl: "templates/session.html",
	          controller: 'SessionCtrl'
	      }
	    }
	});

Step 3: Modify the default route
================ 

Modify the fallback route to default to the list of sessions (last line in app.js):

	$urlRouterProvider.otherwise('/app/sessions');

Step 4: Modify the side menu
================ 

Open menu.html in conference/www/templates

Modify the Playlists menu item as follows (modify both the item label and the href):

	<ion-item menu-close href="#/app/sessions">
	    Sessions
	</ion-item>

Step 5: Test the application
================ 

Make sure ionic serve (your local web server) is still running.

If it's running but you closed your app page in the browser, you can reload the app by loading the following URL: http://localhost:8100
If it's not running, open a command prompt, navigate (cd) to the ionic-tutorial directory and type:

	ionic serve

In the conference application, open the side menu ("Hamburger" icon in the upper left corner) and select Sessions. Select a session in the list to see the session details.



参照[ionic hello world](https://ccoenraets.github.io/ionic-tutorial/install-ionic.html)部分修改