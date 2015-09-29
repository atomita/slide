# jspm workshop



## 使うもの

- node.js
- npm
- jspm
- gulp
  - webserver
  - stylus
- wisp
  - gulpfile用
  - gulpfileの内容をコピペしない場合は不要です



## 準備



### node.js 環境構築

[node.jsのdownload page](https://nodejs.org/en/download/)からpackageをdownloadしてinstallするか  
[anyenvで開発環境を整える](http://qiita.com/luckypool/items/f1e756e9d3e9786ad9ea)を参考にanyenvを使ってndenvをinstallし、ndenvを使ってnode.jsをinstallしてください



### 作業ディレクトリ構造

作業ディレクトリ内に、src/assets/styles,web/assets/scriptsディレクトリを作成しておいてください  
以降の手順は全てこの作業ディレクトリで行うものとします

```tree
.
├── src
│   └── assets
│       └── styles
└── web
    └── assets
        └── scripts
```

 note: 後述の手順で、./node_modules, web/jspm_packagesディレクトリ等が作成されます  
mac/linuxなら`mkdir -p {src,web}/assets/{styles,scripts}`



### 作業ディレクトリのnpm初期処理

```sh
npm init
```

全てEnterで進めて問題ありません

 note: 実際に現場で使う際は必要な設定を施してください



### npm packages install

必要なpackageをinstallします

```sh
npm install --save-dev jspm gulp gulp-load-plugins \
                       gulp-webserver gulp-notify \
                       gulp-plumber gulp-stylus \
                       stylus nib wisp
```



### package.json 編集

"scripts"にgulpとjspmを追加してください

```diff
  "scripts": {
-    "test": "echo \"Error: no test specified\" && exit 1"
+    "test": "echo \"Error: no test specified\" && exit 1",
+    "gulp": "gulp",
+    "jspm": "jspm",
  },
```



### jspm の初期処理

```sh
npm run jspm init
```

上記を実行し対話的に設定をしていきます

```
Would you like jspm to prefix the jspm package.json properties under jspm? [yes]:
Enter server baseURL (public folder path) [./]:web/
Enter jspm packages folder [web/jspm_packages]:
Enter config file path [web/config.js]:
Configuration file web/config.js doesn't exist, create it? [yes]:
Enter client baseURL (public folder URL) [/]:
Do you wish to use a transpiler? [yes]:
Which ES6 transpiler would you like to use, Babel, TypeScript or Traceur? [traceur]:
```

今回は`Enter server baseURL (public folder path) [./]:`に対して`web/`を設定してください  
ほかは全て未入力でEnterしてください



### gulpfile

以下の内容を ./gulpfile.wisp として保存してください

```clojure
(defmacro gulp-line "" [& commandline] (let [symbol-value (fn [symbl] (if (symbol? symbl) (get symbl :name) "")) chunk (fn [commandline] (loop [backs commandline front nil] (if (empty? backs) (list front) (let [fst (first backs) fst-val (symbol-value fst) fst* (if (dictionary? fst) [fst] fst) ] (if (or (= fst-val "|") (= fst-val ">")) (if (symbol? front) (list (list front) backs) (list front backs)) (recur (rest backs) (if front (cons front fst*) fst*))))))) chunked (chunk commandline) pipe (fn [command form] `(.pipe ~form (~@command))) dest (fn [command form] `(.pipe ~form (gulp.dest ~command))) firsts ((fn [commandline] (let [fst (first commandline) fst-val (symbol-value fst) rst (rest commandline) ] (if (and fst-val (= "gulp." (fst-val.slice 0 5))) commandline (if (= (get fst-val 0) ".") `(~fst gulp ~rst) (do (set! (aget (first commandline) :name) (+ "." fst-val)) `(~fst gulp ~rst) )))) ) (first chunked)) ] (loop [commandline (second chunked) form firsts] (if commandline (let [fname (symbol-value (first commandline)) chunked (chunk (rest commandline)) segment (first chunked) command (if (= fname "|") (pipe segment form) (dest segment form)) ] (recur (second chunked) command)) form ))))

(ns app.tasks
  "tasks"
  (:require [gulp]
            [gulp-load-plugins]
            [stylus]
            [nib]
            ))

(let [
      plugins (gulp-load-plugins)
      no-convert "!./**/{node_modules|jspm_packages}/**"
      src {
           :stylus ["src/**/*.styl{,us}" "!./**/_*.styl{,us}" no-convert]
           }
      dist "web/"
      ]

  (gulp.task :default [:stylus :watch])

  (gulp.task
   :watch (fn [] (do
                   (gulp.watch (.-stylus src) [:stylus])
                   )))

  (gulp.task
   :serve (fn [] (gulp-line
                  src [dist]
                  | plugins.webserver {:livereload true, :directoryListing false}
                  )))

  (gulp.task
   :stylus (fn [] (gulp-line
                   src (.-stylus src)
                   | plugins.plumber {:errorHandler
                                      (plugins.notify.onError
                                       {:title "task: stylus"
                                        :message "Error: <%= error.message %>"})
                                      }
                   | plugins.stylus {:define {:url (stylus.resolver)}
                                     "resolve url" true
                                     :use [(nib)]
                                     :import :nib
                                     }
                   > dist
                   )))

  )
```



### web serverの起動

gulp-webserverを使ってweb serverを起動します

```sh
npm run gulp serve
```



## Hello World



### web/index.htmlの用意

以下をコピペしてweb/index.htmlを作成してください

```
<!doctype html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>jspm workshop</title>

		<script src="jspm_packages/system.js"></script>
		<script src="config.js"></script>
		<script>
		System.import('assets/scripts/main.js');
		</script>
	</head>
	<body>
		<header>
			<h1>jspm workshop</h1>
		</header>
		<section id="contents-container">
		</section>
		<footer>
		</footer>
	</body>
</html>
```



### web/assets/scripts/main.jsの用意

以下をコピペしてweb/assets/scripts/main.jsを作成してください

```js
var contents = document.getElementById("contents-container");
contents.innerHTML = "Hello World!!";
```



### 動作確認

<http://localhost:8000> をブラウザで開いてみてください



## jQueryを使ってみる



### jQueryをinstall

```sh
npm run jspm install jquery
```

web/jspm_packages/github/components/ 以下にjqueryがdownloadされます  
また、package.json,web/config.jsにjspmのpackageとして登録されます



### web/assets/scripts/main.jsでjQueryを利用

EcmaScript6のimportか、CommonJSのrequireでjQueryを読み込みます

```js
import $ from 'jquery';

$(()=>{
	var contents = $("#contents-container");
	contents.text("Hello World!! with jQuery");
});
```



### 動作確認

<http://localhost:8000>



## CSSを読み込んでみる

pluginによる拡張でcssを読み込むことも可能です



### css pluginをinstall

```sh
npm run jspm install css
```



### src/assets/styles/main.stylusを用意

以下をコピペしてsrc/assets/styles/main.stylusを作成してください

```stylus
@import 'nib'

reset-html5()

html
	min-height: 100%

body
	min-height: 100%
	background: linear-gradient(top, white, lightblue)
```



### stylusをコンパイル

```sh
npm run gulp stylus
```



### web/assets/scripts/main.jsでcssを読み込み

```js
import $ from 'jquery';
import '../styles/main.css!';

$(()=>{
	var contents = $("#contents-container");
	contents.text("Hello World!! with jQuery");
});
```



### 動作確認

<http://localhost:8000>



***以上おつかれさまでした***
