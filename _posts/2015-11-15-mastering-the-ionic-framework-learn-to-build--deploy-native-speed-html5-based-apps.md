---
layout: post
title: "ionic教程 音乐app"
categories: 手机开发
tags: ionic 
---

准备
================ 
1.  需要懂一点[javascript](http://sivers.org/learn-js)
2.  熟悉[AngularJS](https://thinkster.io/a-better-way-to-learn-angularjs/)
3.  了解[REST](http://www.restapitutorial.com/lessons/whatisrest.html)和[CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete)
3.  安装[Node.js](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)
4.  安装[Ionic CLI](http://ionicframework.com/docs/cli/install.html)


开始
================ 
我们可以[开始一个ionic项目](http://ionicframework.com/docs/cli/start.html)，运行

	ionic start myApp blank

该命令会新建一个空的ionic项目。

为了方便演示，今天我们直接clone完整的[课程项目](https://github.com/EricSimons/ionic-course)

	git clone https://github.com/EricSimons/ionic-course

进入项目目录，运行安装

	npm install

运行web启动，修改文件会自动加载

	ionic serve

项目入口文件为www/index.html，基本上所有的开发工作都是在www目录下。

定义AngularJS的应用名称为 songhop

	<body ng-app="songhop">

所有跳转的view都加载到 ion-nav-view 下面

	  <body ng-app="songhop">
	    <!--
	      The nav bar that will be updated as we navigate between views.
	    -->
	    <ion-nav-bar class="bar-stable">
	      <ion-nav-back-button>
	      </ion-nav-back-button>
	    </ion-nav-bar>
	    <!--
	      The views will be rendered in the <ion-nav-view> directive below
	      Templates are in the /templates folder (but you could also
	      have templates inline in this html file if you'd like).
	    -->
	    <ion-nav-view></ion-nav-view>
	  </body>

项目依靠AngularUI Router进行路由控制，决定页面跳转。打开app.js,查看应用config内的state定义。
一共定义了三个state，名为tab的abstract state 显示templates/tabs.html,其他两个state显示对应
的两个页面。

	 .state('tab.discover', {
	    url: '/discover',
	    views: {
	      'tab-discover': {
	        templateUrl: 'templates/discover.html',
	        controller: 'DiscoverCtrl'
	      }
	    }
	  })

查看templates/tabs.html文件中，在ion-tab中插入了两个 ion-nav-view， 用于设置tab跳转的页面，
这样使得tab和和index页面一样，拥有自己的navigation history。

下面来看看Discover 和 Favorites 页面

templates/discover.html内有一个 Card Image，包含歌曲信息和 favorite/skip 按钮

templates/favorites.html 内包含了favorite song的一个ion-list列表

我们可以看到页面都包含目标hearder bar，通过导航跳转的页面都包含在 ion-view内

所有内容都包含在ion-content 内
  
    ico-nav-bar （hearder bar）

    ico-nav-view 
    	ico-tabs （内包含三个tabs）
    		ico-nav-view  （tab-discover）
    			ico-view
    				ico-content （内容放在这）
    		ico-nav-view  （tab-favorites）
    			ico-view  
    				ico-content   （内容放在这）




补充模块功能
================ 
我们来到Discover模块。

其中包含歌曲信息和favorite/skip 按钮。通常我们从服务器读取数据，本次为简化流程，我们直接直接在

controller内赋值。

	.controller('DiscoverCtrl', function($scope) {
	  // our first three songs
	  $scope.songs = [
	     {
	        "title":"Stealing Cinderella",
	        "artist":"Chuck Wicks",
	        "image_small":"https://i.scdn.co/image/d1f58701179fe768cff26a77a46c56f291343d68",
	        "image_large":"https://i.scdn.co/image/9ce5ea93acd3048312978d1eb5f6d297ff93375d"
	     },
	     {
	        "title":"Venom - Original Mix",
	        "artist":"Ziggy",
	        "image_small":"https://i.scdn.co/image/1a4ba26961c4606c316e10d5d3d20b736e3e7d27",
	        "image_large":"https://i.scdn.co/image/91a396948e8fc2cf170c781c93dd08b866812f3a"
	     },
	     {
	        "title":"Do It",
	        "artist":"Rootkit",
	        "image_small":"https://i.scdn.co/image/398df9a33a6019c0e95e3be05fbaf19be0e91138",
	        "image_large":"https://i.scdn.co/image/4e47ee3f6214fabbbed2092a21e62ee2a830058a"
	     }
	  ];

因为**的原因，上面代码变成了

	.controller('DiscoverCtrl', function($scope) {
	  // our first three songs
	  $scope.songs = [
	     {
	        "title":"Stealing Cinderella",
	        "artist":"Chuck Wicks",
	        "image_small":"img/attach/11.jpeg",
	        "image_large":"img/attach/12.jpeg"
	     },
	     {
	        "title":"Venom - Original Mix",
	        "artist":"Ziggy",
	        "image_small":"img/attach/21.jpeg",
	        "image_large":"img/attach/22.jpeg"
	     },
	     {
	        "title":"Do It",
	        "artist":"Rootkit",
	        "image_small":"img/attach/31.jpeg",
	        "image_large":"img/attach/32.jpeg"
	     }
	  ];

用户访问discover页面的时候，需要指定当前歌曲currentSong

	$scope.currentSong = angular.copy($scope.songs[0]);

将当前歌曲信息显示到discover.html 

      <div class="item item-image">
        <img ng-src="{ { currentSong.image_large }}">
      </div>
      <div class="item">
        <h2>{ { currentSong.title }}</h2>
        <p>{ { currentSong.artist }}</p>
      </div>

现在我们来处理 skip or favorite， 处理新的song

新建sendFeedback()方法（传入bool参数）处理skip or favorite两个操作，分别跳过和收藏。

	  // fired when we favorite / skip a song.
	  $scope.sendFeedback = function (bool) {

	    // set the current song to one of our three songs
	    var randomSong = Math.round(Math.random() * ($scope.songs.length - 1));

	    // update current song in scope
	    $scope.currentSong = angular.copy($scope.songs[randomSong]);

	  }

在页面中调用sendFeedback()方法

      <div class="item tabs tabs-secondary tabs-icon-left">
        <a class="tab-item" ng-click="sendFeedback(false)">
          <i class="icon ion-close"></i>
          Skip
        </a>
        <a class="tab-item" ng-click="sendFeedback(true)">
          <i class="icon ion-heart"></i>
          Favorite
        </a>
      </div>

现在点击skip 或者 favorite 都会切换一首song


使用动画
================ 
下面我们用动画来切换song，查看style.css，里面有定义动画的样式。

修改sendFeedback()

	  $scope.sendFeedback = function (bool) {

	    // set variable for the correct animation sequence
	    $scope.currentSong.rated = bool; //该参数用于区分skip和favorite
	    $scope.currentSong.hide = true;  //隐藏

	    $timeout(function() {
	      // $timeout to allow animation to complete before changing to next song
	      // set the current song to one of our three songs
	      var randomSong = Math.round(Math.random() * ($scope.songs.length - 1));

	      // update current song in scope
	      $scope.currentSong = angular.copy($scope.songs[randomSong]);

	    }, 250);
	  }

在函数头添加参数:

	.controller('DiscoverCtrl', function($scope, $timeout) {


在discover.html 添加动画class

    <div class="list card current-song"
          ng-class="{skipped: currentSong.rated == false,
                     favorited: currentSong.rated == true}"
          ng-show="currentSong && !currentSong.hide">

试试动画是否已经生效


添加和删除favorite songs
================ 

现在点击favorite，只是切换songs，并不会存储。

新建一个service来存储songs

在services.js 添加

	angular.module('songhop.services', [])
	.factory('User', function() {

	  var o = {
	    favorites: []
	  }

	  return o;
	});

并添加一个增加songs的方法

	  o.addSongToFavorites = function(song) {
	    // make sure there's a song to add
	    if (!song) return false;

	    // add to favorites array
	    o.favorites.unshift(song);
	  }

更新favorite显示，获取favorites

	.controller('FavoritesCtrl', function($scope, User) {
	  // get the list of our favorites from the user service
	  $scope.favorites = User.favorites;
	})

显示到页面favorites.html

  	<ion-item ng-repeat="song in favorites" class="item-avatar">
        <img ng-src="{{ song.image_small }}">
        <h2>{{ song.title }}</h2>
        <p>{{ song.artist }}</p>
      </ion-item>

在discover函数头添加User

	.controller('DiscoverCtrl', function($scope, $timeout, User) {

并在sendFeedback()调用song添加

    // first, add to favorites if they favorited
    if (bool) User.addSongToFavorites($scope.currentSong);

添加歌曲，并查看favorites页面，可以看到刚刚添加的歌曲了（可以重复添加）

现在我们再来关注一下删除

在services添加删除方法

	  o.removeSongFromFavorites = function(song, index) {
	    // make sure there's a song to add
	    if (!song) return false;

	    // add to favorites array
	    o.favorites.splice(index, 1);
	  }

在favorites controllers 添加删除方法

	 $scope.removeSong = function(song, index) {
	    User.removeSongFromFavorites(song, index);
	  }

现在我们在页面favorites.html实现滑动删除

	   <ion-list>
	      <ion-item ng-repeat="song in favorites" class="item-avatar">
	        <img ng-src="{{ song.image_small }}">
	        <h2>{{ song.title }}</h2>
	        <p>{{ song.artist }}</p>

	        <ion-option-button class="button-assertive"
	                       ng-click="removeSong(song, $index)">
	          <i class="ion-minus-circled"></i>
	        </ion-option-button>
	      </ion-item>
	    </ion-list>

查看是否已经生效

基础功能已经完成了，下面演示与服务端交互数据

添加server
================
下面使用django创建一个server应用。
访问http://127.0.0.1:8000/recommendations/
返回

	[{
	image_large: "https://i.scdn.co/image/aae42d9dee324f885a0cce92a383cdd068ec33d9",
	song_id: "54dd0bc8a57fa90c00770580",
	open_url: "https://open.spotify.com/track/7ejDWW73B2pinnUSV4D6DB",
	image_medium: "https://i.scdn.co/image/952e954c06a504d85a7a8381e1f1078e54789a43",
	title: "Keep Them Kisses Comin'",
	artist: "Craig Campbell",
	image_small: "https://i.scdn.co/image/8482b9f065dbe7e80215f7d0a8dcfb5796dc4653",
	preview_url: "https://p.scdn.co/mp3-preview/00ecd34c3d0af9330420f3e3031775bcf7122fa9"
	},
	{
	image_large: "https://i.scdn.co/image/910e7824f90744ed0e605677da22722efb9d0203",
	song_id: "54dd0bc8a57fa90c0077061e",
	open_url: "https://open.spotify.com/track/65xGo48KiS0ONsGo9KrI66",
	image_medium: "https://i.scdn.co/image/cd62c3838e486786ebbab2f901c3cccb5a180db6",
	title: "Confident",
	artist: "Justin Bieber",
	image_small: "https://i.scdn.co/image/315d30bfa460a9903b27e0119c362bec60186834",
	preview_url: "https://p.scdn.co/mp3-preview/f60b82b468484ce721f4bcf56c1ad3746acd00fc"
	},
	{
	image_large: "https://i.scdn.co/image/0e84cacbfd46e09fe96d442fcb835520e3a8b1a0",
	song_id: "54dd0bc8a57fa90c0077053b",
	open_url: "https://open.spotify.com/track/191DgH31PULWyfGuTUeOtP",
	image_medium: "https://i.scdn.co/image/b3481286c31b4ace7aca1b2bf253605706a8b79e",
	title: "Into The Blue",
	artist: "Kylie Minogue",
	image_small: "https://i.scdn.co/image/44385c6c79257ec77710bf5f7ad4c2ed94652ec9",
	preview_url: "https://p.scdn.co/mp3-preview/e28a569cf7c2c37af8de2181570bcd4b83f5d0cb"
	},
	{
	image_large: "https://i.scdn.co/image/5ba8be17615f3bcd1519e63c54892fba4c698d6f",
	song_id: "54dd0bc8a57fa90c0077055a",
	open_url: "https://open.spotify.com/track/0DJZtb8ghBDpBJ2qXxlPBK",
	image_medium: "https://i.scdn.co/image/1556ff85180ff3cf207f18378a5e3951c82ab5a5",
	title: "No Rest For The Wicked",
	artist: "Lykke Li",
	image_small: "https://i.scdn.co/image/f12c39b007999c1a5368d5a59a1ffeb1396d734d",
	preview_url: "https://p.scdn.co/mp3-preview/aba9b9d69f6dfe077264f008fcb4aa906ff096c4"
	},
	{
	image_large: "https://i.scdn.co/image/9ae687579451fa030d0655b328c08c7fb7a40643",
	song_id: "54dd0bc8a57fa90c007705de",
	open_url: "https://open.spotify.com/track/6FXwtxAFWLrEsYQjcqBpnh",
	image_medium: "https://i.scdn.co/image/38bd653256d3138e145f63cd3cf5bb891b9021bf",
	title: "With A Heavy Heart (I Regret To Inform You)",
	artist: "Does It Offend You, Yeah?",
	image_small: "https://i.scdn.co/image/3f319005a04e012ed7ac822f674a5b749ac50416",
	preview_url: "https://p.scdn.co/mp3-preview/1f0ecb3e5a335e6030e39b7e07cd1b99949ae924"
	},
	{
	image_large: "https://i.scdn.co/image/13723a1297917911b8242f17a67f9f48f2f99d90",
	song_id: "54dd0bc8a57fa90c00770604",
	open_url: "https://open.spotify.com/track/0wpbHnOW0zVUtV10LSj9c9",
	image_medium: "https://i.scdn.co/image/3464917116ef04e7e8c9c5d7fedf911be8505225",
	title: "Down On Me",
	artist: "Jeremih",
	image_small: "https://i.scdn.co/image/c1a82033d85f518e61a0624c96f0fc0ddfa2f2a0",
	preview_url: "https://p.scdn.co/mp3-preview/f71786bdb8dd7f2ccd7505e919cb72ea4bb0df10"
	},
	{
	image_large: "https://i.scdn.co/image/9e6e5d85156c88b1edc4f5a305ae2f64b26d0f5f",
	song_id: "54dd0bc8a57fa90c007705f6",
	open_url: "https://open.spotify.com/track/03kFFDai1VaBsSWny2092v",
	image_medium: "https://i.scdn.co/image/32390cc5c22a9e10d9275599998ccad10669dbec",
	title: "Michael",
	artist: "The Dreamers",
	image_small: "https://i.scdn.co/image/b233457f93d9d94b99141af79f0a978b3d663b74",
	preview_url: "https://p.scdn.co/mp3-preview/4a3e603ec76c1a838e3174b932d25abf9b2dac3c"
	},
	{
	image_large: "https://i.scdn.co/image/9a6cc907ddd8845d1f3eca7b4c3a51e1077b2b1b",
	song_id: "54dd0bc8a57fa90c007705c6",
	open_url: "https://open.spotify.com/track/4jCJDQiiLMTh3ix6dXqfvo",
	image_medium: "https://i.scdn.co/image/4be4df2e3b65bf6ed7d5fe13cafc857f3359ac6b",
	title: "Levitate",
	artist: "Hadouken!",
	image_small: "https://i.scdn.co/image/6e3460dd7888a3ea5c5d78862a986a4570d79788",
	preview_url: "https://p.scdn.co/mp3-preview/dbce66f874c462f4755e1d4083542b011415a88d"
	},
	{
	image_large: "https://i.scdn.co/image/a780f1b9bb4b49bd27d3365d3ca0e191163fefe9",
	song_id: "54dd0bc8a57fa90c0077049e",
	open_url: "https://open.spotify.com/track/0UVkeeh91kqgf3JuWVrBuT",
	image_medium: "https://i.scdn.co/image/cf6c2a8a173934948f384c217a6d3a3082027d9c",
	title: "Let It Rain",
	artist: "David Nail",
	image_small: "https://i.scdn.co/image/c47c4e16858d5377ce2ff59130b2247562e2c303",
	preview_url: "https://p.scdn.co/mp3-preview/6224900985467761fd7ecbb04696f3e10bdc0ff1"
	},
	{
	image_large: "https://i.scdn.co/image/0044e40a310cdb7883ef4cf85908dc570d6c2de3",
	song_id: "54dd0bc8a57fa90c00770509",
	open_url: "https://open.spotify.com/track/59b7H63ATlrSdfoEQHgDdY",
	image_medium: "https://i.scdn.co/image/0b37a52c1f2bf850d1467c194119d82631b64a8a",
	title: "Be Without You",
	artist: "Mary J. Blige",
	image_small: "https://i.scdn.co/image/a0a5e8a6bff2224b806bae27d511555c75d4e1e5",
	preview_url: "https://p.scdn.co/mp3-preview/84cd8d177e8af2600cc49792ec4c457892175879"
	}]

现在使用改service会出现跨域调用的问题$http No ‘Access-Control-Allow-Origin’ problem 

下面列举三个解决CORS问题的方法：

1.  [在ionic设定访问代理](http://ionicinaction.com/blog/how-to-fix-cors-issues-revisited/)，该方法只适合开发环境使用

2.  使用[jsonp](http://my.oschina.net/blogshi/blog/303758)

3.  在server端设定cors头给返回值。

本例采用第三种方法，使用[django-cors-headers](https://github.com/ottoyiu/django-cors-headers)

下面回到我们的song手机app项目。

在app.js中创建一个server

	.constant('SERVER', {
	   url: 'http://192.168.1.100:8000'
	});

在services.js中创建factory，从server返回一个数组。

	.factory('Recommendations', function($http, SERVER) {
	  var o = {
	    queue: []
	  };

	  return o;
	})

Recommendations中的数组通过http://SERVER-URL/recommendations来获取。

创建getNextSongs()方法

	.factory('Recommendations', function($http, SERVER) {
	  console.log('SERVER', SERVER.url + '/recommendations');

	  var o = {
	    queue: []
	  };

	  o.getNextSongs = function() {
	    return $http({
	      method: 'GET',
	      url: SERVER.url + '/recommendations'
	    }).success(function(data){
	      console.log('Got some data: ', data);
	      // merge data into the queue

	      o.queue = o.queue.concat(data);
	    });
	  };

	  return o;
	})


app启动初始化需要设定第一个song，在DiscoverCtrl调用Recommendations.getNextSongs()

即可，该方法设定queue[0]为当前song。

	.controller('DiscoverCtrl', function($scope, $timeout, User, Recommendations) {
		 // get our first songs
	  Recommendations.getNextSongs()
	    .then(function(){
	      $scope.currentSong = Recommendations.queue[0];
	    });

接下来处理skip和favorite操作

在Recommendations添加nextSong方法

	 o.nextSong = function() {
	    // pop the index 0 off
	    o.queue.shift();

	    // low on the queue? lets fill it up
	    if (o.queue.length <= 3) {
	      o.getNextSongs();
	    }

	  }

更新sendFeedback方法，调用Recommendations.nextSong()

    // prepare the next song
    Recommendations.nextSong();

    $timeout(function() {
      // $timeout to allow animation to complete
      $scope.currentSong = Recommendations.queue[0];
    }, 250);

使用预加载加速图片加载，在Discover中添加nextAlbumImg()方法

	 // used for retrieving the next album image.
	  // if there isn't an album image available next, return empty string.
	  $scope.nextAlbumImg = function() {
	    if (Recommendations.queue.length > 1) {
	      return Recommendations.queue[1].image_large;
	    }

	    return '';
	  }

在 discover.html中添加div

 	<div class="img-lookahead">
      <img ng-src="{ { nextAlbumImg() }}" />
    </div>

这样就可以了吗？是的，就这样简单

听歌
================ 

在我们返回的数据中，preview_url字段，预听歌曲的地址，我们可以直接用浏览器访问。

在Recommendations中，我们创建一个方法来播放歌曲。

Recommendations的顶部，创建media变量存储歌曲。

	var media;

创建两个方法，播放和暂停

	  o.playCurrentSong = function() {
	    var defer = $q.defer();

	    // play the current song's preview
	    media = new Audio(o.queue[0].preview_url);

	    // when song loaded, resolve the promise to let controller know.
	    media.addEventListener("loadeddata", function() {
	      defer.resolve();
	    });

	    media.play();

	    return defer.promise;
	  }

	  // used when switching to favorites tab
	  o.haltAudio = function() {
	    if (media) media.pause();
	  }

记得吧$q加到传入参数中。

修改nextSong()，调用它时停止播放当前歌曲。

	  o.nextSong = function() {
	    // pop the index 0 off
	    o.queue.shift();

	    // end the song
	    o.haltAudio();

	    // low on the queue? let's fill it up
	    if (o.queue.length <= 3) {
	      o.getNextSongs();
	    }
	  }

修改DiscoverCtrl，app加载时候开始播放 Recommendations.playCurrentSong()

	  Recommendations.getNextSongs()
	    .then(function(){
	      $scope.currentSong = Recommendations.queue[0];
	      Recommendations.playCurrentSong();
	    });

最后，修改sendFeedback()，切换歌曲的时候调用Recommendations.playCurrentSong()

    $timeout(function() {
      // $timeout to allow animation to complete
      $scope.currentSong = Recommendations.queue[0];
    }, 250);

    Recommendations.playCurrentSong();

现在我们的app已经可以顺利播放app了，还有一个问题

在切换到favorite选项时，歌曲没有停止，

在切换的时候，调用haltAudio()，返回的时候继续播放。

修改TabsCtrl，添加enteringFavorites

	.controller('TabsCtrl', function($scope, Recommendations) {
	  // stop audio when going to favorites page
	  $scope.enteringFavorites = function() {
	    Recommendations.haltAudio();
	  }

	});

修改tabs.html，

	<ion-tab title="Favorites" icon-off="ion-music-note" icon-on="ion-music-note" on-select="enteringFavorites()" href="#/tab/favorites">

跳转到favorite时候，调用暂停，返回的时候继续播放。

返回时，我们可以调用Recommendations.playCurrentSong()播放。

但是这里又有一个问题，如果返回的时候queue为空怎么办？

下面我们修改代码来处理这个问题

在Recommendations中创建init方法，如果queue时，获取下一首

	  o.init = function() {
	    if (o.queue.length === 0) {
	      // if there's nothing in the queue, fill it.
	      // this also means that this is the first call of init.
	      return o.getNextSongs();

	    } else {
	      // otherwise, play the current song
	      return o.playCurrentSong();
	    }
	  }

DiscoverCtrl中，把getNextSongs() 改为 init()

	  Recommendations.init()
	    .then(function(){
	      $scope.currentSong = Recommendations.queue[0];
	      Recommendations.playCurrentSong();
	    });

在TabsCtrl中创建Recommendations.init()方法，在离开favorite时调用

	  $scope.leavingFavorites = function() {
	    Recommendations.init();
	  }

调用该方法

	<ion-tab title="Favorites" icon-off="ion-music-note" icon-on="ion-music-note" on-select="enteringFavorites()" on-deselect="leavingFavorites()" href="#/tab/favorites">

试一试吧。


优化ui
================
根据网络状况，app内容加载可能会有延迟，下面我们为等待加载添加一个等待图标。

我们使用 $ionicLoading 来实现这个效果。

修改DiscoverCtrl方法，添加参数$ionicLoading

添加显示和隐藏$ionicLoading的方法，并且调用它，只在初始化的时候调用。

	.controller('DiscoverCtrl', function($scope, $ionicLoading, $timeout, User, Recommendations) {
	  // helper functions for loading
	  var showLoading = function() {
	    $ionicLoading.show({
	      template: '<i class="ion-loading-c"></i>',
	      noBackdrop: true
	    });
	  }

	  var hideLoading = function() {
	    $ionicLoading.hide();
	  }

	  // set loading to true first time while we retrieve songs from server.
	  showLoading();

在DiscoverCtrl的Recommendations.init()加载完成的时候调用 hideLoading()，歌曲开始播放后，隐藏加载项。

	  Recommendations.init()
	    .then(function(){
	      $scope.currentSong = Recommendations.queue[0];
	      Recommendations.playCurrentSong();
	      hideLoading();
	    });




Adding badges to the tab bar
================ 

Creating and persisting user data
================ 

Some Ionic challenges for those hungry for more
================ 

在android和ios中调试
================ 
ionic platform ios android

ionic plugin add com.ionic.keyboard

ionic plugin add org.apache.cordova.inappbrowser

ionic plugin add cordova-plugin-whitelist

cordova plugin add org.apache.cordova.geolocation

cordova plugin add org.apache.cordova.statusbar

cordova plugin add org.apache.cordova.device

ionic run android

[白名单](https://www.sencha.com/forum/showthread.php?301494-cordova-No-Content-Security-Policy-meta-tag-found)

[content-security-policy](http://stackoverflow.com/questions/29614858/how-to-configure-cordova-android-4-0-with-white-list)

[cordova-plugin-whitelist](https://github.com/apache/cordova-plugin-whitelist)



D/SystemWebChromeClient( 2047): file:///android_asset/www/plugins/cordova-plugin-whitelist/whitelist.js: Line 25 : No Content-Security-Policy meta tag found. Please add one when using the cordova-plugin-whitelist plugin.

W/Web Console( 2047): No Content-Security-Policy meta tag found. Please add one when using the cordova-plugin-whitelist plugin. at file:///android_asset/www/plugins/cordova-plugin-whitelist/whitelist.js:25




ionic run android


[chrome 远程调试 android](https://developers.google.com/web/tools/chrome-devtools/debug/remote-debugging/remote-debugging)

================ 

================ 

================ 


参照[原文](https://thinkster.io/ionic-framework-tutorial)修改