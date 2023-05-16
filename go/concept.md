# Concept Summnary

일단 언어에 대한 기본적인 지식을 쌓기 위해서 해당 사이트들 참고

https://gobyexample.com/

http://golang.site/Go/Basic﻿

https://gowebexamples.com/

go build를 하면 exe 실행파일로 만들어주고 go run하면 실행!! 예전에 수업에서 듣기론 go가 다른 python이나 js와 다르게 pip npm으로 패키지를 설치하는걸 go라는 커맨드로 다 할 수 있다고 했던 것 같다.

go를 실습해보기 위해선 goland를 이용했고(학생은 jetbrain 무료라서 이용)

기본적으로 변수 선언과 데이터 타입에 대해 공부를 시작했다
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