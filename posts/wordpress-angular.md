# AngularJS + WordpressでSinglePageSite

atomita  
2014/09/27



## Applicationではありません

Single Page Application ではなく Single Page Site です  
アプリ作る機会ないですorz



## 今回の話で目指すところ
TwentyFourteenテーマを改造してSinglePage化

3行で言うと

* TwentyFourteenテーマの子テーマで
* クエリパラメータでページ全体を返すか一部を返すか切り替えて
* angular-routeを使って表示



## 必要なもの

* WordPress
* AngularJS
* angular-route
* Wordpressが動くwebサーバー



## その他、使ったもの

* angular-animate
* OpenShift Online
* CoffeeScript
* Stylus
* Yeoman + Yeopress
* Bower
* Composer



## style.styl (style.css)

### TwentyFourteenの子テーマにしたいのでtwentyfourteen を指定

```stylus
/*
Theme Name: child
Template: twentyfourteen
*/

@import url('../twentyfourteen/style.css');
```



### あとanimate用の記述を少し

```stylus
.slow
	-webkit-transition all ease 2s
	-moz-transition all ease 2s
	-o-transition all ease 2s
	transition all ease 2s

.fade
	&.ng-enter
		opacity 0

		&.ng-enter-active
			opacity 1

		&.slow
			transition-delay 2s

	&.ng-leave
		opacity 1

		&.ng-leave-active
			opacity 0
```



## レイアウトで包む
### ざっくりWordPressのhtml出力の流れ
> index.php 等
>
>> header.php
>
>> コンテンツ
>
>> footer.php

こんな感じで、全てindex.php(他)がheader.php、footer.phpを呼び出す形で組まれますが、少しやりにくいので



### ↓な感じで

レイアウトファイルが出力する流れにちょいと改造します

> layout
>
>> header.php
>
>> index.php 等のコンテンツ
>
>> footer.php



## layout

### layout/default.php

index.phpなどの出力が \$contents に入っている体  
それ以外は、 ng-view があるのと \$contents の出力を消すJSが組んであるくらい

```php
<?php
include TEMPLATEPATH . '/header.php';
?>
<div id="__static_contents">
	<?php echo $contents; ?>
</div>
<script>jQuery('#__static_contents').html('');</script>

<div ng-view id="__dynamic_contents" class="slow fade"></div>
<?php
include TEMPLATEPATH . '/footer.php';
```



### layout/content-only.php

コンテンツだけを出力するやつも作っておきます

```php
<?php
echo $contents;
```



## header, footer

こちらは通常は何も出力しないようにしちゃいます

header.php

```php
<?php
if (is_admin()){
	include TEMPLATEPATH . '/header.php';
}
```

footer.php

```php
<?php
if (is_admin()){
	include TEMPLATEPATH . '/footer.php';
}
```



## functions.php

### レイアウトの適用

自作のLayoutStyleThemeというのを使ってますが、Twig等のテンプレートエンジン使ったほうがより良いと思います  

今回はクエリパラメータ__contentsonly__が"t"の場合はコンテンツのみ出力するようにします

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use \atomita\wordpress\LayoutStyleThemeFacade as LayoutStyle;
LayoutStyle::initialize();

// レイアウト変更
if (isset($_GET['__contentsonly__']) && $_GET['__contentsonly__'] === 't'){
	LayoutStyle::layout('content-only');
}
```



### JavaScriptの読み込み

```php
wp_enqueue_script('angular', get_stylesheet_directory_uri() . "/bower_components/angular/angular.min.js", ['jquery'], '1.2');
wp_enqueue_script('angular-route', get_stylesheet_directory_uri() . "/bower_components/angular-route/angular-route.min.js", ['angular'], '1.2');
wp_enqueue_script('angular-animate', get_stylesheet_directory_uri() . "/bower_components/angular-animate/angular-animate.js", ['angular'], '1.2');
wp_enqueue_script('app', get_stylesheet_directory_uri() . "/assets/scripts/app.js", ['angular-route', 'angular-animate'], '1.2');
global $wp_scripts;
$wp_scripts->add_data('app', 'data', 'var config = {};');
```



### baseタグの出力（おまじない）

```php
add_action('wp_head', function()
{
	$home = home_url('/');
	echo "<base href='{$home}' />\n";
});
```



### bodyのidとかclassの設定

```php
add_filter('body_class', function($classes)
{
	echo 'id="ng-app" ng-app="app" xmlns:ng="http://angularjs.org" '; // 邪道!!
	$classes[] = 'ng-app:app';
	return $classes;
});
```



## app.coffee (app.js)

### angular.module

```coffee
class @App

	constructor: (@angular, @config, @jQuery)->
		@_config.$inject = ["$locationProvider", "$compileProvider", "$routeProvider", "$logProvider", "$animateProvider"]
		@_run.$inject = ["$rootScope", "$location", "$window", "$log", "$timeout", "$browser"]

		@app = @angular.module('app', ['ngRoute', 'ngAnimate'])
		@app.config(@_config)
		@app.run(@_run)
```



### cofig

html5モードである必要があります

```coffee
	_config: ($locationProvider, $compileProvider, $routeProvider, @$logProvider, $animateProvider)=>
		$locationProvider.html5Mode(true).hashPrefix('!')

		@_route($routeProvider)
		@_animation($animateProvider)
```



### run

```coffee
	_run: (@$rootScope, @$location, @$window, @$log, @$timeout, @$browser)=>
		@config.baseHref = $browser.baseHref()
```



### route (1)

```coffee
	_route: ($routeProvider)=>
		$routeProvider
		# admin
		.when(':base*?/wp-admin/:__all__*?',
			controller: ($window, $routeParams)=>
					base = @config.baseHref + (if $routeParams.base then "#{$routeParams.base}/" else "")
					all = if $routeParams.__all__ then $routeParams.__all__ else ""
					$window.location = "#{base}wp-admin/#{all}"
			template: ""
		)
```



### route (2)

```coffee
		# all
		.when('/:__all__*?',
			controller: ($location, $routeParams, $scope)=>
				# することなし
			templateUrl: ($routeParams)=>
				@$log.debug("Application.templateUrl", arguments)
				all = if $routeParams.__all__? then $routeParams.__all__ else ""
				delete $routeParams.__all__
				query = @jQuery.param(angular.extend({"__contentsonly__": "t"}, $routeParams))
				"#{@config.baseHref}#{all}?#{query}"
		)
		.otherwise(
			redirectTo: '/'
		)
```



### animation

cssでやるので特に書くことなし

```coffee
	_animation: ($animateProvider)->
```	



### インスタンス化

```coffee
app = new App(angular, config, jQuery)
```



## といった形で構築したのが

### http://awsmokumoku-at0m.rhcloud.com/ 

## おわり。

### ご清聴ありがとうございました
