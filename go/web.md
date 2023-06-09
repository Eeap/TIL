# Web Document
이번에는 go를 이용해서 웹 어플리케이션을 만들고자 한다. go에는 기본적으로 `net/http`라는 내장 패키지가 존재하고 github의 여러 개의 웹프레임워크가 존재하는데 https://github.com/mingrammer/go-web-framework-stars 이쪽에 가면 go web framework를 잘 정리해놔서 참고하면 좋을 것 같다!

z
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