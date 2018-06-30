# Picodom改めultradom改めSuperfineを元にLibraryを作った話

2018-06-30

atomita



## [xatto](https://www.npmjs.com/package/xatto)っていうのを作りました

VirtualDOMを用いたFunction & Context ベースのView Layer Library

Picodomがbaseになっています



PicodomはHyperappのLibrary版といった立ち位置  

> ### Picodomとは?
>
> [Picodom](https://github.com/picodom/picodom) は VDOM でページを操作する 1 KB の JS ライブラリで、API は 2 つのみ。だからもちろん Picodom をそのまま使って SPA を作ることもできるけど、あくまでも軽量化を優先したスタイル。
>
> どちらかというと、ユーザーが自分の VDOM 系ビューライブラリ（P/React とか、Inferno, Riot, Hyperapp など）を作るのに便利に使えるということが、Picodom の存在意義だと思っている。Picodom をベースにしたオリジナルビューライブラリや、VDOM 系ツールとか、ユーザーが楽しく簡単に作れたら嬉しいと思う。
>
> [2018 年は Hyperapp の年だ](https://qiita.com/JorgeBucaran/items/c48446babe0627e25ee6#%E9%9B%A3%E3%81%97%E3%81%8B%E3%81%A3%E3%81%9F%E3%81%A8%E3%81%93%E3%82%8D)

- Picodom
- ultradom
- ultraDOM
- Superfine ←New!



> ## [Hyperapp](https://github.com/hyperapp/hyperapp) とは？
>
> Web アプリのフロントエンド用 JavaScript ライブラリ。React, Preact, Vue といった代表的なものよりもずっと小さく、1 KB という超軽量サイズ。他のライブラリに依存することなく使えて、さらに[スピードもある](http://www.stefankrause.net/js-frameworks-benchmark7/table.html) :fire: 
>
> Elmアーキテクチャーに基づいてて、アプリケーション設計はElmやReact、Reduxと似てるけど、ボイラープレートは少ないし、TypeScriptにも[対応して](https://github.com/hyperapp/hyperapp/blob/master/hyperapp.d.ts)、とにかくシンプル。
>
> [2018 年は Hyperapp の年だ](https://qiita.com/JorgeBucaran/items/c48446babe0627e25ee6#%E9%9B%A3%E3%81%97%E3%81%8B%E3%81%A3%E3%81%9F%E3%81%A8%E3%81%93%E3%82%8D)



### Motive



fault

- Hyperappは`actions`に後から機能を追加するのが難しい
  - v2で解消される予定
   - [RFC: Hyperapp 2.0 · Issue #672 · hyperapp/hyperapp](https://github.com/hyperapp/hyperapp/issues/672)
- `context`を扱う[hyperapp-context](https://www.npmjs.com/package/hyperapp-context)を使おうとすると既に作ったComponentsを作す必要がでてくる
  - HyperappのComponentsは下の2つの形で作れます
    - `(attrs, children) => VNode`
    - `(attrs, children) => (state, actions) => VNode`
    - es5で書くと
      - `function (attrs, children) { return VNode }`
      - `function (attrs, children) { return function (state, actions) { return VNode } }`
  - hyperapp-contextを適用すると後者の形が使えなくなり、変わりに下の形が使えるようになります
    - `(attrs, children) => (context, setContext) => VNode`



suggest

- 最初から`context`の仕組みを持ったものでいいんじゃ？
- 拡張libraryの作者が既存の動作を壊しにくいように、Componentsは`(attrs, children) => VNode`の形だけにしたらいいんじゃ？
- [RFC: Hyperapp 2.0](https://github.com/hyperapp/hyperapp/issues/672)で提案されているようにactionは単純な関数にできそう



## どんな感じで使うのか



Counters by xatto

<iframe height='265' scrolling='no' title='counters by xatto' src='//codepen.io/atomita/embed/eKRpmP/?height=265&theme-id=light&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/atomita/pen/eKRpmP/'>counters by xatto</a> by atomita (<a href='https://codepen.io/atomita'>@atomita</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



## Hyperappとの比較



Counters by Hyperapp

<iframe height='265' scrolling='no' title='Counters by hyperapp' src='//codepen.io/atomita/embed/oyQdMx/?height=265&theme-id=light&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/atomita/pen/oyQdMx/'>Counters by hyperapp</a> by atomita (<a href='https://codepen.io/atomita'>@atomita</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



Counters by React Context

<iframe height='265' scrolling='no' title='React Counters with Context' src='//codepen.io/atomita/embed/NzEBwr/?height=265&theme-id=light&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/atomita/pen/NzEBwr/'>React Counters with Context</a> by atomita (<a href='https://codepen.io/atomita'>@atomita</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



## 感想とか



- 自分でいいと思ったものを作るのは楽しい
- VirtualDOMを使ったLibraryを作りたいという野望がやっと達成された
  - 3年前のハッカーズチャンプルーのときに抱いてた
- よかったらgithubでstarください！



## whoami

- [atomita](https://github.com/atomita)
- Programmer
- 琉球インタラクティブ



### Comunity

- [Hackers Champloo](http://hackers-champloo.org/)
- [Golang 勉強会 in Okinawa](https://okinawa-go.doorkeeper.jp/)
- [JAWS-UG沖縄](https://jaws-ug-okinawa.doorkeeper.jp/)
- [GCPUG Okinawa](https://okipug.connpass.com/)
- [Okinawa Frontend](https://okinawa-frontend.doorkeeper.jp/)
- [Laravel MeetUp Okinawa](https://laravel-meetup-okinawa.connpass.com/)
- [JavaScript MeetUp Okinawa](https://javascript-meetup-okinawa.connpass.com/)
- [Okinawa.rb](http://ruby.okinawa/)
- [CoderDojo Ginowan](http://www.coderdojo-ginowan.com/)



## End
