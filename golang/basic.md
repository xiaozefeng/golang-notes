# Golang 基础

- golang 中的 `空` 用 `nil` 表示

## 定义变量

- 变量类型写在变量名之后
- 用 `_` 表示忽略结果
- 大写开头变量名是 public ,  小写开头的变量名是 private
- 没有char , 只有rune (32位)

### 变量类型

```go
// uint8 uint16 uint uint32 uint64 uint
// int8 int16 int int32 int64 
// bool string 
// float32 float64
// byte rune (字符)

// 初始值
// int => 0
// float32 => 0
// string => ""
// bool => false
```



### 变量初始化

```go
// zreo value
	var a int
	var b bool
	var c string
	var d float32
// initial value
	var a int = 1
	var b bool = true
	var c string = "hello world"
	var d float64 = 28.90
// type deduction
	var a, b, c, d = 1, true, "yes", 11.6
// shorter
	a, b, c, d := 1, true, "hello", 11.9
// package variable
var (
	B  = 1
	KB = 1024 * B
	MB = 1024 * KB
	GB = 1024 * MB
)
```



### 类型转换

> go语言只有强制类型转换

```go
// 数字转换
var a int = 1
var b float64 = (float64)a 

// 字符串和数字互转
result,err = strconv.Atoi("5")
if err != nil{
  	fmt.Println("转换错误", err)
}else {
  	fmt.Println(result)
}

str := strconv.Itoa(result)
```



### 常量

```go
const (
  B  = 1 << (10 *iota)
	KB 
	MB
	GB
)

// or
const filename = "abc.txt"

// 特殊的常量 iota
const (
  c = iota
  java
  python
	golang
)
// c = 0 , java = 1 , python = 2 , golang = 3

```



## 分支 branch

### if

- if 语句不需要 `()`
- if 语句中可以赋值 ，但是在if语句中赋值的变量作用域只在这个if语句中

```go
if contents, err := ioutil.ReadFile("abc.txt"); err != nil {
		fmt.Println(err)
} else {
		fmt.Println(string(contents))
}
```



### switch

- switch 会自动 `break` 除非加 `fallthrough`
- swithc 后面可以没有表达式，在case 中加入条件即可

```go
// 第一种用法
func eval(a, b int, op string) int {
	var result int
	switch op {
	case "+":
		result = a + b
	case "-":
		result = a - b
	case "*":
		result = a * b
	case "/":
		result = a / b
	default:
		panic("unsupported operator" + op)
	}
	return result
}
// 第二种用法
func grade(score int) string {
	g := ""
	switch {
	case score < 60:
		g = "F"
	case score < 80:
		g = "C"
	case score < 90:
		g = "B"
	case score < 100:
		g = "A"
	default:
		panic(fmt.Sprintf("Wrong score : %d", score))
	}
	return g
}
```



## 循环

- `for` 语句可以 不加括号
- `for` 可以省略起始条件，结束条件，递增表达式

```go
// 没有起始条件
func convertToBin(n int) string {
	result := ""
	for ; n > 0; n /= 2 {
		lsb := n % 2
		result = strconv.Itoa(lsb) + result
	}
	return result
}
// 没有起始条件和递增条件
func printfile(filename string) {
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}

	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
}
```



### 遍历 数组，切片，Map

```go
// 遍历数组 和切片
for i:=0; i< len(arr);i++{
    fmt.Println(i, arr[i])
}
// 第二种
for index,value := range arr{
    fmt.Println(i, arr[i])
}

// 遍历 map
for key, value := range map{
  fmt.Println(key, value)
}

```





## 函数 func

- 使用 `func` 关键字定义函数
- 与定义变量一样, 定义函数参数时也是 变量名在前面，变量类型在后
- `golang` 中函数可以有多个返回值

```go
// 多返回值 , 给返回值命名
// 13 /3 = 4 ... 1
func div(a, b int) (q, r  int) {
	q = a / b
  r = a % b
  return q , r
}

// 第二个值一般用来表示是否有错误
func Eval(a int, b int, op string) (int, error) {
	switch op {
	case "+":
		return a + b, nil

	case "-":
		return a - b, nil
	case "*":
		return a * b, nil
	case "/":
		return a / b, nil
	default:
		return -1, fmt.Errorf("unsupported operation: %s", op)
	}
}
```



###  可变参数

```go
func sum(numbers ...int) int {
	sum := 0
	for _, v := range numbers {
		sum += v
	}
	return sum
}

// 调用代码
fmt.Println(sum(1, 2, 3, 4, 5))
```



### 参数传递是 引用传递 or  值传递 ?

- go 语言中只有`值传递`



### 函数式编程

- 函数是 `一等公民` 可以作为变量，参数或者返回值

```go
func apply(op func(int, int) int, a, b int) int {
	return op(a, b)
}
func pow(a, b int) int {
	return int(math.Pow(float64(a), float64(b)))
}
// 调用代码
fmt.Println(apply(pow, 3, 4))
// or
fmt.Println(
  apply(func(a, b int) int {
    return int(math.Pow(float64(a), float64(b)))
  }, 3, 4),
)
```

#### 闭包

- 返回一个函数，函数中引用了一个局部变量，当前函数之前完成后，本来局部变量应该被销毁了，但是由于返回的函数中引用了这个局部变量，导致这个局部变量不会被销毁

```go
// 一个简单的闭包
func adder() func(int) int {
	sum := 0
	return func(v int) int {
		sum += v
		return sum
	}
}
// 调用
func main() {
	a := adder()
	for i := 0; i < 10; i++ {
		fmt.Printf("0 + ... %d = %d\n", i, a(i))
	}
}
```



##### 斐波那契数列

```go
func fibonacci() func() int {
	a, b := 0, 1
	return func() int {
		a, b = b, a+b
		return a
	}
}
```



####   为函数实现接口

```go
// 返回自定义的 fibGen
func fibonacci() fibGen {
	a, b := 0, 1
	return func() int {
		a, b = b, a+b
		return a
	}
}

// 定义函数类型
type fibGen func() int

// fibGen 实现 Read 接口, fibGen就可以作为一个 Reader来使用了
func (this fibGen) Read(p []byte) (n int, err error) {
	next := this()
	if next > 10000 {
		return 0, io.EOF
	}
	s := fmt.Sprintf("%d\n", next)
	return strings.NewReader(s).Read(p)
}

// 读取 reader 读取reader 中的内容
func read(r io.Reader) {
	scanner := bufio.NewScanner(r)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
}
```



#### 函数作为参数

```go
type treeNode struct {
	value       int
	left, right *treeNode
}
//中序遍历
func (this *treeNode) traverseFunc(f func(*treeNode)) {
	if this == nil {
		return
	}
	this.left.traverseFunc(f)
	f(this)
	this.right.traverseFunc(f)
}
// 统计节点个数
nodeCount := 0
root.traverseFunc(func(n *treeNode) {
  nodeCount++
  fmt.Print(n.value, "")
})
fmt.Println("nodeCount:", nodeCount)
```





## 指针 Pointer

- 指针不能运算

```go
a := 3
pa := &a
*pa = 5
fmt.Println(a)
```





## 数组、切片、容器

### 数组

- 数组是值类型
- [5]int  和[10]int 是不同的 `类型`
- go语言中一般不直接使用 数组

```go
// 定义数组
var arr1 [5]int
arr2 := [3]int{1, 2, 3}
arr3 := [...]int{2, 4, 6, 8}
fmt.Println(arr1)
fmt.Println(arr2)
fmt.Println(arr3)

var grid [4][5]int
fmt.Println(grid)
```



### 切片 Slice

- Slice 本身是没有数据的，是对底层数组的一个视图 (view)
- Slice 可以向后扩展，不可以向前拓展
- s[i] 不可超过 len(s) , 向后不可以超越底层数组 cap(s)

```go
// create slice
// update slice and look the origin array
// reslice

arr := [...]int{1, 2, 3, 4, 5, 6, 7}
fmt.Println("arr=", arr)
fmt.Println("arr[:6] =", arr[:6])
fmt.Println("arr[2:] =", arr[2:])
s1 := arr[2:6]
fmt.Println("s1 =", s1)
s2 := arr[:]
fmt.Println("s2 =", s2)

fmt.Println()
fmt.Println("Update slice s1")
updateSlice(s1)
fmt.Println("s1 =", s1)
fmt.Println("arr =", arr)
fmt.Println("Update slice s2")
updateSlice(s2)
fmt.Println("s2 =", s2)
fmt.Println("arr =", arr)

fmt.Println("Reslice s2")
s2 = s2[:5]
fmt.Println("s2 =", s2)
s2 = s2[2:]
fmt.Println("s2 =", s2)
fmt.Println("Extending slice")
arr[0], arr[2] = 0, 2
s1 = arr[2:6]
s2 = s1[3:5]
fmt.Println("s1= ", s1)
fmt.Println("s2= ", s2)
```



### Slice 操作

- Slice `append` 操作在 `cap` 不足时会自动扩容



#### 创建 Slice

```go
// 第一种方式
var s []int // slice zero value is nil
// 第二种方式
arr := [5]{1,2,3,4,5}
s := arr[:]
// 第三种方式  10 为长度(len)， 20为容量(cap), len 和 cap可以不指定
s := make([]int, 10, 20)
```



#### 打印 Slice 

```go
func printSlice(name string, s []int) {
	fmt.Printf("name=%s, slice=%v, len(s)=%d, cap(s)=%d\n", name, s, len(s), cap(s))
}
```



#### 添加元素 `append`

```go
var s []int
for i:=0 ; i <100 ; i++ {
  s = append(s, 2 * i + 1)
}
// 神奇的地方在于，第一次添加 s == nil ，append 操作也不会报错, 并且能添加成功
```



#### 拷贝 copy

```go
s1 := []int{2, 4, 6, 8}
s2 := make([]int, 10, 20)

fmt.Println("Copying slice")
copy(s2, s1)
printSlice("s1", s1)
printSlice("s2", s2)
```



#### 删除 Slice

```go
// 删除头部
s3 := s2[1:]
// 删除尾部
s3 := s2[:len(s2)-1]
// 删除中间 这里是第4个元素
s3 := append(s2[:3], s2[4:]...)
```



### Map

- map 使用了哈希表, 必须可以比较相等
- 除了 slice , map, function 的内建类型都可以作为 key 
- Struct 不包含 slice ，map，fucntion 也可以作为 key

#### 创建 Map

```go
m := map[string]string{
	"name":    "rose",
}

var m1 map[string]int      // m1 == nil
m2 := make(map[string]int) // m2 == empty map
fmt.Println(m, m1, m2)		
```


#### 遍历 Map

-  遍历时不能保证`key`的顺序

```go
for k, v := range m{
  fmt.Println(k,v)
}
```



#### 根据key 获取值

- `key ` 不存在时获取的是 `value ` 的 zreo value

```go
m := map[string]string{
	"name":    "rose",
}

name := m["name"]
fmt.Println(name)

if n,ok := m["n"]; ok {
  fmt.Println(n)
}else {
  fmt.Println("找不到key对应的value")
}
```



#### 根据key 删除 

```go
delete(m , "key")
```



## 字符串操作

- `rune  == int32`  相关于 `Java` 中的`char`

- 获取字符串的字符数量使用 `len` 是不准确的, 使用 `utf8.RuneCountInString(s)`  能准确获取字符数量

- 使用 `len` 获取的只是字符串的字节长度

- 字符串和 byte 的相互转化   

   ```go
  var s string = "hello world"
  bytes := []byte(s)
  s = string(bytes)
  ```

- 多行字符串

  ```go
  s := `abc "d"
  kkk
  123
  p`
  ```

  

### 字符串常用操作

- Fields , Split, Join
- Contains, Index
- Tolower, ToUpper
- Trim, TrimRight, TrimLeft

```go
fmt.Println("compare")
fmt.Println(
  strings.Compare("a", "b"), // -1
  strings.Compare("b", "a"), // 1
  strings.Compare("b", "b"), // 1
)

fmt.Println("contains")
fmt.Println(
  strings.Contains("seafood", "food"), //true
  strings.Contains("seabar", "bar"),   //true
  strings.Contains("seafood", ""),     //true
  strings.Contains("", ""),            //true
)

fmt.Println("contains any")
fmt.Println(
  strings.ContainsAny("team", "i"),        // false
  strings.ContainsAny("failure", "u & i"), //true
  strings.ContainsAny("foo", ""),          // false
  strings.ContainsAny("", ""),             // false
)

fmt.Println("contains rune")
fmt.Println(
  strings.ContainsRune("aardvark", 97), // true
  strings.ContainsRune("timeout", 97),  //false
)

fmt.Println("count")
fmt.Println(
  strings.Count("cheese", "e"), // 3
  strings.Count("five", ""),    //5
)

fmt.Println("equals fold")
fmt.Println(
  strings.EqualFold("Go", "go"),         // true
  strings.EqualFold("Golang", "goLang"), // true
  strings.EqualFold("JAVA", "Java"),     // true
)

fmt.Println("filed")
fmt.Println(
  strings.Fields(" foo bar baz  "),
)

fmt.Println("filed func")
fmt.Println(
  strings.FieldsFunc("foo1;bar2;baz3...123", func(r rune) bool {
    return !unicode.IsLetter(r) && !unicode.IsNumber(r)
  }),
)

fmt.Println("has prefix")
fmt.Println(
  strings.HasPrefix("Gopher", "Go"),
  strings.HasPrefix("Gopher", "C"),
  strings.HasPrefix("Gopher", ""),
)
fmt.Println("has suffix")
fmt.Println(
  strings.HasSuffix("Amigo", "go"),  //true
  strings.HasSuffix("Amigo", "O"),   // false
  strings.HasSuffix("Amigo", "Ami"), //false
  strings.HasSuffix("Amigo", ""),    // true
)

fmt.Println("index")
fmt.Println(
  strings.Index("chicken", "ken"),
  strings.Index("chicken", "dir"),
)
fmt.Println("index any")
fmt.Println(
  strings.IndexAny("chicken", "aeiouy"),
  strings.IndexAny("crwth", "aeiouy"),
)

s := []string{"foo", "bar", "baz"}
fmt.Println("join")
fmt.Println(
  strings.Join(s, ","),
)

fmt.Println("map")
rot13 := func(r rune) rune {
  switch {
    case r >= 'A' && r <= 'Z':
    return 'A' + (r-'A'+13)%26
    case r >= 'a' && r <= 'z':
    return 'a' + (r-'a'+13)%26
  }
  return r
}
fmt.Println(
  strings.Map(rot13, "'Twas brillig and the slithy gopher..."),
)

fmt.Println("replace")
fmt.Println(
  strings.Replace("oink oink oink", "k", "ky", 2),
  strings.Replace("oink oink oink", "oink", "moo", -1),
)

fmt.Println("split")
fmt.Println(
  strings.Split("a,b,c", ","),
  strings.Split("a man a plan a canal panama", "a"),
  strings.Split(" xyz ", ""),
  strings.Split("", "123"),
)

fmt.Println("split after")
fmt.Println(
  strings.SplitAfter("a,b,c", ","),
)

fmt.Println("title")
fmt.Println(
  strings.Title("her royal highness"),
)


```



## 面向对象

### 结构体

- go 语言仅仅支持封装，不支持继承和多态



```go
// 定义struct
type treeNode struct {
	value       int
	left, right *treeNode
}
var root treeNode
fmt.Println(root)
root = treeNode{value: 3}
root.left = &treeNode{}
root.right = &treeNode{5, nil, nil}
root.right.left = new(treeNode)
root.left.right= createNode(2)
fmt.Println(root)

// 没有构造函数 ， 使用工厂函数
func createNode(value int) *treeNode {
	return &treeNode{value: value}
}
```



#### 给 struct 定义方法

```go
// 值接收者, 不能改变对应的值
func (this treeNode) print() {
	fmt.Println(this.value)
}
// 指针接收者, 可以改变对应的状态
func (this *treeNode) setValue(value int) {
	this.value = value
}
```



#### 值接受值 vs 指针接受者

- 值接受者是 go 语言专有
- 值/ 指针 接收者 均可接收值/指针
- 建议: 一致性，能用指针接收就不用值接收



### 封装 (golang 封装是以 package 为单位的)

- 首字母大写表示 public ,首字符小写表示 private
- 同一个 package 内，只能有一个 包名





## 扩展已有类型 (其他语言中一般使用继承)

#### 定义别名

```go
type Queue []int

func (this *Queue) Push(v int ){
  *this = append(*this, v)
}

func (this *Queue ) Pop()int {
  head:= (*this)[0]
  *this = *this[1:]
  return head
}
```



#### 使用组合

```go
type Array struct {
  data []int
  size int 
}

func (this *Array) Add(e  int ){
  this.data[this.size] = e 
  this.size ++
}
```



### 接口 `interface`

- golang 中实现接口是隐式的
- 只要一个 struct 的拥有接口锁声明的方法，那可以说这个 struct 实现了这个接口
- 接口变量自带指针
- 接口变量同样采用值传递，几乎不需要使用接口的指针
- 指针接收者实现只能以指针方式使用，值接收者都可

#### duck typing 

- “当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。”
- 我们并不关心对象是什么类型，到底是不是鸭子，只关心行为。



#### 一个接口的例子

```go
// mock retriever 
package mock

type Retriever struct {
	Content string
}

func (this Retriever) Get(url string) string {
	return this.Content
}

// real retriever 
package real

import (
	"net/http"
	"net/http/httputil"
	"time"
)

type Retriever struct {
	UserAgent string
	TimeOut   time.Duration
}

func (this Retriever) Get(url string) string {
	resp, err := http.Get(url)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()
	result, err := httputil.DumpResponse(resp, true)

	if err != nil {
		panic(err)
	}
	return string(result)
}


// 调用代码
package main

import (
	"fmt"

	"github.com/xiaozefeng/just-go/oop/interface/retriever/mock"
	"github.com/xiaozefeng/just-go/oop/interface/retriever/real"
)

func main() {
	var r Retriever
	r = mock.Retriever{"this is a fake baidu"}
	fmt.Println(download(r))
	r = real.Retriever{}
	fmt.Println(download(r))
}

type Retriever interface {
	Get(url string) string
}

func download(r Retriever) string {
	return r.Get("http://www.baidu.com")
}

```



#### Switch 判断接口类型

```go
func inspect(r Retriever) {
	fmt.Printf("%T, %v\n", r, r)
	switch v := r.(type) {
	case mock.Retriever:
		fmt.Println("Contents:", v.Content)
	case real.Retriever:
		fmt.Println("UserAgent:", v.UserAgent)
	}
}
```



#### Type Assertion

```go
if realRetriever, ok := r.(real.Retriever); ok {
  fmt.Println(realRetriever.UserAgent)
}else {
  fmt.Println("not a real retriever.")
}
```



#### 接口变量里面有什么

- 实现者的类型
- 实现者的值 或者 指针





#### 接口的组合

```go
type Retriever interface {
	Get(url string) string
}

type Poster interface {
	Post(url string, form map[string]string) string
}

type RetrieverPoster interface {
	Retriever
	Poster
}
 
// 只用实现了 Post(url string, form map[string]string) string 和 Get(url string) string 
// 相当于实现了三个接口
```



#### 系统标准接口

##### Stringer 接口  (相当于 toString)

```go
type Stringer interface {
  Stringer() string
} 
// 实现
func (this Retriever) String() string {
	return fmt.Sprintf("Retriever: {Contents=%s}", this.Content)
}
```



##### Reader & Writer

- 在 `golang` 中 文件，字符串， 网络 以及一些实现 `Read`  和 `Write` 方法的 `struct` 都可以作为` Reader` 和 `Writer` 使用

```go
type Reader interface {
  Read(p []byte)(n int, err error)
}

type Writer interface {
  Write(p []byte)(n int, err error)
}
```



### 资源管理 & 错误处理

#### defer

- `defer ` 确保调用在函数结束时发生
- 一个函数有多个 `defer ` 语句时， 遵循 `先进后出` 的原则
- defer 后面可以是一个普通函数，也可以是一个匿名函数

```go
func tryDefer() {
	defer fmt.Println(1)
	defer fmt.Println(2)
	fmt.Println(3)
}
// 3 2 1 
```

#### panic

- `panic ` 相当于 throw new RuntimeException(), 停止当前函数执行，一直向上返回，执行每一层的 `defer`
- 如果 panic 时 没有遇见 `recover`, 程序退出

#### recover

- recover 仅能在 `defer` 中调用, recover() 可以获取到 `panic` 的值, 如果无法处理，还可以重新 panic

```go
func tryRecover() {
	defer func() {
		r := recover()
		if err, ok := r.(error); ok {
			fmt.Println("Error occurred: ", err)
		} else {
			panic(r)
		}
	}()
	panic(errors.New("this is a panic"))
}
```



#### 处理http server 异常

```go
type appHandler func(writer http.ResponseWriter, request *http.Request) error

// 全局异常处理, 最终返回能够被 http.HandleFunc 所能处理的函数类型
func globalErrorHandling(handler appHandler) func(http.ResponseWriter, *http.Request) {
  return func(writer http.ResponseWriter, request *http.Request) {
    err := handler(writer, request)
    if err != nil {
      log.Printf("Error handing request: %s\n", err.Error())
      status := http.StatusOK
      switch {
        case os.IsNotExist(err):
        	status = http.StatusNotFound
        default:
        	status = http.StatusInternalServerError
      }
      http.Error(writer, http.StatusText(status), status)
    }
  }
}

// 函数入口
func main() {
  http.HandleFunc("/list/", globalErrorHandling(handlerFileList))
  http.ListenAndServe(":8080", nil)
}

func handlerFileList(writer http.ResponseWriter, request *http.Request) error {
  path := request.URL.Path[len("/list/"):]
  file, err := os.Open(path)
  if err != nil {
    return err
  }
  defer file.Close()

  all, err := ioutil.ReadAll(file)
  if err != nil {
    return err
  }
  writer.Write(all)
  return nil
}

```

#### error vs panic

- 意料之中的用 `error` ， 如文件打不开
- 意料之外的采用 panic, 如数组越界



## 测试

### 表格驱动测试

- 测试数据与测试逻辑相分离
- 明确的出错信息
- 可以部分失败

```go
// 进入文件目录 运行测试 go test -v  -run TestNoneRepeating
func TestNoneRepeating(t *testing.T) {
	tests := []struct {
		arg string
		ans int
	}{
		// Normal case
		{"abcabcbb", 3},
		{"pwwkew", 3},

		// Edg case
		{"", 0},
		{"b", 1},
		{"bbbbb", 1},
		{"abcabcd", 4},

		// Chinese case
		{"一二三二一", 3},
		{"这里是广州", 5},
	}

	for _, tt := range tests {
		if actual := lengthOfNoneRepeatingSubStr(tt.arg); actual != tt.ans {
			t.Errorf("call lengthOfNoneRepeatingSubStr(%s) got %d, expected : %d",
				tt.arg, actual, tt.ans)
		}
	}
}
```



#### 性能测试

```go
// 运行性能测试 go test -bench  BenchmarkNoneRepeating
func BenchmarkNoneRepeating(b *testing.B) {
	s := "这里是广州"
	ans := 5
	for i := 0; i < b.N; i++ {
		if actual := lengthOfNoneRepeatingSubStr(s); actual != ans {
			b.Errorf("call lengthOfNoneRepeatingSubStr(%s) got %d, expected : %d",
				s, actual, ans)
		}
	}
}
```

#### 性能耗时长的点

```bash
# 生成性能分析文件 cpu.out
go test -bench  BenchmarkNoneRepeating -cpuprofile=cpu.out
# 进入交互式命令行
go tool pprof cpu.out
# 输入 web 可以看到 性能分析图 需要安装Graphviz
```



### HTTP 测试

```go 
import (
	"errors"
	"fmt"
	"io/ioutil"
	"net/http"
	"net/http/httptest"
	"os"
	"strings"
	"testing"
)

func errorPanic(writer http.ResponseWriter, request *http.Request) error {
	panic("this is a panic")
}

func errorUserError(writer http.ResponseWriter, request *http.Request) error {
	return myUserError("user error")
}

func errorNotFound(writer http.ResponseWriter, request *http.Request) error {
	return os.ErrNotExist
}

func errorPermission(writer http.ResponseWriter, request *http.Request) error {
	return os.ErrPermission
}

func errorUnknown(writer http.ResponseWriter, request *http.Request) error {
	return errors.New("this is a unknown error")
}

func noError(writer http.ResponseWriter, request *http.Request) error {
	fmt.Fprintf(writer, "no error")
	return nil
}

var tests = []struct {
	handler appHandler
	code    int
	message string
}{
	{errorPanic, 500, "Internal Server Error"},
	{errorUserError, 400, "user error"},
	{errorNotFound, 404, "Not Found"},
	{errorPermission, 403, "Forbidden"},
	{errorUnknown, 500, "Internal Server Error"},
	{noError, 200, "no error"},
}

//通过假的 http request 和 假的 http response
func TestGlobalErrorHanding(t *testing.T) {
	for _, tt := range tests {
		f := globalErrorHandling(tt.handler)
		response := httptest.NewRecorder()
		request := httptest.NewRequest(http.MethodGet, "http://www.baodu.com", nil)
		f(response, request)
		b, _ := ioutil.ReadAll(response.Body)
		body := strings.Trim(string(b), "\n")
		if response.Code != tt.code || body != tt.message {
			t.Errorf("expected (%d, %s); got (%d, %s)", tt.code, tt.message, response.Code, body)
		}
	}
}

// 真实的启动一个服务器
func TestGlobalErrorHandingInServer(t *testing.T) {
	for _, tt := range tests {
		f := globalErrorHandling(tt.handler)
		server := httptest.NewServer(http.HandlerFunc(f))
		response, _ := http.Get(server.URL)
		b, _ := ioutil.ReadAll(response.Body)
		body := strings.Trim(string(b), "\n")
		if response.StatusCode != tt.code || body != tt.message {
			t.Errorf("expected (%d, %s); got (%d, %s)", tt.code, tt.message, response.StatusCode, body)
		}
	}
}
```



## 文档生成 

```bash
# 查看文档
go doc fmt.Prinln

# 本地 6060 端口启动了一个web服务器，可以访问本地生成的文档
godoc -http :6060
```



## http 性能分析

```go
// 导入包
import _ "net/http/pprof"
// 访问 /debug/pprof

// 使用 go tool pprof 分析性能
// 1. 启动服务
// 2. 执行 go tool pprof http:localhost:8080/debug/pprof/profile 
// 3. 使用并发模拟工具去访问 启动的服务
// 4. 等待第2步结束，在第二步的窗口输入 web
```

