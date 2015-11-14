# fluent-plugin-aws-elasticsearch-service

# を作りました

### 2015-11-14

### atomita



## 自己紹介

- <h3>4年ほど前に沖縄にきました</h3>
- <h3>4年ほどWebプログラマ</h3>

- <h3>3ヶ月ほど前から  
  琉球インタラクティブに所属してます</h3>
    - 最近 [JobAntenna](https://www.jobantenna.jp/) をリニューアル

- https://github.com/atomita



## fluentdのpluginです



## fluentdとは



> Fluentd（フルエントディー）とは、オープンソースのデータコレクターやデータログ収集ツールと呼ばれるソフトウェアです。
> Fluentdを用いれば、今までのログ収集方式より格段に手軽にログを収集し、活用することができます。

> http://www.idcf.jp/words/fluentd.html



## fluentdからelasticsearchに送るpluginがすでにありました

### [fluent-plugin-elasticsearch](https://rubygems.org/gems/fluent-plugin-elasticsearch/)



## しかし、これはBasic認証しか（？）サポートしていないようです



## せっかくAWSなので

## IAMでの認証をできるようにしたのが

## [fluent-plugin-aws-elasticsearch-service](https://rubygems.org/gems/fluent-plugin-aws-elasticsearch-service/)です



## 現状の問題



### 1

#### 海外の方からの[issue](https://github.com/atomita/fluent-plugin-aws-elasticsearch-service/issues/2)が1つ解決できていません(;_;)

#### Elasticsearch Serviceの認証でつまづいているようですが

#### 再現できておらず...



### 2

#### faraday_middleware-aws-signers-v4に

#### モンキーパッチをあてて使っているので

#### ちゃんと対応して

#### faraday_middleware-aws-signers-v4にプルリクしたい



## ご清聴ありがとうございました！
