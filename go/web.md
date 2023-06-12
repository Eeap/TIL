# Web Document
이번에는 go를 이용해서 웹 어플리케이션을 만들고자 한다. go에는 기본적으로 `net/http`라는 내장 패키지가 존재하고 github의 여러 개의 웹프레임워크가 존재하는데 https://github.com/mingrammer/go-web-framework-stars 이쪽에 가면 go web framework를 잘 정리해놔서 참고하면 좋을 것 같다!

아래는 간단하게 net/http를 이용해서 http req를 처리하는 코드이다. `http.HandleFunc`을 이용해서 새로운 핸들러를 등록할 수 있고 `ListenAndServe`를 통해 몇번 포트로 listen할지 정할 수 있다. `http.Request`는 request와 파라미터에 대한 정보를 담고 있다. GET 요청에 대한 파라미터를 받으려면 `r.URL.Query().Get("test")`를 이용하면 되고 POST에 대한 데이터를 가져올 경우 `r.FormValue("test")`를 이용하면 된다.
```go
package main

import (
	"fmt"
	"net/http"
)
func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, This is request %s\n", r.URL.Path)
	})
	http.ListenAndServe(":80", nil)
}
```
위와 달리 아래는 정적 파일인 images나 js 같은 파일들에 요청에 대한 처리를 구현한 코드이다.
```go
func main() {
	fs := http.FileServer(http.Dir("static/"))
	http.Handle("/static/", http.StripPrefix("/static/", fs))
	http.ListenAndServe(":80", nil)
}
```

## Web Framework
Go에는 net/http 패키지 같은 것 말고도 많은 framework가 존재한다. Gin이랑 Fiber 둘 중 고민하다가 Gin이 무거워서 Fiber랑 echo를 쓰는 경우가 많다해서 그 중에 나는 Fiber를 사용해보려고 한다! 설치는 매우 간단하다 `go get github.com/gofiber/fiber/v2`이 명령어만 입력해주면 끝이다!

두번째 파라미터는 핸들러 함수 이고 함수 안에는 *fiber.Ctx는 재활용돼서 다른 req에서 사용된다고 한다.
```go
package main

import "github.com/gofiber/fiber/v2"

func main() {
	app := fiber.New()
	app.Get("/", func(c *fiber.Ctx) error {
		return c.SendString("hello")
	})
	app.Listen(":3000")
}
```
아래처럼 app 뒤에 메소드를 정의해서 사용할 수 있고 여러 개의 handler를 등록해서 사용할 수도 있다.`app.Method(path string, ...func(*fiber.Ctx) error)`
```go
package main

import "github.com/gofiber/fiber/v2"

func main() {
	app := fiber.New()
	//app.Get("/", func(c *fiber.Ctx) error {
	//	return c.SendString("hello")
	//})
	// parameter
	app.Get("/:value", func(c *fiber.Ctx) error {
		return c.SendString("hello, " + c.Params("value"))
	})
	// optional parameter
	app.Get("/:value?", func(c *fiber.Ctx) error {
		if c.Params("value") != "" {
			return c.SendString("hello," + c.Params("value"))
		}
		return c.SendString("Who r u ?")
	})
	// wildcards
	app.Get("/api/*", func(c *fiber.Ctx) error {
		return c.SendString("api path: " + c.Params("*"))
	})
	// staic files
	app.Static("/static", "./")
	app.Listen(":3000")
}

```
static file의 경우엔 `app.Static(prefix, root string, config ...Static)` 첫 번째 파라미터에 prefix를 적어주게 해당 prefix 이후에 요청하는 static 파일을 root 경로에서 찾아서 반환해준다. 예를 들어 위에서 /static/index.html을 요청했다면 현재 디렉토리에 있는 index.html 파일을 찾아서 반환해주게 된다.