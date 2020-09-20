# Go study 1주차

## 우리는 Go를 왜 배워야 할까?

*성민이 형이 회사에서 쓰고 있기 때문..이 아니라,*

Go는 Python과 Java의 장점을 잘 섞어 놓은 언어다.

- Go는 놀랄 정도로 단순하다.
- 성능이 매우 좋다.(컴파일 속도가 C, C++보다 빠르며, 성능은 Node보다 훨씬 좋다)
- 고루틴을 활용한 비동기적 개발이 좋다.
- 정적언어다.

실무에서 쓰는 이유는 적은 리소스로 더 많은 일을 할 수 있기 때문이라고 생각한다.

**추가적으로 흥미로운 아티클들을 찾았다.**

- [https://medium.com/@EJSohn/번역-코딩-인터뷰에서-golang을-사용해야하는-이유-8b638ab33068](https://medium.com/@EJSohn/%EB%B2%88%EC%97%AD-%EC%BD%94%EB%94%A9-%EC%9D%B8%ED%84%B0%EB%B7%B0%EC%97%90%EC%84%9C-golang%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0-8b638ab33068)
- [https://edykim.com/ko/post/ipify-to-30-billion-and-beyond/](https://edykim.com/ko/post/ipify-to-30-billion-and-beyond/)
- [https://sangwook.github.io/2014/07/06/farewell-node-js-tj-holowaychuk.html](https://sangwook.github.io/2014/07/06/farewell-node-js-tj-holowaychuk.html)

개인적으로 Node.js를 사용하고 있는 입장에서, Go는 매력적인 선택지가 될 수 있다고 생각한다.

다만, 시스템 프로그래밍에 가까워지는 느낌이다.

## A Tour of Go

이 사이트가 처음 Go를 접하는 사람에게 아주 유용할 꺼 같다.

Go의 문법을 빠르게 익힐 때 이 사이트를 주로 사용한다고 한다.

[https://tour.golang.org/list](https://tour.golang.org/list)

## Chap 1. Go 시작하기

### 변수 선언하기

```go
var x int = 1
// x := 1
```

그냥 사람이 말하듯 `변수 x는 int형이다` 와 같은 느낌

참고로 세미콜론 안 찍어도 된다.

`const` 키워드가 존재하며 `:=` 을 통해서는 선언할 수 없다.

```go
const hi = "hi"
```

### Basic Types

엄청 많다.

추가적으로 int, uint, uintptr은 보통 32비트 시스템에서는 32비트지만, 64비트에서는 64비트라고 한다.

```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

- Zero Values

    초기 값 없이 선언된 경우, zero value라는 게 주어지게 되며, 각 타입에 따라

    `0` : 숫자 타입

    `false` : boolean 타입

    `""` : string 타입이 주어지게 된다.

또한, Type conversion도 가능하지만 명시적일 때만 가능하다.

### Type Inference

이런 식으로 타입이 명확한 것은 생략할 수 있다.

```go
var i = 10
var p = &i

i := 10
p := &i
```

또한 변수에 정적으로 `자료형`이 고정된다. (C++과 동일한 메커니즘이다)

### Function

함수는 이렇게 구현한다. (param에 변수명 다음 타입이 나오는 것에 유의하자!)

반환형을 param  옆에 덩그러니 적어 주는 것이 포인트

```go
func fac(n int) int {}

// func function1(x int, y int) {}

func function1(x, y int) {}
```

놀랍게도 여러 개를 반환하는 것도 가능하다.

```go
func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
}
```

심지어 이름을 붙여서 반환하는 것도 된다. 

(args 없이  return하는 것을 "naked return"이라고 한다. 
→ 이런 방식은 가독성을 해치기 때문에 추천하지 않는다고 한다!)

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}
```

### Control

if, for와 같은 구조에 소괄호가 생략되어 있다(상당히 파이썬스럽기도 하다)

추가적으로 `while`문이 없다.

```go
// 참고로 i 변수는 반복문을 벗어나면 소멸된다. 
func facItr(n int) int {
	result := 1
	for i := 2;i <=n; i++ {
		result *= i
	}
	return result
}
```

swift에서도 이런 문법을 봤었는데, Go에서도 된다고 한다.

```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}
```

## Chap 2. Go 환경설정 하기

### GOROOT? GOPATH?

심플하면서도 깔끔한 설명이다. GOPATH를 잘 설정하자.

[https://medium.com/chequer/goroot와-gopath-77f44cbaa1d8](https://medium.com/chequer/goroot%EC%99%80-gopath-77f44cbaa1d8)

여담으로 아래 나오는 package를 찾을 때, 
$GOROOT/pkg(표준 패키지) | $GOPATH/pkg(3rd Party 혹은 사용자 패키지)를 찾는다고 한다.

### GO Module(GO Mod)

나도 예전에 Go를 공부할 때 GOPATH에 대한 이해가 없어서 던졌던 기억이 있었는데,

이제는 `Go Module`이라는 게 정식적으로 나왔고, GOPATH는 역사의 뒤안길로 사라지고 있다고 한다.

간단하게 이해할 수 있을 법한 내용을 넣어 보았다.

- [https://velog.io/@taesunny/Go-Module-정리-golang-wiki-정리](https://velog.io/@taesunny/Go-Module-%EC%A0%95%EB%A6%AC-golang-wiki-%EC%A0%95%EB%A6%AC)
- [https://blog.kesuskim.com/archives/migrate-gopath-proj-into-gomod/](https://blog.kesuskim.com/archives/migrate-gopath-proj-into-gomod/)

### Package

Go에서 기본 제공하는 go 도구를 이용하면, 디렉터리 하나가 한 패키지가 된다.

메인 패키지(bin에 설치됨)와 라이브러리 패키지가 존재한다.

다른 패키지에서 함수를 이용할 수 없게 하려면 함수의 첫 글자를 소문자로 적는다. (접근 제한자 느낌)

- 패키지 내부의 things에 대한 접근에 대한 이야기

    패키지 내에는 함수, 구조체, 인터페이스, 메서드 등이 존재하는데, 이들의 이름(Identifier)이 첫 문자를 대문자로 시작하면 이는 public 으로 사용할 수 있다. 즉, 패키지 외부에서 이들을 호출하거나 사용할 수 있게 된다.

- Package 이름이 동일한 경우

    만약 패키지 이름이 동일할 수 있는데, 이런 경우에는 alias를 통해 구분할 수 있다고 한다.

    ```go
    import (
        mongo "other/mongo/db"
        mysql "other/mysql/db"
    )

    func main() {
        mondb := mongo.Get()
        mydb := mysql.Get()
        //...
    }
    ```

- Main Package에 관하여

    Main이라는 이름을 가진 패키지는 실행 파일로 만들기 때문에, Main이라는 이름을 가진 Package, function을 만들 때는 주의해야 한다.

import할 때, 빈 칸을 띄우면 다른 순서로 정렬이 된다.

(gofmt가 정렬할 때)

```go
import (
	"g"
	"d"

	"k"
	"a"
	"f"
)
```

```go
import (
	"d"
	"g"

	"a"
	"f"
	"k"
)
```

라이브러리 이름은 너무 일반적인 이름은 피하되, 너무 길게 적지 않는다.(디렉토리가 의미를 뜻하기 때문)