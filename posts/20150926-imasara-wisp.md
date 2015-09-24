# いまさら wisp

### 2015-07-25

### atomita



## 自己紹介

- <h3>4年半ほど前に沖縄にきました</h3>
- <h3>4年半ほどWebプログラマ</h3>

- <h3>3ヶ月ほど前から  
  琉球インタラクティブに所属してます</h3>

- https://github.com/atomita



## まとめ



- <h2>ClojureでJavaScript</h2>
- <h2>まずはgulpfileから</h2>



## altJS 使ってますか？



- <h2>CoffeeScript</h2>
- <h2>TypeScript</h2>
- <h2>他いろいろ</h2>



## wisp も altJS の1つ
## [https://github.com/Gozala/wisp](https://github.com/Gozala/wisp)



- <h2>Clojure syntax</h2>
	- S式
	- マクロ  
- <h2>node.js でコンパイル</h2>
	- ブラウザ上でもコンパイルできる  
- <h2>ClojureScriptと比較して  
  可読性の高いJavaScriptが出力される</h2>
	- [http://www.jeditoolkit.com/try-wisp/](http://www.jeditoolkit.com/try-wisp/)



## Clojureってなに？



## 少しカッコが少ないLispです



## カッコはそれなりに多い...？



```wisp
; wisp
(console.log (.to-string (Date.)))
```



```js
// js
console.log(new Date().toString());
```



## この程度なら数は同じ

jsと張り合っても... という声が聞こえてきそうですが



## gulpfile から wisp
## 始めてみませんか



## wisp なら



## これが

```js
// js
var gulp = require("gulp"),
  plugins = require("gulp-load-plugins"),
  stylus = require("stylus"),
  nib = require("nib");

gulp.task('stylus', function () {
  return gulp.src(['./**/*.styl(us)', '!./**/_*.styl(us)'])
    .pipe(plugins.plumber({
      'errorHandler': plugins.notify.onError({
        'title': 'task: stylus',
        'message': 'Error: <%= error.message %>'
      })
    }))
    .pipe(plugins.stylus({
      'define': { 'url': stylus.resolver() },
      'resolve url': true,
      'use': [nib()],
      'import': 'nib'
    }))
    .pipe(gulp.dest('./'))
    .pipe(plugins.minifyCss())
    .pipe(plugins.rename({ 'extname': '.min.css' }))
    .pipe(gulp.dest('./'));
});
```



## こう書けます

```wisp
; wisp
(ns app.tasks
  "tasks"
  (:require [gulp]
            [gulp-load-plugins :as plugins]
            [stylus]
            [nib]))

(gulp.task :stylus
   (fn []
     (gulp-line
      src ["./**/*.styl(us)" "!./**/_*.styl(us)"]
      | plugins.plumber
                {:errorHandler
                 (plugins.notify.onError
                  {:title "task: stylus"
                   :message "Error: <%= error.message %>"})
                 }
      | plugins.stylus
                {:define {:url (stylus.resolver)}
                 "resolve url" true
                 :use [(nib)]
                 :import :nib
                 }
      > "./"
      | plugins.minifyCss
      | plugins.rename {:extname ".min.css"}
      > "./"
      )))
```

※ defmacroは省略してます



## ご静聴ありがとうございました



## wisp のいいところ

個人的に、です



## ns と require



```wisp
; wisp
(ns app.main
  "doc comment"
  (:require [jquery]
            [app.sub]))
```



```js
// js
{
    var _ns_ = {
            id: 'app.main',
            doc: 'doc comment'
        };
    var jquery = require('jquery');
    var app_sub = require('./sub');
}
```



## foo?



```wisp
; wisp
(foo?)
```



```js
// js
isFoo();
```



## 四則/論理演算が可変長引数



```wisp
; wisp
(+ 1 2 3 4 5 6 7 8 9 10)
(and a b c d) 
```



```js
// js
1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10;
a && b && c && d;
```



## exports.foo



```wisp
; wisp
(def foo "fooooo")
```



```js
// js
var foo = exports.foo = 'fooooo';
```



## Overloads



```wisp
; wisp
(fn log
  ([] nil)
  ([v] (console.log v))
  ([& args] (console.dir args)))
```



```js
// js
(function log() {
    switch (arguments.length) {
    case 0:
        return void 0;
    case 1:
        var v = arguments[0];
        return console.log(v);
    default:
        var args = Array.prototype.slice.call(arguments, 0);
        return console.dir(args);
    }
});
```



## macro



```wisp
; wisp
(defmacro unless
  [condition form]
  (list 'if condition nil form))

(unless true (console.log "bar"))
```



```js
// js
true ? void 0 : console.log('bar');
```



## もう少しmacroを掘り下げて
## 公式に書いてある
## defmacro -> の解説



```wisp
; wisp
(defmacro ->
  [& operations]
  (reduce
   (fn [form operation]
     (cons (first operation)
           (cons form (rest operation))))
   (first operations)
   (rest operations)))
```



## これは何かというと
## メソッドチェーンを
## 書きやすしてる
## (wisp的に)



## wispでmacroを使わずに書くと



```wisp
; wisp
(.fade-out (.delay (.fade-in ($ ".target")) 1000))
```



```js
// js
$('.target').fadeIn().delay(1000).fadeOut();
```



## ...



## -> を使ってみる



```wisp
; wisp
(-> ($ ".target") (.fade-in) (.delay 1000) (.fade-out))
```



## -> が何をやっているかというと



## 最初の引数`($ ".target")`を
## 最初矢印のところに入れてS式を構成して
## そのS式を次の矢印のところに入れて
## そのS式を〜と展開している

```wisp
; wisp
(-> ($ ".target") (.fade-in  ) (.delay   1000) (.fade-out  ))
;;                         ↑          ↑                 ↑
```



## ほんとうにご清聴ありがとうございました
