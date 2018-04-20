# はじめてのImages Go API

2018-04-21

atomita



#### Self introduction

- 琉球インタラクティブ
- Programmer
- Go lang勉強会 core member
- Hackers Champloo 2018 実行副委員長



## Images Go APIって？



> The Images API provides the ability to serve images directly from Google Cloud Storage or Blobstore, and to manipulate those images on the fly.
> 
> [Images Go API Overview  |  App Engine standard environment for Go  |  Google Cloud](https://cloud.google.com/appengine/docs/standard/go/images/)



Images APIは、Google Cloud StorageやBlobstoreから直接画像を提供し、その画像をその場で操作する機能を提供します。

類似Services
- [Cloudinary](https://cloudinary.com/)
- [imgix](https://www.imgix.com/)
- [Image Resizer](https://imageresizer.io/)
- ...



## 既にできていた構成



![before](./assets/images/20180421/before.png)



## 変更後の構成



![before](./assets/images/20180421/after.png)



## 

`http://lh*.googleusercontent.com/xxxxxxx`って感じのURLが返されます





## Notes



### 画像はpublicな状態

- lockをかけることはできません



### URLに付けられるparameters

- documentには、`s`(size)と`c`(crop)しか載ってないですが、他にもあります
  - [HerokuからGoogle App EngineのImage APIを使ってGoogle Cloud Storageにある画像を動的変換する - crispy blog](http://blog.crispy-inc.com/entry/2016/12/17/185254)の「Image APIに関して」が一番詳しかった
- ただし、<div>
> Important: Only the resize and crop arguments listed above are supported. Using any other arguments might result in breaking failures.
> 
> 重要: 上記のresizeとcropの引数だけがサポートされています。他の引数を使用すると、障害が発生する可能性があります。
</div>とのことです



### `gcloud app deploy`はvendoringをsupportしていません

- directory構成をうまいことすると大丈夫らしいので先人に習いましょう
  - [実践的なGAE/Goの構成について #golang #gcpja - Qiita](https://qiita.com/koki_cheese/items/216fe73caf958db34aa2)
  - `app.yml`を1段深いdirectoryに収めて、ghqを使うようです
- 案件では依存するtoolsを減らしたかったので、deploy時にvendorsを移動させることでerrorを回避しました(力技



## End
