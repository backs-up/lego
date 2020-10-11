# Go study 3주차

## Chap 6. 웹 애플리케이션 작성하기

### 웹서버

```go
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "Hello, 세계!")
    })
    log.Fatal(http.ListenAndService(":8080", nil))
}
```

매우 간단하게 간단한 웹 앱을 띄울 수 있다

### DAO (Data Access Object)

```go
// task를 식별하기 위한 자료형
type ID string

// task를 수행하기 위한 인터페이스
type DataAccess interface {
    Get(id ID) (task.Task, error)
    Put(id ID, t task.Task) error
    Post(t task.Task) (ID, error)
    Delete(id ID) error
}
```

### Response 구조체

RESTful 한 형태로 인터페이스를 구현해본다.
응답 구조체는 아래와 같은 형태를 띈다

```go
type Response struct {
    ID      ID              `json:"id,omitempty"`
    Task    task.Task       `json:"task"`
    Error   ResponseError   `json:"error"`
}
```

int64형 필드는 자바스크립트와의 호환을 고려하여 JSON 태그로 string을 달아주는 습관을 들이자.  
자바스크립트에서는 정수형이 없으므로 실수형으로 처리되며, 64비트 정수를 제대로 읽을 수 없다는 문제가 있다. (53비트 정도까지만 정확하게 읽을 수 있다)

### 메인 함수

```go
const pathPrefix = "/api/v1/task/"

func apiHandler(w http.ResponseWriter r *http.Request) {
    ...
}

func main() {
    http.HandleFunc(PathPrefix, apiHandler)
    log.Fatal(http.ListenAndServe(":8887", "nil")
```

이처럼 핸들러와 URI를 연결해준다.  
라우터를 이용하면 좀 더 편리하게 핸들러를 연결시킬 수도 있다.

### 라우터 사용
Gorilla Web Toolkit의 mux 등 라이브러리 이용
```go
const (
apiPathPrefix = "/api/v1/task/"
htmlPathPrefix = "/task/"
idPattern = "/{id:[0-9]+}"
)

func main() {
    r := mux.NewRouter()
    r.PathPRefix(htmlPathPrefix).
      Path(idPattern).
      Methods("Get").
      HandlerFunc(htmlHandler)
    
    s := r.PathPrefix(apiPathPrefix).Subrouter()
    s.HandleFunc(idPattern, apiGetHandler).Methods("GET)
    ...
    
    http.Handle("/", r)
    log.Fatal(http.ListenAndServe(":8884", nil))
}
```

### 에러 처리

#### 자료형을 이용한 에러 처리하기

여태까지 에러처리를 위해 이용한 error 자료형은 인터페이스 자료형이다.

```go
type error interface {
    Error() string
}
```

에러에 대한 부가정보를 얻어야 할 때는 다음과 같이 에러 메시지에 값을 담아준다.

```go
return fmt.Errorf("ID %d is negative", id)
```

그러나 해당 부가 정보를 프로그램 내에서 확인하기 위해서는 새로운 에러 자료형이 필요할 때가 있다.
error 자료형이 인터페이스라는 점을 이용해서 새로 정의한다.

```go
type ErrNegativeID ID

func (e ErrNegativeID) Error() String {
    return fmt.Sprintf("ID %d is negative", e)
}
```

#### 반복된 에러 처리 피하기

```go
if err := f(); err != nil {
    panic(err)
}
```

이와 같은 코드가 반복적으로 나와야 한다면, Must와 같은 이름의 함수를 만들어서 사용할 수 있다.

```go
// error 만을 리턴받을 때
func Must(err error) {
    if err != nil {
        panic(err)
    }
}

Must(f())

// 두 값을 리턴받을 때
func Must(i int64, err error) int64 {
    if err != nil {
        panic(err)
    }
    return i
}

parsed := Must(strconv.ParseInt("123", 10, 64))
```

#### panic 처리하기

- recover를 이용하여 처리 할 수 있다.

```go
func f() (i int) {
    defer func() {
        if r:= recover(); r != nil {
            fmt.Println("Recovered in f", r)
            i = -1
        }
    }

    ...
}
```


### 기타
- [HTML 템플릿](https://golang.org/pkg/html/template)
- 코드 리팩토링
    - 데이터 엑세스 인터페이스 / 구현체, 응답 자료형, 핸들러로 파일 구분 가능
- HTTP 파일 서버 ([http.FileServer](https://golang.org/pkg/net/http/#FileServer))


## Chap 7. 동시성

### 고루틴

```go
go f(x, y, z)
```
아래 두 흐름이 **메모리를 공유**하는 [논리적으로 별개의 흐름](https://tour.golang.org/concurrency/1)이 됨
1. f(x, y, z) 호출
1. 현재 함수의 흐름

### 병렬성과 동시성

- 병렬성 (Parallelism) : 물리적으로 별개의 업무를 수행하는 것
- 동시성 / 병행성 (Concurrency) : 물리적으로 두 흐름이 있지는 않으나, 동시에 두 가지를 수행하는 것

동시성이 있는 두 루틴은 서로 의존 관계가 없음.
> **동시성 O** : 커피 마시기 / 신문기사 읽기  
> **동시성 X** : 양말 신기 / 신발 신기

다운로드와 압축을 동시에 수행하는 등 여러 작업을 수행하기 위해서는 흐름에 대한 컨트롤이 필요함

-  sync.WaitGroup

```go
wg.Add(len(urls))
for _, url := range urls {
    go func(url string) {
        defer wg.Done()
        if _, err := download(url); err != nil {
            log.Fatal(err)
        }
    } (url)
}
wg.Wait()
```

wg 안에 카운터가 존재하고, `Wait()` 함수는 카운터가 0이 될때까지 기다린다.  
`wg.Add()` 로 카운터에 숫자를 더하며, `wg.Add(-1)` 혹은 `wg.Done()` 으로 숫자를 뺀다.  
고루틴 내부에서 `wg.Add()`를 실행할 경우에는 메인 고루틴이 `wg.Wait()`를 먼저 통과할 가능성이 있음에 유의한다. (레이스 컨디션)

### 채널

- 공유메모리를 이용하지 않고 통신할 수 있도록 하는 자료구조 (일급 시민)  
> Do not communicate by sharing memory; instead, share memory by communicating.
> [출처](https://blog.golang.org/codelab-share)

```go
// 채널 생성
c1 := make(chan int)
// 같은 객체를 가리키는 c2 채널 변수 생성
var chan int c2 = c1
// 자료 출력만 가능한 채널 (receive only 채널)
var <-chan int c3 = c1
// 자료 입력만 가능한 채널 (send only 채널)
var chan<- int c4 = c1

// 자료를 보낼 때
c1 <- 100
// 자료를 받을 때
data := <-c1 
```

#### 채널의 사용

보내는 고루틴과 받는 고루틴이 모두 주고받는 부분의 코드를 실행하고 있어야 전달이 이루어진다.  
즉, 서로 송수신하는 데이터의 개수를 알아야 한다.

```go
func Example_simpleChannel() {
    c := make(chan int)
    go func() {
        c <- 1
        c <- 2
        c <- 3
        close(c)
    }()
    for num := range c {
        fmt.Println(num)
    }
}
```

이를 개선하기 위해 반복문으로 데이터를 가져오면 송수신 할 데이터의 개수를 알지 못하더라도 된다.  
그러나 채널 객체 하나를 서로 주고받는 부분이 깔끔해 보이지 않는다.

```go
func Example_simpleChannel() {
    c := func() <-chan int {
        c := make(chan int)
        go func() {
            defer close(c)
            c <- 1
            c <- 2
            c <- 3
            close(c)
        }()
        return c
    }()
    for num := range c {
        fmt.Println(num)
    }
}
```

해당 부분은 이와같이 함수로 만들어 개선할 수 있다.  
또한 의도하지 않은 채널의 동작을 막을 수 있다. (receive only / send only)

#### 채널과 클로저 비교

- 클로저 이용
    ```go
    func FibonacciGenerator(max int) func() int {
        next, a, b := 0, 0, 1
        return func() int {
            next, a, b = a, b, a+b
            if next > max {
                return -1
            }
            return next
        }
    } 
   
    func ExampleFibonacciGenerator() {
        fib := FibonacciGenerator(15)
        for n := fib(); n >= 0; n = fib() {
            fmt.Print(n, ",")
        }
    }
    ```
- 채널 이용
    ```go
    func Fibonacci(max int) <-chan int {
        c := make(chan int)
        go func() {
            defer close(c)
            a, b := 0, 1
            for a <= max {
                c <- a
                a, b = b, a+b
            }
        }()
    return c
    }
  
    func ExampleFibonacci() {
        for fib := range Fibonacci(15) {
            fmt.Print(fib, ",")
        }
    }
    ```

채널 이용시 아래와 같은 장점이 있다.

1. 생성하는 쪽에서는 상태 저장 방법을 복잡하게 고민할 필요가 없다.
2. 받는 쪽에서는 for의 range를 이용할 수 있다.
3. 채널 버퍼를 이용하면 멀티코어를 활용하거나 입출력 성능상의 장점을 이용할 수 있다.

#### 버퍼와 함께 사용하는 채널

버퍼가 없다면, 보내는 고루틴과 받는 고루틴이 동시에 준비되어 있어야 한다.  
받는 쪽이 준비되어 있지 않아도 미리 버퍼에 보내둘 수 있다.

```go
c := make(chan int, 10)
```

- 장점
    - 버퍼를 이용하면 보내는 쪽과 받는 쪽의 코드가 균일한 속도로 수행되지 않고 두 고루틴 사이에 어느정도 격차가 생겨도 계속 동작할 수 있다. (성능 향상)  
- 단점
    - 논리적인 오류 등으로 버퍼가 가득 찰 경우에는 다른 고루틴이 채널에서 받아가지 않는 이상, 고루틴이 채널에 보내는 곳에서 멈춰있는다.
    - 이 때, 막히면 버퍼 크기를 더 늘리는 식으로 문제를 해결할 경우에는 코드가 점점 더 복잡해지고 예측 불가능하게 된다.
- **결론**
    - **버퍼 없는 채널로 동작하는 코드를 만들고, 필요에 따라 성능 향상을 위해 버퍼 값을 조절하자.**

#### 닫힌 채널

```go
val, ok := <- c
// 채널이 열려있다면 val = 받은 값, ok = true
// 채널이 닫혀있다면 val = 기본값(0, 빈 문자열 등), ok = false
```

- 채널이 닫힌 상태라면 받는 코드는 기다리지 않고 기본 값을 받아온다.  
- 이미 닫힌 채널을 또 닫는다면 패닉이 발생한다.
 
### 동시성 패턴

#### 파이프라인 패턴

```go
func PlusOne(in <-chan int) <-chan int {}
    out := make(chan int)
    go func() {
        defer close(out)
        for num := range in {
            out <- num + 1
        }
    }()
    return out
}

func ExamplePlusOne() {
    c:= make(chan int)
    go func() {
        defer close(c)
        c <- 5
        c <- 3
        c <- 8
    }()
    for num := range(PlusOne(PlusOne(c)) {
        fmt.Println(num)
    }
}
```

- 자연스럽게 출력을 입력으로 연결하여 일직선으로 연결된 파이프라인을 구성할 수 있다.
- 들어오는 데이터와 나가는 데이터 집중하여 문제를 풀 수 있다.
- 버퍼를 이용하면 성능상의 장점도 얻을 수 있다.
- 데이터를 보내는 쪽에서 채널을 닫지 않으면 파이프라인이 꼬인다는 것에 주의하자.

##### 파이프라인 합치기

```go
func Chain(ps ...IntPipe) IntPipe {
    return func(in <-chan int) <-chan int {
        c := in
        for _, p : range ps {
            c = p(c)
        }
        return  c
    }
}

PlusTwo := Chain(PlusOne, PlusOne)
```

- 바로 이용하지 않고 다른 곳에 넘길경우에 Chain 고계 함수를 이용할 수 있다.
 
#### 채널을 공유하여 팬아웃

> 팬아웃(Fan-out) : 게이트 하나의 출력이 게이트 여러 입력으로 들어가는 경우

파이프라인에서 앞의 과정에서는 결과물이 빨리 나오지만 뒤의 과정은 시간이 오래걸리는 경우에, 여러 곳에 결과물을 나누어 줄 수 있다.
```go
func main() {
    c := make(chan int)
    for i := 0; i < 3; i++ {
        go func(i int) { // i를 각 고루틴에 따로 넘겨주지 않으면 메인 고루틴의 i를 이용해 버리므로 유의할 것
            for n := range c {
                time.Sleep(1) // 다른 고루틴도 실행 할 수 있도록 컨텍스트 스위칭
                fmt.Println(i, n)
            }
        }(i)
    }
    for i := 0; i < 10; i++ {
        c <- i
    }
    close(c)
}
```

- 채널을 닫는 것은 broadcast 효과를 지닌다. (모두에게 신호를 전달할 수 있는 방법)

#### 팬인

> 팬인(Fan-in) : 하나의 게이트에 여러 개의 입력선이 들어가는 경우

```go
func FanIn(ins ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    wg.Add(len(ins))
    for _, in := range ins {
        go func(in <-chan int) {
            defer wg.Done()
            for num := range in {
                out <- num
            }
        }(in)
    }

    // 채널을 닫는 고루틴
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}

c := FanIn(c1, c2, c3)
```

- 보내는 고루틴에서 채널을 닫으면, 같은 채널을 여러 번 닫기 때문에 패닉이 발생한다.
- 채널을 닫기 위한 별도의 고루틴을 만든다.

#### 분산처리
```go
func Distribute(p IntPipe, n int) IntPipe {
    return func(in <-chan int) <-chan int {
        cs := make([]<-chan int, n)
        for i := 0; i < n; i++ {
            cs[i] = p(in)
        }
        return FanIn(cs...)
    }
}

// Distribute와 Chain을 함께 이용한 파이프라인
out := Chain(Cut, Distribute(Chain(Draw, Paint, Decorate), 10), Box)(in)
```

#### select

select를 이용해 동시에 여러 채널과 통신할 수 있다.

- 모든 case가 계산되며, 함수 호출 코드는 select를 수행할 때 모두 호출된다.
- 각 case는 채널에 입출력하는 형태가 되며, 막히지 않고 입출력이 가능한 case가 있으면 그 중 하나가 선택되에 입출력이 수행되고 해당 case의 코드만 수행된다.
- 모든 case에 입출력이 불가능할 때 default가 수행된다. default가 없다면, 어느 case 하나라도 가능해질 때까지 기다린다.

```go
select{
    case n := <-c1:
        fmt.Println(n, "is from c1")
    case n := <-c2:
        fmt.Println(n, "is from c2")
    case c3 <- f(): // c3 채널이 준비되어 있지 않더라도 f()는 호출된다.
        fmt.Println(n, "is from c3")
    default:
        fmt.Println("No channel is ready")
}
``` 

##### 팬인

고루틴을 여러 개 이용하지 않고도 팬인을 할 수 있다.

```go
select {
    case n:= <-c1: c<-n
    case n:= <-c2: c<-n
    case n:= <-c3: c<-n
}
```

- 닫힌 채널의 처리는 nil 값을 이용하면 깔끔하다. (p.264)

##### 채널을 기다리지 않고 받기

채널에 값이 있으면 받고, 없으면 스킵하도록 할 수 있다.

```go
select {
    case n:= <-c:
        fmt.Println(n)
    default:
        fmt.Println("Data is not ready. Skipping...")
}
```

##### 시간 제한 걸기

```go
select {
    case n := <-recv:
        fmt.Println(n)
    case send <- 1:
        fmt.Println("sent 1")
    case <-time.After(5 * time.Second)
        fmt.Println("No send and receive communication for 5 seconds")
        return
}
```

#### 파이프라인 중단하기

- done 채널을 하나 더 두어 보내는 고루틴에게 채널을 닫도록 신호를 줄 수 있다.(p.269)

#### 컨텍스트(context.Context) 활용하기

- 여러 고루틴에 종료 신호 이외에 다른 정보가 공유되어야 할 때 [context](!https://blog.golang.org/context) 패턴을 이용한다.

#### 요청과 응답 대응시키기 (p.272)

#### 동적으로 고루틴 이어붙이기 (p.274)

#### 기타 주의사항 (p.278)

### 경쟁 상태 (race condition)

- p.280
- sync 라이브러리, atomic 라이브러리를 사용해야 할 때 경쟁상태가 생길 수 있다.

### 문맥 전환(Context Switching)

Go 컴파일러는 주로 다음의 경우에 문맥 전환을 하는 코드를 생성 할 수 있다.

- 파일이나 네트워크 연산과 같이 시간이 오래 걸리는 입출력 연산이 있을 때
- 채널에 보내거나 받을 때
- go로 고루틴이 생성될 때
- 가비지 커넥션 사이클이 지난 뒤

## Chap 8. 실무 패턴

### 오버로딩

- 자료형에 따라 다른 이름 붙이기
    - 자료형에 따라 다른 함수의 이름을 붙이자
- 동일한 자료형의 자료 갯수에 따른 오버로딩
    - 가변인자를 사용하자
- 자료형 스위치 활용하기
    - 인터페이스로 인자를 받고, 메서드 내에서 자료형 스위치로 다른 자료형에 맞춰 다른 코드가 수행되게 할 수 있다. (5장)
- 다양한 인자 넘기기
    - 기본 값을 포함한 여러 설정을 넘겨야 할 경우에는, 이들을 묶은 구조체를 넘기자
- 인터페이스 활용하는 것이 더 나은 경우가 있다.

### 템플릿 및 제네릭 프로그래밍

#### 유닛 테스트

- 테이블 기반 테스트 (5장)

#### 컨테이너 알고리즘

- 인터페이스 활용

#### 자료형 메타 데이터 (p.296)

- 라이브러리를 개발 할 때 보자.

#### go generate

### 객체 지향

#### 다형성과 인터페이스

- 인터페이스로 구현 가능

#### 상속

- HasA 관계
    - 객체 구성(object composition)으로 구현 
- IsA 관계

#### 오버라이딩
#### 서브타입

### 디자인패턴