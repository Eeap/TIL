# Concept Summnary

일단 언어에 대한 기본적인 지식을 쌓기 위해서 해당 사이트들 참고

https://gobyexample.com/

http://golang.site/Go/Basic﻿

https://gowebexamples.com/

go build를 하면 exe 실행파일로 만들어주고 go run하면 실행!! 예전에 수업에서 듣기론 go가 다른 python이나 js와 다르게 pip npm으로 패키지를 설치하는걸 go라는 커맨드로 다 할 수 있다고 했던 것 같다.

go를 실습해보기 위해선 goland를 이용했고(학생은 jetbrain 무료라서 이용)

기본적으로 변수 선언과 데이터 타입에 대해 공부를 시작했다
## variables

``` go
package main

import "fmt"

func main() {
	k := 10
	var k int = 10 // 위와 같은 표현
	var k int  // 만 하면 0
	
}
```
기본적으로 타입도 아래처럼 존재

부울린 타입 bool
문자열 타입 string
정수형 타입 int int8 int16 int32 int64uint uint8 uint16 uint32 uint64 uintptr
Float 및 복소수 타입float32 float64 complex64 complex128
기타 타입  => byte: uint8과 동일하며 바이트 코드에 사용  /   rune: int32과 동일하며 유니코드 코드포인트에 사용한다

---
## condition

그리고 go는 조건식에도 다른 언어의 for문처럼 문장식으로 사용할 수 있다.
``` go
package main

import "fmt"

func main() {
	k := 10
	if k == 1 {
		fmt.Println("k1")
	} else if k == 10 {
		fmt.Println(k)
	}
	i := 1
	max := 3
	// if 문에서 조건식을 사용하기 이전에 간단한 문장을 함께 실행 가능 val 은 if 문 블럭 안에서만 사용 가능 
	if val := i * 2; val < max {
		fmt.Println(val)
	}
}
```
---
## loop

go에서는 여러개의 string 가변파라미터 ...을 사용해서 받을 수도 있다.

그리고 아래에서 특이하게 for 루프를 구성할 수 있는데 msg라는 컬렉션에서 이제 idx와 해당 값 x를 하나씩 뽑아오는 그런 형태입니다
```go
func variadicFunc(msg ...string) {
	for idx, x := range msg {
		fmt.Println(idx, x)
	}
}
```
그리고 c++ 처럼 pass by ref로 주소를 넘겨 포인터로 받아서 참조를 할 수 있다.
``` go
package main

import "fmt"

func main() {
	
	msg := "test"
	passByRef(&msg)

}
func passByRef(msg *string) {
	fmt.Println(msg)
	*msg = "change"
}
```

---
## functions

아래처럼 return 값의 형식을 파라미터 뒤에 정의해줘서 사용할 수 있다. 위처럼 복수 개의 return 도 가능! 또한, Named Return Parameter라는 것도 존재하는데 이 방법은 리턴값들을 할당하여 리턴하는 방법이다. 마지막처럼 미리 리턴 타입 정의 부분에 파라미터명을 정의하고 마지막에 return 문만 써주면된다.
```go
func sum_num1(nums ...int) int {
	s:=0
	for _,n := range nums {
		s += n
	}
	return s
}
func main()  {
	total,count := sum_num(1,2,3,4,5)
	fmt.Println(total,count)
}
func sum_num2(nums ...int) (int,int) {
	s := 0
	count :=0
	for _, n := range nums {
		s += n
		count++
	}
	return s,count
}

func sum_num3(nums ...int) (s int, count int) {
	for _, n := range nums {
		s += n
		count++
	}
	return
}
```

아래는 익명 함수의 예제이며 형태는 js에서 썼던거랑 비슷한 것 같다,,
```go
func main() {
	sum := func(n ...int) (s int) {
		for _, i := range n {
			s += i
		}
		return
	}
	result := sum(1, 2, 3, 4)
	fmt.Println(result)
}
```

go에서 함수는 일급함수로서 go의 기본 타입과 동일하게 취급된다. 따라서 함수의 파라미터나 리턴에 함수 자체를 넣을 수도 있다.
```go
package main

import "fmt"

func main() {
	add := func(i int, j int) int { return i + j }

	r1 := calc(add, 10, 1)
	fmt.Println(r1)
	r2 := calc(func(i int, j int) int { return i - j }, 10, 1)
	fmt.Println(r2)
}

func calc(f func(i int, j int) int, a int, b int) int {
	result := f(a, b)
	return result
}
```

type문은 구조체 등 custom한 type을 정의하기 위해서 사용된다. 함수의 원형을 간단히 표현하는데도 사용된다. 예를 들어 위에서 calc 안에 있는 함수 타입을 아래처럼 minus로 선언하면 아래처럼 간단히 minus로 해서 간단히 표현 가능하다.
``` go
type minus func(i int, j int) int
func calc(f minus, a int, b int) int {
    result := f(a, b)
	return result
}
```
---
## closure
go 언어에서 함수는 closure로서도 사용이 가능한데 closure란 함수 바깥에 있는 변수를 참조하는 함수값을 말한다. 아래 코드처럼 nextValue함수는 익명함수를 리턴하고 그 함수는 또 int를 리턴하는 형태이다. 여기서 익명함수가 바깥에 있는 idx를 참조하게 되고 그 값을 하나 증가시키고 반환하게 된다. 그런데 여기서 신기한게 idx가 로컬 변수로 선언된것처럼 보이지만 사실상 next를 두번 호출하면 idx값이 그대로 계속 증가하게 된다. 이유는 익명함수 자체가 로컬 변수 idx를 갖게는 아니라 참조를 하게 되므로 idx의 값이 유지되게 되는 것 같다. 그래서 새로 다른 변수에 함수를 할당하고 실행하면 idx가 새로운 메모리에 만들어지고 그 변수는 그 새로 만들어진 idx만 참조하기 때문에 이전에 있던 next를 호출하면 이전에 참조하던 idx 값이 증가하게 된다.
```go
package main

import "fmt"

func nextValue() func() int {
	idx := 0
	return func() int {
		idx++
		return idx
	}
}

func main() {
	next := nextValue()
	fmt.Println(next())
	fmt.Println(next())

	anotherNext := nextValue()
	fmt.Println(anotherNext())
	fmt.Println(anotherNext())
}


```
---
## collection

### array
배열은 java랑 비슷한 것 같고 다른 언어랑 대체적으로 비슷

```go
package main

func main() {
	var a1 = [3]int{1, 2, 3} // 배열 크기 지정
	var a2 = [...]int{1, 2, 3, 4, 5} // 배열 크기 자동 지정

	var a3 = [2][4]int{
		{1, 2, 3, 4},
		{5, 6, 7, 8}, // 끝에 ,를 추가하지 않으면 에러가 남.
	}
}
```

### slice
go에서 배열은 배열의 크기를 동적으로 증가시키거나 부분 배열을 substr하는 기능을 가지고 있지 않기 때문에 slice라는 기능을 사용해야한다. go Slice는 배열의 크기를 미리 지정하지 않고 추후 그 크기를 동적으로 조절할 수 있고 부분 배열을 substr하는 기능을 가지고 있다.
```go
func main() {
	var a []int //slice 변수 선언
	a = []{1,2,3}  //값 지정
	s := make([]int, 5, 10)
	fmt.Println(len(s), cap(s))

	var s1 []int
	if s1 == nil {
		fmt.Println("nil")
		fmt.Println(len(s1), cap(s1))
	}
}
```
위처럼 slice를 선언하는 방법은 배열과 달리 빈 상태로 선언하게 되고 make은 slice를 생성하는 또 다른 방법이다. make은 첫번째 파라미터는 타입이고 두번째는 length(길이), 세번째는 capacity(최대 길이)이다. 처음에 slice를 make를 선언하면 옆에서는 5개의 요소가 모두 0값을 갖게 된다. 그냥 slice를 선언하게 되면 s1처럼 Nil slice가 되고 이는 용량과 길이를 모두 0값을 갖는다.
<br>
부분 슬라이스 경우에는 기존에 파이썬처럼 [startIndex,fromIndex)형식을 가진다.

또한, slice는 추가 병합 복사가 가능한데 추가 가능 경우엔 append를 이용하면 된다. 아래 예제는 append를 이용해 s라는 배열에 추가하는 예제이며, append의 파라미터로는 여러 개의 인자가 올 수 있다. 그리고 slice의 경우엔 length가 내부 최대 길이인 capacity를 넘게 되면 현재 capacity의 2배로 자동으로 늘어나게 된다. 실제로 print를 찍어보면 3,6,12,24순으로 늘어나게 된다.
```go
func main() {
	s := []int{0, 1}
	s = append(s, 2, 3, 4)

	newSlice := make([]int, 0, 3)
	for i := 0; i <= 15; i++ {
		newSlice = append(newSlice, i)
		fmt.Println(len(newSlice), cap(newSlice))
	}
}
```
go에서 병합을 할때도 마찬가지로 append라는 함수를 쓰게 된다. 이때 sliceB의 ...인 ellipsis를 붙이게 되는데 이건 sliceB의 모든 요소들의 집합을 의미한다.
```go

func main() {
	sliceA := []int{1, 2, 3}
	sliceB := []int{4, 5, 6}
	sliceA = append(sliceA, sliceB...)
	fmt.Println(sliceA)
}
```
slice를 복사하려고 하는 경우에는 copy라는 함수를 사용하게 된다. 이때 src 포인터의 데이터를 dst 포인터의 데이터로 배열을 복제하게 된다.
```go
func main() {
	src := []int{1,2,3}
	dst := make([]int, len(src),cap(src)*2)
	copy(dst,src)
	fmt.Println(dst)
	fmt.Println(len(dst),cap(dst))
}
```
슬라이스는 내부적으로 배열의 부분 영역인 세그먼트에 대한 정보를 가지고 있다. 슬라이스는 3개의 필드로 구성되어 있는데 첫번째 필드는 배열에 대한 포인터 정보이고 두번째 필드는 length, 세번째는 capacity 정보가 있다. 길이와 용량을 지정해서 선언하면 용량만큼 배열이 만들어지고 슬라이스 첫번째 필드인 포인터는 배열의 첫번째 요소를 가리키게 된다.

---
## Map
맵은 흔히 쓰이는 key value형 해시테이블을 구현한 자료구조이다. go에서는 `map[key타입]value타입` 이런식으로 선언해서 사용한다. 아래처럼 mapTest의 경우 nil 값을 가지게 되며 Nil Map이라고 부른다. Nil map에는 어떤 데이터도 쓸 수 없으며 panic에러가 뜨게 된다. `panic: assignment to entry in nil map` map을 초기화하기 위해서는 make함수를 사용할 수 있다. 이제 데이터를 넣으면 잘 출력이 되는 걸 확인할 수 있다. make 함수는 해시테이블 자료구조를 메모리에 생성하고 그 메모리를 가리키는 map value를 리턴하게 된다. 따라서 mapTest는 hashTable을 가리키는 map을 가리키게 되는 구조이다..!

그리고 아래처럼 리터럴을 사용해서 초기화할 수도 있다. (이때 마지막에 ,찍는거 잊지 않기!!) 값을 읽을때는 보통 다른 언어처럼 `변수명[key값]`이고 만약 값이 없다면 nil이나 zero가 리턴된다. 그리고 삭제할때도 delete라는 함수를 써서 변수명과 key값을 같이 주면 쉽게 삭제할 수 있다.
```go
func main() {
	var mapTest map[int]string
	fmt.Println(mapTest)
	mapTest = make(map[int]string)
	mapTest[1] = "test"
	fmt.Println(mapTest)
	target := map[string]string{
		"id":   "test",
		"name": "test1",
	}
	fmt.Println(target)
	delete(target,"id")
}
```
Map 키가 존재하는 체크하는 방법도 존재! 아래처럼 value와 존재 여부에 대한 exists를 받아서 확인할 수 있다.
```go
func main() {
	target := map[string]string{
		"id":   "test",
		"name": "test1",
	}
	value, exists := target["id"]
	if !exists {
		fmt.Println("No id value")
	}
	fmt.Println(value)
}
```
그리고 key, value를 for문으로 가져오는 방법도 있는데 이건 이전의 for문이 idx,val을 가져왔던 방법이랑 똑같다. 여기서 idx가 단순히 key로 바뀐거라고 생각하면 이해하기 편하다.
```go
func main() {
	target := map[string]string{
		"id":   "test",
		"name": "test1",
	}
	for key, val := range target {
		fmt.Println(key, val)
	}
}
```

---
## Package
Go에서는 패키지를 통해 코드의 모듈화나 코드의 재사용 기능을 제공. 그래서 패키지를 사용해서 작은 단위의 컴포넌트를 작성하고 이러한 패키지들을 활용해서 프로그램을 작성한다. go는 개발에 필요한 패키지들을 표준 라이브러리로 제공! 표준 라이브러리 패키지들은 GOROOT/pkg안에 존재한다. GOROOT 환경변수는 Go 설치 디렉토리를 가라키는데 이 부분은 자동으로 설치시 추가된다. [standard package](https://pkg.go.dev/std)
Go compiler는 main 패키지 안의 main함수를 프로그램의 시작점인 엔트리 포인트로 인식한다. 다른 패키지를 사용할 때는 import를 이용해서 사용하면 된다. 이전에 fmt같은 패키지를 사용했는데 그것처럼 하면 된다.
go에서는 표준인 경우에는 앞서 말한 것처럼 GOROOT/pkg에서 찾지만 사용자 패키지나 3rd party 패키지의 경우엔 GOPATH/pkg에서 패키지를 찾는다. GOPATH 같은 경우는 사용자가 지정해주어야 한다.
<br>
패키지 내에는 함수, struct, 인터페이스 등이 존재하는데 이들의 이름의 첫문자가 대문자로 시작하면 public으로 인식되고 소문자로 시작하면 non-public이 된다. public 같은 경우엔 흔히 알듯이 외부에서 호출해서 사용이 가능하고 non-public 같은 경우엔 패키지 내부에서만 사용이 가능하다. 
<br>
패키지 실행시 처음으로 호출되는 부분은 init 함수를 작성하면 자동으로 호출되도록 할 수 있다.
```go
package test

var mapTest map[string]string

func init() {
	mapTest = make(map[string]string)
}

//init 함수만 호출하고자 하는 경우에는 _라는 alias를 지정하면 된다.
import _ "ex/testLib"
// 패키지 이름이 동일해도 alias를 사용해서 구분 가능!
import (
	mongo "ex/mongo/db"
	mysql "ex/mysql/db"
)
func test() {
	mondb := mongo.get()
	mysql := mysql.get()
}
```

사용자 정의 패키지 같은 경우 src 폴더 아래 폴더를 만든 후(패키지 이름과 동일한) 거기에 .go파일들을 만들어 구성한다. 그렇게 되면 하나의 서브 폴더 안에 있는 go 파일들은 동일한 패키지명을 갖게 된다. 추가적으로 사이즈가 큰 라이브러리 같은 경우 `go install` 이라는 명령을 통해서 라이브러리를 컴파일하여 cache할 수 있다. 이렇게 하면 빌드 타입을 크게 줄일 수 있다. (이 부분은 c언어를 생각하면 편할 것 같다. c에서도 라이브러리를 만들면 .a 파일로 만들어지는데 go에서도 go install을 하면 라이브러리를 .a파일로 만들어서 pkg에 보관하는 것 같다..!)

### Struct
go에서는 struct는 custom data type을 표현하는데 사용 그래서 필드 데이터만 갖고 메소드는 갖지 않는다. struct를 정의하기 위해서는 type문을 사용한다.
```go
type person struct {
	name string
	age  int
}

func main() {
	p := person{}
	p.name = "Kim"
	p.age = 26
	fmt.Println(p)
}
```
위처럼 빈 객체를 할당해서 dot을 이용해서 필드값을 채워넣는 방법도 있고 괄호 안에 값을 넣어서 할당하는 방법이 있다. 아래에서 p2의 경우엔 일부 필드가 생략됐을 때 해당 값은 zeron value를 갖게 된다.
```go
	var p1 person
	p1 = person{"kim",25}
	p2 := person{name: "k", age: 24}
```
또 다른 객체 생성 방법으로는 new메소드가 있다. new를 사용하면 모든 필드를 zero value로 초기화하게 되고 해당 객체의 포인터를 리턴하게 된다. c와 달리는 go는 객체 포인터인 경우에도 dot을 사용해서 데이터에 접근한다.(포인터가 자동으로 Dereference되기 때문?!)
```go
	p3 := new(person)
	p.name = "kim"
```
struct 같은 경우도 struct 개체를 다른 함수의 파라미터로 넘길 경우 포인터를 전달하면 된다.
```go
type person struct {
	name map[int]string
}

func newStruct() *person {
	temp := person{}
	temp.name = map[int]string{}
	return &temp
}

func main() {
	p := newStruct()
	p.name[1] = "Kim"
	fmt.Println(p)
}

```

---
## Struct
struct에서 go는 java나 c와 달리 필드만 가지고 메소드는 갖지 않는다고 했는데 go에서는 아래처럼 메소드를 분리해서 쓰게 된다.
```go
package main

import "fmt"

type Rect struct {
	width, height int
}

func (r Rect) rectangle() int {
	return r.width * r.height
}

func main() {
	rect := Rect{5, 10}
	area := rect.rectangle()
	fmt.Println(area)
}
```
위처럼 Rect라는 struct에 rectangle이라는 메소드를 정의해서 사용할 수 있는데 함수명 앞에는 receiver라고 불리고 메소드가 속한 struct 타입과 struct 변수명을 지정하게 된다. 그래서 위에서는 r을 변수로 사용해서 함수 내부에서 입력 파라미터처럼 사용할 수 있다.
<br>
receiver의 종류는 value형과 pointer형이 있다. value receiver는 struct의 데이터를 복사하면 전달하고 pointer receiver는 struct의 포인터만을 전달한다. 따라서 value 같은 경우는 필드값이 변경되더라도 호출된 데이터는 변경되지 않고 포인터는 반대로 그대로 반영되게 된다. 그리고
```go
package main

import "fmt"

type Rect struct {
	width, height int
}
func (r Rect) rectangle() int {
	return r.width * r.height
}
func (r *Rect) rect_Pointer() int {
	r.width++
	return r.width * r.height
}
func main() {
	rect := Rect{5, 10}
	area := rect.rectangle()
	fmt.Println(area)
	area1 := rect.rect_Pointer()
	fmt.Println(area1)
}
```

---
## interface
인터페이스는 method들의 집합체이며 type이 구현해야 하는 메소드 prototype들을 정의한다. 인터페이스를 구현하기 위해서는 해당 타입이 그 인터페이스의 메소드들을 모두 구현하면 되는데 아래는 예로 들면 Shape이라는 인터페이스를 구현하기 위해서 Rect와 Circle이라는 type이 Shape의 메소드들을 아래처럼 모두 구현하면 된다. 아래 allArea처럼 입력 파라미터를 인터페이스를 구현한 타입 객체들을 받는 경우 해당 객체가 인터페이스를 구현(메소드를 전부 정의)하기만 하면 사용될 수 있다.

```go
package main

import (
	"fmt"
	"math"
)

type Shape interface {
	area() float64
	perimeter() float64
}
type Rect struct {
	width, height float64
}
type Circle struct {
	radius float64
}

func (r Rect) area() float64 {
	return r.width * r.height
}
func (r Rect) perimeter() float64 {
	return 2 * (r.width + r.height)
}
func (c Circle) area() float64 {
	return math.Pi * c.radius * c.radius
}
func (c Circle) perimeter() float64 {
	return 2 * math.Pi * c.radius
}
func main() {
	rect := Rect{10., 20.}
	circle := Circle{10}
	allArea(rect, circle)
}
func allArea(shapes ...Shape) {
	for _, s := range shapes {
		fmt.Println(s.area())
	}
}
```
go에서는 empty interface도 존재하는데 이러한 interface는 `interface{}`로 표현된다. 빈 인터페이스는 0개의 메소드를 구현하기 때문에 모든 Type을 나타낼 경우 사용하게 된다. 즉, Dynamic Type이라고도 부르고 아래처럼 정수형을 담았다가 문자열을 담았다가 해도 에러 없이 잘 동작이 되고 마지막에 담은 내용을 출력한다.
```go
func main() {
	var x interface{}
	x = 1
	x = "Kim"
	fmt.Println(x)
}
```
go에서는 빈 인터페이스의 type을 assertion해서 사용할 수 있다. 이런 경우에는 뒤에서 따로 명시한 Type과 일치할 경우엔 해당 값을 리턴하고 아닐 경우에는 error가 발생하게 된다.
```go
package main
import (
	"fmt"
)
func main() {
	var x interface{} = "string"
	i := x.(string)
	fmt.Println(i)
}
```

---
## Error
go에서는 내장 타입으로 error라는 인터페이스를 가진다. 이 인터페이스에 하나의 메소드를 가지고 있고 이걸 이용해서 에러 타입을 커스텀하게 만들 수 있다.
```go
type error interface {
	Error() string
}

func main() {
	f, err := os.Open("./test.txt")
	if err != nil {
		log.Fatal(err.Error())
	}
	fmt.Println(f.Name())
}
```
error를 custom하게 하는 방법은 다음과 같다. pointer receiver로 메소드를 선언해주고 출력하고 싶은 메시지 형태로 바꾸면 된다.
```go
type MyError struct {
	Code string
	Msg  string
}

func (e *MyError) Error() string {
	return e.Code + ", " + e.Msg
}

func main() {
	num, err := test(0)
	if err != nil {
		log.Fatal(err.Error())
	}
	fmt.Println(num)
}
func test(num int) (int, error) {
	if num > 1 {
		return -1, nil
	}
	return 1, &MyError{Code: "200", Msg: "Test"}
}
```
---
## defer & panic
defer 키워드는 특정 문장 혹은 함수를 나중에 실행하도록 한다. 여기서 panic이라는 키워드도 같이 등장하는데 panic함수는 현재 함수를 즉시 멈추고 함수에 defer 키워드를 모두 실행한 후 즉시 리턴한다. panic 모드 실행 방식은 다시 상위함수에도 똑같이 적용되고 계속 콜스택을 타고 올라가서 결국엔 에러를 내고 종료하게 된다. 현재는 main이라서 종료되지만 만약 함수 내에서 panic함수를 쓸 경우 결국엔 main에서 종료된다.
```go
func main() {
	f, err := os.Open("1.txt")
	if err != nil {
		panic(err)
	}
	// main 마지막에 파일 close 실행
	defer f.Close()
	bytes := make([]byte, 1024)
	f.Read(bytes)
	println(len(bytes))
}
```
panic 함수에 의한 패닉 상태를 다시 정상으로 돌리는 방법도 있는데 recover함수가 그 역할을 한다.
```go
package main

import (
	"fmt"
	"os"
)
func main() {
	openFile("./test.txt")

	println("Done.")
}
// panic함수는 defer 함수를 다 실행시키고 리턴함을 이용
func openFile(fileName string) {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("open error", r)
		}
	}()
	f, err := os.Open(fileName)
	if err != nil {
		panic(err)
	}

	defer f.Close()
}
```

---
## goroutine
goroutine은 go 런타임이 관리하는 경량화 논리적 쓰레드이다. go라는 키워드를 통해 새러운 goroutine을 실행하고 goroutine은 비동기적으로 함수루틴을 실행하며 여러 코드를 동시에 실행하는데 사용된다.<br>
goroutine은 os 쓰레드보다 휠씬 가볍게 비동기 concurrent 처리를 구현하기 위하여 만든 것으로 기본적으로 go의 런타임이 자체 관리한다. 즉, os thread 1개에서 goroutine이 여러 개가 실행되곤 한다. go 런타임은 goroutine을 관리하면 go channel을 통해 goroutine간의 통신을 쉽게 할 수 있도록 한다.
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func test(msg string) {
	fmt.Println(msg)
	for i := 0; i < 10; i++ {
		fmt.Println(msg, i)
	}
}

func main() {
	var wait sync.WaitGroup
	wait.Add(2)
	go test("test")
	go func() {
		defer wait.Done()
		fmt.Println("go1")
	}()
	go func(msg string) {
		defer wait.Done()
		time.Sleep(5)
		fmt.Println(msg)
	}("msg")
	wait.Wait()
}
```
위처럼 익명함수를 이용해서 goroutine으로 비동기 처리할 수도 있다. waitgroup은 해당 개수(Add 메소드에 파라미터로 준 개수)만큼의 goroutine이 끝날 때까지 끝나길 기다린다.
<br>
go는 디폴트로 cpu 1개를 사용하고 여러 개의 goroutine은 cpu 1개를 concurrent하게 처리한다. 여러 개의 cpu를 이용해서 병렬처리하게 하려면 `runtime.GOMAXPROCS(2)`과 같은 함수를 호출하면 된다.
```go
func main() {
	runtime.GOMAXPROCS(4)
	var wait sync.WaitGroup
	wait.Add(2)
	go test("test")
	go func() {
		defer wait.Done()
		fmt.Println("go1")
	}()
	go func(msg string) {
		defer wait.Done()
		time.Sleep(5)
		fmt.Println(msg)
	}("msg")
	wait.Wait()
}
```