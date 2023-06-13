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
<br>
### fiber Config
이제부터는 fiber의 routing 이외에 config나 middleware 등 여러 가지에 대해 더 알아보도록 하자!
```go
//아래처럼 config를 사용해서 초기에 custom하게 app을 설정할 수 있다. field는 이외에도 다양하게 존재하니 참고!
app := fiber.New()
app := fiber.New(fiber.Config{
	AppName: "Test App",
	CaseSensitive: true,
	StrictRouting: true,
})
```
static의 경우도 위의 app의 config를 이용해서 custom한 것처럼 config를 이용해서 더 다양한 설정을 할 수 있다. 아래 옵션 이외에도 download cacheduration 같은 더 다양한 옵션들이 있고 필요한 경우 설정해서 쓰면 좋을 것 같다.
```go
app.Static("/", "./public", fiber.Static{
  Compress:      true,
  ByteRange:     true,
  Browse:        true,
  Index:         "index.html",
  MaxAge:        3600,
})
```

### Use keyword
fiber에서는 Use라는 func이 존재하는데 이건 middleware 패키지를 사용하거나 url을 prefix하기 위해서 사용된다. 아래는 Use를 prefix를 위해서 사용하였고 Next 함수를 통해서 해당 api 이외에 맨 아래에 있는 Get 요청을 받고 있는 라우팅도 처리가 될 수 있다.
```go
package main

import (
	"fmt"
	"github.com/gofiber/fiber/v2"
)

func main() {
	app := fiber.New()
	// 모든 요청에 대해
	app.Use(func(c *fiber.Ctx) error {
		fmt.Println("nothing")
		return c.Next()
	})
	// /api로 오는 요청에 대해
	app.Use("/api", func(c *fiber.Ctx) error {
		fmt.Println("api")
		return c.Next()
	})
	// /test나 /use로 오는 요청에 대해
	app.Use([]string{"/test", "/use"}, func(c *fiber.Ctx) error {
		return c.Next()
	})
	// 여러 개의 핸들러를 등록하는 경우
	app.Use("/multi", func(c *fiber.Ctx) error {
		return c.Next()
	}, func(c *fiber.Ctx) error {
		return c.Next()
	})
	app.Get("/", func(c *fiber.Ctx) error {
		return c.SendString("hello, one")
	})
	app.Listen(":3000")
}
```

### Mount
fiber에서는 mount를 통해 fiber instance를 마운팅할 수 있다. 아래처럼 app에 mount fiber 인스턴스가 마운팅되게 되면 mount의 get은 /app/mount라는 경로에 대해서 요청을 처리하게 된다. endpoint가 마운트 순서에 따라 달라지기 때문에 mount같은 경우는 order도 중요 요소인 것 같다.
package main

import (
	"fmt"
	"github.com/gofiber/fiber/v2"
	"log"
)
func main() {
	app := fiber.New()
	mount := fiber.New()
	app.Mount("/app", mount)

	mount.Get("/mount", func(ctx *fiber.Ctx) error {
		return ctx.SendStatus(fiber.StatusOK)
	})
	one := fiber.New()
	two := fiber.New()
	three := fiber.New()
	two.Mount("/three", three)
	one.Mount("/two", two)
	app.Mount("/one", one)
	fmt.Println(one.MountPath()) // /one
	fmt.Println(two.MountPath()) // /one/two
	fmt.Println(three.MountPath())  // /one/two/three
	fmt.Println(app.MountPath())   // nothing
	log.Fatal(app.Listen(":3000"))
}
```

### Group
fiber에서는 group을 이용해서 routing을 그룹으로 처리할 수도 있다. 최종적으로 v1 인스턴스는 /app/v1/list에 대해 요청을 처리하게 된다.
```go
package main

import (
	"github.com/gofiber/fiber/v2"
	"log"
)

func main() {
	app := fiber.New()

	api := app.Group("/api", func(ctx *fiber.Ctx) error {
		return ctx.Next()
	})
	v1 := api.Group("/v1", func(ctx *fiber.Ctx) error {
		return ctx.Next()
	})
	v1.Get("/list", func(ctx *fiber.Ctx) error {
		return ctx.SendString("api/v1/list")
	})

	log.Fatal(app.Listen(":3000"))
}

```

### Route
fiber에서는 Route라는 func을 이용해서 prefix를 설정해서 routing할 함수를 정의할 수도 있다. 요기서 Name이라는게 등장하는데 요건 추후에 뒤에서 다룰 내용이지만 route에 naming을 해서 나중에 route을 get할때 name을 가지고 가져올 수 있다.
```go
package main

import (
	"github.com/gofiber/fiber/v2"
	"log"
)

func main() {
	app := fiber.New()

	app.Route("/routes", func(router fiber.Router) {
		router.Get("/v1", func(c *fiber.Ctx) error {
			return c.SendString("test/v1")
		}).Name("v1")

	}, "routes.")

	log.Fatal(app.Listen(":3000"))
}

```

### Server shutdown
fiber에서는 다양한 server를 종료하는 func들이 제공되는데 먼저 Shutdown 같은 경우는 active connection들이 종료될 떄까지(listener들의 종료를 기다린다) 기다렸다가 종료하게 되고 ShutdownWithTimeout은 강제로 시간이 자니면 close하게 되고 ShutdownWithContext은 context의 deadline이 넘게되면 종료되게 된다.(여기서 Context는 유지해야할 value 값을 저장해서 전달하고 필요한 곳에서 값을 꺼내 사용하기 위해서 사용되는 go의 타입!)
```go
func (app *App) Shutdown() error
func (app *App) ShutdownWithTimeout(timeout time.Duration) error
func (app *App) ShutdownWithContext(ctx context.Context) error
```

### Stack
stack을 이용하면 내가 정의한 router에 대한 method, path, params에 대한 정보를 확인할 수 있다.
```go
func main() {
	app := fiber.New()

	app.Get("/test/:val", handler)
	app.Post("/post", handler)

	data, _ := json.MarshalIndent(app.Stack(), "", "	")
	fmt.Println(string(data))

	log.Fatal(app.Listen(":3000"))
}
```
```json
[
	[
		{
			"method": "GET",
			"name": "",
			"path": "/test/:val",
			"params": [
					"val"
			]
		}
	],
	[
		{
			"method": "HEAD",
			"name": "",
			"path": "/test/:val",
			"params": [
					"val"
			]
		}
	],
	[
		{
			"method": "POST",
			"name": "",
			"path": "/post",
			"params": null
		}
	],
]
```

### Name
name함수는 앞에서 언급했듯이 router에 naming을 하기 위한 함수이다. group으로 묶으면 naming을 test.~ 처럼 할 수 있는 것 같다.
```go
func main() {
	app := fiber.New()
	app.Get("/test", handler).Name("main")
	gr := app.Group("/api")
	gr.Name("test.")
	gr.Get("/v1", handler).Name("v1")

	data, _ := json.MarshalIndent(app.Stack(), "", "	")
	fmt.Println(string(data))

	log.Fatal(app.Listen(":3000"))
}
```
```json
[
 [
  {
   "method": "GET",
   "name": "main",
   "path": "/test",
   "params": null
  },
  {
   "method": "GET",
   "name": "test.v1",
   "path": "/api/v1",
   "params": null
  }
 ],
]

```

### GetRoute
이전에 사용했던 naming을 기반으로 router에 대한 정보를 불러오는 함수로 GetRoute가 있다.
```go
func main() {
	app := fiber.New()
	app.Get("/test", handler).Name("main")
	data, _ := json.MarshalIndent(app.GetRoute("main"), "", " ")
	// 전체 라우터에 대한 정보
	data, _ := json.MarshalIndent(app.GetRoutes(true), "", " ")
	fmt.Println(string(data))
	log.Fatal(app.Listen(":3000"))
}
```

### Test
작성한 router에 대한 test를 Test라는 함수를 통해 진행해볼 수 있다. 아래는 req를 하나 생성해서 직접 test를 해보는 예제 코드이다!

```go
package main
import (
	"fmt"
	"github.com/gofiber/fiber/v2"
	"io"
	"net/http/httptest"
)
func main() {
	app := fiber.New()
	app.Get("/", func(c *fiber.Ctx) error {
		fmt.Println(c.BaseURL())
		fmt.Println(c.Get("X-Test-Header"))
		return c.SendString("test resp")
	})
	req := httptest.NewRequest("GET", "http://test.com", nil)
	req.Header.Set("X-Test-Header", "test")
	res, _ := app.Test(req)
	if res.StatusCode == fiber.StatusOK {
		body, _ := io.ReadAll(res.Body)
		fmt.Println(string(body))
	}
}
```