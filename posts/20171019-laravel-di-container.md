# LaravelのDI Containerの話

### 2017-10-19

### atomita



## DI ContainerはLaravelのcoreだと言っても過言ではない！  
と思う



## 前置き



### DI Containerとは



DI(Dependency Injection)してくれるContainer

IoC(Inversion of Control) patternからの派生らしい



#### Dependency Injection?



> 依存性の注入（いそんせいのちゅうにゅう、英: Dependency injection）とは、コンポーネント間の依存関係をプログラムのソースコードから排除し、外部の設定ファイルなどで注入できるようにするソフトウェアパターンである。英語の頭文字からDIと略される。  
[依存性の注入 - Wikipedia](https://ja.wikipedia.org/wiki/%E4%BE%9D%E5%AD%98%E6%80%A7%E3%81%AE%E6%B3%A8%E5%85%A5)

依存性の注入は、constructorかmethodで行うのが一般的



- Constructorの引数で依存性を注入する"constructor injection"
- Methodの引数で依存性を注入する"method injection"
    - setterを用意するものは、"setter injection"とも言われる
    - Interfaceでmethodを定義させる"interface injection"もある
        - phpなら合わせてtraitも定義すると使いやすい



## Monologを使った例



### DIしないで使う場合



```php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

class Example
{
    public function __construct()
    {
        $this->logger = new Logger('app');
        $this->logger->pushHandler(new StreamHandler(__DIR__.'/../logs/app.log', Logger::DEBUG));
    }
    public function run()
    {
        $this->logger->debug("Example class run");
    }
}
```



### DIする場合



```php
use Psr\Log\LoggerInterface;

class Example
{
    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }
    public function run()
    {
        $this->logger->debug("Example class run");
    }
}
```



DIありだと、Example classはinstance化する際に、`LoggerInterface`を渡さなければならなくなりました  
(オブジェクトの生成と使用が分離されている)  
このように、依存しているもの(class)を、外部から渡すようにするのがDI patternです



なぜこのようにするかと言うと、粗結合となるため、テストしやすくなる効果があります  
(テスト以外でも活用できる)



## DI Container



先にも書いたとおり、DI + Container  
ContainerにはObjectの生成方法を設定しておくことで、それを基にDIしてくれます



### 先のExample classをDI Container使用して生成する例



```php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Psr\Log\LoggerInterface;
use Example;

$container = app(); // `app()`はContainerのinstanceを返す

$container->singleton(Psr\Log\LoggerInterface::class, function ($container) {
    $logger = new Logger('app');
    $logger->pushHandler(new StreamHandler(__DIR__.'/../logs/app.log', Logger::DEBUG));
    return $logger;
});

$example = $container->make(Example::class);
```



更にContainerにObjectの生成方法の設定するのを別のところで行うことで(Provider)、単純に使う場合には、依存性を意識することなく使用することができるようになります



### `make`でinstance化って冗長じゃ...

なんて思った人も居られるでしょう



しかし、利用するにあたってはinstance化の部分は意識する必要はありません



**むしろ使っちゃいけません**  
  
Service Locator patternになっちゃうので  
参考: [やはりあなた方のDependency Injectionはまちがっている。 — A Day in Serenity (Reloaded) — PHP, FuelPHP, Linux or something](http://blog.a-way-out.net/blog/2015/08/31/your-dependency-injection-is-wrong-as-I-expected/)



## Laravelがinstance化するとき`make`してくれてます



**Laravelがinstance化してくれる代表的なものは...**



**Controller!!**



### 例



引用: [サービスコンテナ イントロダクション](https://readouble.com/laravel/5.4/ja/container.html#introduction)

```php
namespace App\Http\Controllers;
use App\User;
use App\Repositories\UserRepository;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    protected $users;

    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    public function show($id)
    {
        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```



constractorの引数にType hintingすることで自動的にDIしてくれます



> これはコントローラ、イベントリスナ、キュージョブ、ミドルウェアなどで利用できます。  
> [サービスコンテナ - 依存解決 - 自動注入](https://readouble.com/laravel/5.4/ja/container.html#automatic-injection)



## 他には[Facade](https://readouble.com/laravel/5.4/ja/facades.html)でも利用されています



以下の2行は同様の結果になります  
(設定を変えていなければ)

```php
Log::debug("foo");
app()->make('log')->debug("foo");
```



## 5.0ではMethod injectionも実装



```php
namespace App\Http\Controllers;
use App\User;
use App\Repositories\UserRepository;
use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserController extends Controller
{
    protected $users;

    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    public function show($id, Request $request)
    {
        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user, 'request' => $request]);
    }
}
```



## [Container events](https://readouble.com/laravel/5.4/ja/container.html#container-events)



`resolving`と`afterResolving`の2つのeventがあり、利用例としては[FormRequest](https://readouble.com/laravel/5.4/ja/validation.html#form-request-validation)があり、[`resolving`で初期化用のmethod call、`afterResolving`で`validate` methodがcall](https://sourcegraph.com/github.com/laravel/framework@3a16d196bd8d2b7761c9b0060a30a3687c3ea201/-/blob/src/Illuminate/Foundation/Providers/FormRequestServiceProvider.php#L28)されます



## まとめ



### injectionは自動で

### Type hintingで設定

### methodにもinjection



## ご清聴ありがとうございました
