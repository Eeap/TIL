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

#### array
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

#### slice
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

#### Map
