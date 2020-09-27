# Go study 2주차

## Go의 여러가지 자료구조
### String
* Go의 소스코드는 UTF-8로 되어있다.
* 유니코트 문자는 크기가 가변적 (1~6Byte)
  * https://blog.golang.org/strings
* rune type 자료형 (유니코드 글자 하나를 담는다)
* example 1
    ```go
    for _, r:= range "가나다" {
        fmt.Println(string(r), r)
    }
    ```
    ```go
    가 44032
    나 45208
    다 45796
    ```
* example 2
    ```go
    const nihongo = "日本語"
    for index, runeValue := range nihongo {
        fmt.Printf("%#U starts at byte position %d\n", runeValue, index)
    }
    ```
    ```
    U+65E5 '日' starts at byte position 0
    U+672C '本' starts at byte position 3
    U+8A9E '語' starts at byte position 6
    ```
* string은 읽기 전용이기 때문에, 다음과 같은건 할 수 없다.
```go
s := "가나다"
s[2]++
fmt.Printf(s)
```
* 슬라이스로 변형 가능 (byte, uint8의 가변 array)
* 연산자 지원 ( + , 읽기 전용이기 때문에 메모리 재할당 )
  * 이때 메모리는 어디로 가는가? [go에도 GC가 있을까?](https://engineering.linecorp.com/ko/blog/go-gc/)

### Slice
* C++의 vector<T>, Java의 ArrayList<T>
* 
#### 배열
```go
fruits := [3]string{"사과", "바나나", "토마토"}
for _, fruit := range fruits {
    fmt.Printf("%s는 맛있다.\n",fruit)
}
```
* `fruits := [...]string{"사과", "바나나", "토마토"}`도 가능

#### 슬라이스란?
* 생성
  * `var fruits []string`, `fruits := []string`
* make & resize
  * `fruits := make([]string, n)`

* random access 가능
* `slice[a:b]`는, 해당 slice의 `[a, b)`구간만을 가져옴.
* index가 넘어가면 `Panic` 발생 (에러를 반환시키고, 패닉은 잘 사용하지 않는다.)

* append(..)
  * 인자의 개수가 가변
  * slice에 slice는 직접 append할 수 없다.
    * f3 := append(f1, f2)    (X)
    * f3 := append(f1, f2...) (O)

* make([]type, len, cap) = 실제 나타나는 길이는 len이지만, cap만큼 메모리 할당
* make([]type, 0, cap) = cap 이상으로 개수가 넘지 않으면 realloc X
* slice 복사 : copy(dset, src)

* 포인터에 주의!
```go
x := []int{3, 4, 5}
y := x[:]
```
* 복사가 아니라 참조 이동이 발생한다.
* 삽입/삭제 함수가 존재하지 않는다.

### Map
* HT (Hash Table)
```go
var m map[KeyType]valueType
m := make(map[KeyType]ValueType)
m := map[keyType]valueType{}
```
* value, ok := m[key] 존재 여부 확인도 가능
* 맵의 비교 방법 : reflect.DeepEqual
* Go의 Map에서, KeyType은 상수나 문자열같열 불변 자료형만 들어갈 수 있다.

### I/O
* io.Reader, io.Writer
  * 화면, 파일, 소켓에 능계 없이 자료 R/W 가능

## Go의 함수
Go는 항상 Call by Value만 지원한다.
* Call by reference를 하려면, 포인터를 넘겨주어야 한다.

### Argument와 Return
* func FuncName(valName Type,, valName *[]Type) (int64, err) {...}
* if err := FuncName(); err != nil {...}

* Named return params
  ```go
  func FuncName(valName Type) (**n** int64, **err** error){
      var nw, err = fmt.Fprintln(~~)
      n+= int64(nw)
      if err!=nil{
          return
      }
  }
  ```
* 내부에 선언된 n과 err이 밖으로 튀어나감.

* func F(lines... string) (int64, err) {...}
* n,err = F(lines...)

### 값으로 취급되는 함수
* Python처럼 Go도 함수는 1급 객체 (1급 시민이라던데.. 여기선 citizen이라고 하는 듯)
* ex
```go
func add(a, b int) int {
    return a + b
}

add2 := func(a, b int) int {
    return a + b
}
```
* 람다처럼 사용 가능
```go
printHello := func(){
    fmt.Println("Hi")
}

printHello()
```

### Clouser
* 함수 바깥의 변수를 참조하는, 함수값
* 소스코드 보는게 이해가 빠름
```go
func nextValue() func() int{
    i := 0
    return func() int{
        i++
        return i
    }
}
```
```go
func main() {
    next := nextValue() // func() int가 리턴됨
    println(next()) // 1
    println(next()) // 2
    println(next()) // 3
}
```

### Named Type
* type runes []rune
* typedef임

```go
type BinOP func(int, int) int
func OpThreeAndFour(f BinOP){
    fmt.Println(f(3, 4))
}
```

* 함수 명명, 자료형 명명을 사용하고 함수 자체를 주고 받으면 높은 레벨의 추상화가 가능하다.

### Method
* 메서드를 사용하면 클래스 단위의 추상화가 가능하다.

```go
type MultiSet map[string]int

func (m MultiSet) Insert(val string) {
    m[val]++
}

func (m MultiSet) Erase(val string){
    if m[val] <= 1{
        delete(m, val)
    } else {
        m[val]--
    }
}

func (m MultiSet) Count(val string) int{
    return m[val]
}
```

* Public, Private는 함수 첫 글자가 대문자인지 소문자인지로 결정
* Public에 주석을 달면 godoc이 문서 만들어줌

## 구조체, serialize, interface
### 구조체, ENUM
```go
type stats int
type Task struct{
    title string
    status status
    due *time.Time
}
```

* ENUM
```go
const (
    UNKNOWN status = 0
    TODO    status = 1
    DONE    status = 2
)
```
```go
const (
    UNKNOWN status = iota
    TODO    status = iota
    DONE    status = iota
)
```
```go
const (
    UNKNOWN status = iota
    TODO
    DONE
)
```
```go
type ByteSize float64
const (
    _ = iota
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
)
```


### 직렬화
* json만들어줌

### 인터페이스
* 추상클래스 같은거
