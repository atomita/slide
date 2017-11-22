# LaradockでElastic Beanstalkにdeploy

2017-11-22

atomita



## laradockにawsってcontainerがあった



### documentを読んでみたらeb(Elastic Beanstalk) commandが使えるcontainerの様子



### ebにdeployするためだよね？



## やってみた



### Require tools



- docker-compose
- git
- ~~ssh-keygen~~

Note: ssh-keygenはcontainerの中のものを使用するようにしました



### Install Laradock



```shell-session
mkdir laradock-eb
cd laradock-eb
git clone git@github.com:laradock/laradock.git
cd laradock
cp env-example .env
```

- references
  - [Getting Started - Laradock](http://laradock.io/getting-started/)

Note: `git clone`ではなくzipをdownloadしてunzipするのでも大丈夫です



### Install Laravel



workspace containerのcomposerを使ってlaravelをinstallします

```shell-session
docker-compose run --user=laradock workspace bash -c 'composer create-project --prefer-dist laravel/laravel src'
``` 

- references
  - [Usage - Getting Started - Laradock](http://laradock.io/getting-started/#usage)
  - [インストール 5.4 Laravel](https://readouble.com/laravel/5.4/ja/installation.html#installing-laravel)



```shell-session
tree -L 1 ..
> ..
> ├── laradock
> └── src
```

こういうdirectory構成になっているはず



### Run Laravel (apache2) by Laradock



#### 2つの設定

Note: nginxだったら1つでいいのですが、Elastic Beanstalkでphpを使う場合のAMIがapache2なため、それに合わせてapache2を使います



まずapache2のvirtual host



```apache
<VirtualHost *:80>
  ServerName localhost
  DocumentRoot /var/www/public/
  Options Indexes FollowSymLinks

  <Directory "/var/www/public/">
    AllowOverride All
    <IfVersion < 2.4>
      Allow from all
    </IfVersion>
    <IfVersion >= 2.4>
      Require all granted
    </IfVersion>
  </Directory>

</VirtualHost>
```

を`laradock/apache2/sites/localhost.conf`として保存します



次にlaradockの.env(`../laradock/.env`)を開いて下記diffを参考に変更します

```diff
- APPLICATION=../
+ APPLICATION=../src/
```



#### Run apache2



```shell-session
docker-compose up -d apache2
curl http://localhost
```

Note: localでweb serverやmysqlを動かしている環境ではportが衝突してerrorになります  
その場合、laradockの.envで該当のportの番号を変更してください



### Deploy



- AWS IAM User
- laradock aws
- php.ini by ebextensions

の準備が必要になります

- references
  - [IAM とは - AWS Identity and Access Management](http://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/introduction.html)
  - [AWS Elastic Beanstalk PHP プラットフォームを使用する - AWS Elastic Beanstalk](http://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/create_deploy_PHP.container.html#php-configuration-namespace)



#### AWS IAM



[IAM Management Consoleのユーザー](https://console.aws.amazon.com/iam/home?region=ap-northeast-1#/users)でユーザーを追加します



##### ユーザー詳細の設定

- ユーザー名
  - ex) eb-deploy-with-laradock
- アクセスの種類
  - [x] プログラムによるアクセス



##### アクセス権限を設定

既存のポリシーを直接アタッチをクリックして

- [x] AWSElasticBeanstalkFullAccess


Note: 仕事で利用する場合はFullAccessではないほうがいいかも？



確認 → ユーザーの作成 でIAM Userが作成されます

表示されたアクセスキーIDとシークレットアクセスキーは後ほど使用するので、memoしておきます

Note: シークレットアクセスキーは再表示できません  
忘れてしまった場合はアクセスキーIDを再作成してください



#### laradock awsの設定



deployのためのssh keyはeb commandでgenerateできるのですが  
laradock awsがなぜかssh key fileを必須としているので  
`aws/ssh_keys` directoryにdummy.pubを置きます  
後ほどeb commandでgenerateするので、ここではdummy fileを置くだけです

```shell-session
mkdir aws/ssh_keys
touch aws/ssh_keys/dummy.pub
```



#### php.ini by ebextensions



```yaml
option_settings:
  aws:elasticbeanstalk:container:php:phpini:
    document_root: /public
    memory_limit: 128M
    zlib.output_compression: "Off"
    allow_url_fopen: "On"
    display_errors: "Off"
    max_execution_time: 60
    composer_options: ""
```

を`../src/.ebextensions/php-settings.config`として保存します

- references
  - [AWS Elastic Beanstalk PHP プラットフォームを使用する - AWS Elastic Beanstalk](http://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/create_deploy_PHP.container.html)

Note: `composer_options`があることから推察した方がおられるかもしれませんが、`composer install`を実行してくれます



#### Prepare for deploy



gitのlocal repositoryを作成します

Note: eb commandはdeployするfileをgitから取得するため



```shell-session
docker-compose run aws bash <<EOS
  git init
  git config user.email "you@example.com"
  git config user.name "Your Name"
  git add .
  git commit -m "first commit"
EOS
```

Note: hostのgitでもok



続いて`eb init`を実行します



```shell-session
docker-compose run aws bash -c "eb init && cp -a ~/.ssh ./; cp ~/.aws ./; chown -R `id -u`:`id -u` .ssh; chown -R `id -u`:`id -u` .aws"
```

対話形式で入力を求められるので順次入力します


Note: `eb init`以降のcommandはssh keyとcredentialsを永続化させるためのものです  
`../src/.ssh`, `../src/.aws`にcopyされます  
強引な手法ですが、現状laradock awsでいい感じの永続化が適用されていないもので(^^;



- Select a default region (default is 3): 9
- (aws-access-id): IAM UserのアクセスキーID
- (aws-secret-key): IAM Userのシークレットアクセスキー
- Enter Application Name (default is "www"):
- It appears you are using PHP. Is this correct? (Y/n):
- Select a platform version. (default is 1): 5
- Do you wish to continue with CodeCommit? (y/N) (default is n):
- Do you want to set up SSH for your instances? (Y/n):
- Select a keypair. (default is 1):
  - [ Create new KeyPair ]
- Type a keypair name. (Default is aws-eb): 
- Enter passphrase (empty for no passphrase):
- Enter same passphrase again:

最後に`WARNING: Uploaded SSH public key for "aws-eb" into EC2 for region ap-northeast-1.`と出力されると思いますが問題ないはず

[EC2 Management Consoleのキーペア](https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#KeyPairs:sort=keyName)にいくとssh keyがuploadされているのが確認できます

Note: なんでwarningなのか誰かおしえてください...



##### First deploy (eb create)



```shell-session
docker-compose run aws bash -c 'cp -a .ssh ~/; cp -a .aws ~/; eb create'
```

対話形式で入力を求められるので順次入力します



- Enter Environment Name (default is www-dev):
- Enter DNS CNAME prefix (default is www-dev):
- Select a load balancer type (default is 1): 2



2回目以降のdeploy commandは↓になります

```shell-session
docker-compose run aws bash -c 'cp -a .aws ~/; eb deploy'
```



### End

以上で今回のsessionは終わりです



----


## おまけ

※ 以降は書きなぐりです

- databaseを使う
  - ladadockのmysql
    - Scaffold and Migrate
  - RDSのmysql
    - Migrate with ebextensions
- buildしたassetsをgit管理下におかないで、それもdeployする
  - [プロジェクトフォルダーの代わりに圧縮ファイルをデプロイする - EB CLI の設定 - AWS Elastic Beanstalk](http://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/eb-cli3-configuration.html#eb-cli3-artifact)
- phpのsession dataの保存先をdbに



### Use mysql with laradock



#### Configure



laradockの.env(`../laradock/.env`)を開いて下記diffを参考に変更します

```diff
- MYSQL_VERSION=8.0
- MYSQL_DATABASE=default
- MYSQL_USER=default
+ MYSQL_VERSION=5.7.19
+ MYSQL_DATABASE=homestead
+ MYSQL_USER=homestead
MYSQL_PASSWORD=secret
```



laravelの.env(`../src/.env`)を開いて下記diffを参考に変更します

```diff
- DB_CONNECTION=127.0.0.1
+ DB_CONNECTION=mysql
```



#### Scaffold and Migrate



laravelの標準機能で、user登録が簡単に構築できるのでそれを使います

Note: 標準機能では、user認証用のscaffoldしか用意されていません  
他のscaffoldは[laravel5.4でscaffoldしたい - Qiita](https://qiita.com/positrium/items/aa17b635dfc8ba03cffc)などを参考にthird partyのpackageを利用してください



```shell-session
docker-compose up -d mysql
docker-compose run --user=laradock workspace bash <<EOS
  php artisan make:auth
  php artisan migrate
EOS
```

reference: [認証 5.4 Laravel](https://readouble.com/laravel/5.4/ja/authentication.html)



tableはmigrations, password_resets, usersの3つが作られているはず

```
docker-compose run mysql mysql -u homestead -psecret
use homestead;
show tables;
exit
```

Note: migrationsは`php artisan migrate`をどこまで実行したかの記録  
password_resets, usersは`../src/database/migrations/`にあるmigration fileを元に作られたものです



[http://localhost](http://localhost)でuser登録ができるようになっているはず！



### Use database (RDS)



#### Create instance



[RDS · AWS Console](https://ap-northeast-1.console.aws.amazon.com/rds/home?region=ap-northeast-1#launch-dbinstance:ct=dashboard:)でdatabase instanceを作成



#### Set inbound in security group



[EC2 Management Console](https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#SecurityGroups:sort=groupId)でnetworkの設定をします  
RDS instanceのdetailsにlinkがあります



インバウンド → 編集

- type: MYSQL/Aurora
- protocol: TCP
- port: 3306
- source: custom  
  value: security group ID.
  - RDSに付けられているsecurity groupのIDを
    - しっかりsecurity groupを設定したほうがいいかもね
  - ex) sg-64f1071d



#### Set security group in Elastic Beanstalk

- references
  - [EB CLI の設定 - AWS Elastic Beanstalk](http://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/eb-cli3-configuration.html)



### Set environment



#### Migrate with ebextensions



src/.ebextensions/migrate.config

```yaml
commands:
  01migrate:
    command: php artisan migrate
    leader_only: true
```

- references
  - [Linux サーバーでのソフトウェアのカスタマイズ - AWS Elastic Beanstalk](http://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/customize-containers-ec2.html)

