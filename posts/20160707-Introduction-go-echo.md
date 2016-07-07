# [echo](https://github.com/labstack/echo) を試してみた

### 2015-11-14

### atomita



## まとめ

- スクリプト言語と錯覚するくらい簡単
- JSONを出力するのは`map[string]interface{}`を`json.Marshal`に渡すのが一般的？



## 環境構築



### anyenv いれて

```sh
git clone https://github.com/riywo/anyenv ~/.anyenv
export PATH="$HOME/.anyenv/bin:$PATH"
eval "$(anyenv init -)"
```

`export`, `eval`は .profile とかに書いとく

参考 [anyenvで開発環境を整える - Qiita](http://qiita.com/luckypool/items/f1e756e9d3e9786ad9ea)



### goenv いれて

```sh
anyenv install ndenv
```



### go いれて

```sh
goenv install 1.6
goenv global 1.6
```



### GOPATH 設定用に direnv いれて

```sh
git clone http://github.com/zimbatm/direnv
cd direnv
make install DESTDIR=$HOME

eval "$(direnv hook bash)"
```

`eval`は .profile とかに書いとく



### directory つくって .envrc 設定して direnv 有効化して

```sh
mkdir -p workspace/tryecho && cd $_
echo 'layout go' > .envrc
echo 'path_add GOPATH packages' >> .envrc
echo 'export GOBIN=$(pwd)/bin' >> .envrc

direnv allow
```

参考 [direnvで解決するGOPATHの3つの問題点 - None is None is None](http://doloopwhile.hatenablog.com/entry/2014/06/18/010449)



### 準備完了...



## Hellow World



### コードを書く

```go
package main

import (
	"github.com/labstack/echo"
	"github.com/labstack/echo/engine/standard"
	"github.com/labstack/echo/middleware"
	"net/http"
)

func main() {
	e := echo.New()

	e.Use(middleware.Logger())
	e.Use(middleware.Recover())

	e.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "Hello, World!")
	})

	e.Run(standard.New(":8080"))
}

```



### おもむろに実行して browser で開く

```sh
go run main.go
```

[http://localhost:8080](http://localhost:8080)



## JSON 返してぇ



### コード書く

```go
	e.GET("/api/helloworld", func(c echo.Context) error {
		data := map[string]interface{}{"result": "hellow, world"}
		encoded, _ := json.Marshal(data)
		return c.String(http.StatusOK, string(encoded))
	})
```

参考 [https://gobyexample.com/json](https://gobyexample.com/json)



## おしまい
