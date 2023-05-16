# Concept Summnary

일단 언어에 대한 기본적인 지식을 쌓기 위해서 해당 사이트들 참고

https://gobyexample.com/

http://golang.site/Go/Basic﻿

https://gowebexamples.com/

go build를 하면 exe 실행파일로 만들어주고 go run하면 실행!! 예전에 수업에서 듣기론 go가 다른 python이나 js와 다르게 pip npm으로 패키지를 설치하는걸 go라는 커맨드로 다 할 수 있다고 했던 것 같다.

go를 실습해보기 위해선 goland를 이용했고(학생은 jetbrain 무료라서 이용)

기본적으로 변수 선언과 데이터 타입에 대해 공부를 시작했다
### variables
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
### condition

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
### loop

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
### functions

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
### closure
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
### collection
배열은 java랑 비슷한 것 같고 다른 언어랑 대체적으로 비슷한 것 같다.
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