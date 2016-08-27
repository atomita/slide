# stylusのselectorが  
# 便利だった話

### 2016-08-27

### atomita



## まとめ



- BEMで書くときにはstylusが便利
- マルチクラスな設計ならばsassもあり
- sassで`&`の親selectorを参照する方法あったら教えてください



## やっと最近、CSSをBEMで書くようになりました

- [BEM](https://en.bem.info/)
- [BEMによるフロントエンドの設計 - 基本概念とルール | CodeGrid](https://app.codegrid.net/entry/bem-basic-1)



## なぜBEMにしたか



- シングルクラスになるのでCSSの適用順で悩むことが減るはず
- 塊だなと思ったらblockで名前つければいい感じなので、意外とやりやすそう？



## BEMのCSSの例

```css
.block,
.block_modifier {
  width: 600px;
  margin-top: 20px;
}
.block__element,
.block__element_modifier {
  background: #999;
  color: #f00;
}
.block__element_modifier {
  color: #0f0;
}
.block_modifier {
  margin-top: 30px;
}
```



## しかし、そのままCSS書くのは辛い感じ



## CSSプリプロセッサを使おう

- [Sass](http://sass-lang.com/)
  - 最初に使ったCSSプリプロセッサ
- [LESS - The Dynamic Stylesheet language](http://less-ja.studiomohawk.com/)
- [stylus](http://stylus-lang.com/)
  - sassから乗り換えた
- 他いろいろ



## sassでやってみる

```sass
.block
  width: 600px
  margin-top: 20px

  &__element 
    background: #999
    color: #f00

  &__element_modifier 
    @extend .block__element
    color: #0f0

  &_modifier 
    @extend .block
    margin-top: 30px
```



## `@extend .block__element`って書くのが...

## `&`みたいにやれれば楽なのに



## いろいろ試した結果



```sass
.block
  @at-root
    %abstract.block
      width: 600px
      margin-top: 20px

    %abstract#{&}__element
      background: #999
      color: #f00

  @extend %abstract#{&}

  &__element
    @extend %abstract#{&}

    &_modifier
      @extend %abstract#{str-slice(#{&}, 0, -10)}  // -10は"_modifier"の文字数 * -1 - 1
      color: #0f0

  &_modifier
    @extend %abstract#{str-slice(#{&}, 0, -10)}
    margin-top: 30px
```



## `str-slice(#{&}, 0, -11)`

## 辛い...



## stylusなら`selector()`があるよ

- http://stylus-lang.com/docs/selectors.html#selector-bif



```stylus
.block
  /$abstract&
    width 600px
    margin-top 20px

  &
    @extend $abstract{selector('../')}

  &__element
    /$abstract&
      background #999
      color #f00

    &
      @extend $abstract{selector('../')}

    &_modifier
      @extend $abstract{selector('../')}
      color #0f0

  &_modifier
    @extend $abstract{selector('../')}
    margin-top 30px
```



## 工夫点

- `$`を使って`@extend`されるためだけのものを作ってゴミが出力されないようにしてます
- `@extend`されるものは`&`を使って、その場所のselector値を利用
- `@extend`するところでは`selector('../')`を使って、1つ上のselectorを参照



## おまけ



sassは`&`などの変数の保持が、1クラス単位になっている様子

```sass
.block
  &__element
    content: #{inspect(&)}

    &_moddify
      content: #{length(nth(&, 1))}
      content: #{length(&)}
      display: none

  .multi
    content: #{inspect(&)}
    
    .mod, .x, a, p
      content: #{length(nth(&, 1))}
      content: #{length(&)}
```



なので、`&`の上のselectorを参照するのはマルチクラスな設計であればできそう  
sassでシングルクラスで`&`の上のselectorを参照するほうほうがあったら教えてください



## おわり
