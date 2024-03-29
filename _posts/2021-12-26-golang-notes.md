---
layout: mypost
title: Golang学习笔记
categories: [Golang]
---

### 安装

```
GOROOT=...
GOPATH=...
PATH=%GOROOT%\bin
```

1. 使用 go env 命令查看 go 的环境配置

2. 使用 go get 时会把第三方包装在第一个 GOPATH

```
// Hello World 测试程序
// go run hello.go
package main
import "fmt"

func main() {
    fmt.Println("hello world")
}
```

### 数据类型和变量声明

提供了类型推导，推荐不声明类型，直接使用 var 定义变量

但是一旦确定类型后，不能把其它类型的值赋值给它

### 基础类型

- bool

- int(取决于系统) int8 int16 int32 int64 uint(取决于系统) uint8 uint16 uint32 uint64

- uintptr 无符号整型，由系统决定占用位大小，足够存放指针即可，和 C 库或者系统接口交互

- float32 float64

- complex64 complex128 复数类型

- string

- 字符类型 rune 代表单个 Unicode 字符，等价 int32

- 错误类型 error 本身就是一个预定义好的接口，里面定义了一个 method

### 复合类型

- 指针

- 数组

- 切片

- 字典 map

- 通道 channel

- 结构体 struct

- 接口 interface

### 定义

```
var name1 [type], name2 [type] [= value1, value2]

//用于区别这是全局变量
var (
    name1 [type] [=value1]
    name2 [type] [=value2]
)

//系统自动推断，声明语句写上 var 关键字其实是显得有些多余了
//这种不带声明格式的只能在函数体中出现
name1, name2 := value1, value2
```

### 常量

常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型

```
const name [type] = value
```

iota，特殊常量，可以认为是一个可以被编译器修改的常量

在每一个 const 关键字出现时，被重置为 0，然后再下一个 const 出现之前，每出现一次 iota，其所代表的数字会自动增加 1。

```
const (
        a = iota   //0
        b          //1
        c          //2
        d = "ha"   //独立值，iota += 1
        e          //"ha"   iota += 1
        f = 100    //iota +=1
        g          //100  iota +=1
        h = iota   //7,恢复计数
        i          //8
)
//0 1 2 ha ha 100 100 7 8
```

### 字符串，数组，切片，字典的操作

### 字符串

字符串连接 x+y

字符串字节长度，中文长度为 3 len(s)

取字节符 s[i]

Go 语言支持两种方式遍历字符串

一种是以字节数组的方式遍历

```
var ss = "你好a"
for i := 0; i < len(ss); i++ {
    fmt.Println(i, ss[i]) //uint8
}
//0 228
//1 189
//2 160
//3 229
//4 165
//5 189
//6 97
```

另一种是以 Unicode 字符遍历

以 Unicode 字符方式遍历时，每个字符的类型是 rune,而不是 byte(int8,一个字节)

```
var ss = "你好a"
// 如果不需要i的值可以将i替换为_
for i, ch := range ss {
    fmt.Println(i, ch)//int32 string(ch)输出实际字符
}
//0 20320
//3 22909
//6 97
```

### 数组

数组声明，注意数组一旦声明，其长度和类型都是不可以变的

在 Go 语言中数组是一个值类型，所有的值类型变量在赋值和作为参数传递时都将产生一次复制动作

数组在作为函数的参数定义是，要明确指出大小,不然的话就是切片了

```
func change(arr [4]int) {
	arr[0] = 5
	fmt.Println(arr)
}
```

```
var a = [5]byte{'1','2','3'}
//[49 50 51 0 0]，自动补零

var a [5]byte
a = [5]byte{'1', '2', '3'}
//a = {'1', '2', '3'}是错误写法

//a = [5]byte{'1', '2', '3', '4', '5', '6'} 错误写法

//自动识别长度
var a = [...]byte{'1', '2', '3', '4', '5','6'}
```

### 切片

数组的长度在定义之后无法再次修改；数组是值类型，每次传递都将产生一份副本。显然这种数据结构无法完全满足开发者的真实需求。

使用切片(slice)可以弥补数组的不足，初看起来，数组切片就像一个指向数组的指针，实际上它拥有自己的数据结构，而不仅仅是个指针。数组切片的数据结构可以抽象为以下 3 个变量：

- 一个指向原生数组的指针；

- 数组切片中的元素个数；

- 数组切片已分配的存储空间。

```
//切片可以基于数组和切片创建
var myArray = [10] int32{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
var mySlice = myArray[2:8]
fmt.Println(mySlice)

//也可以直接创建
//创建一个初始元素个数为5的数组切片，元素初始值为0
var mySlice1 = make([]int, 5)
//创建一个初始元素个数为5的数组切片，元素初始值为0，并预留10个元素的存储空间
var mySlice2 = make([]int, 5, 10)
//直接创建并初始化包含3个元素的数组切片
var mySlice3 = []int32{1, 2, 3}
```

可动态增减元素是数组切片比数组更为强大的功能

数组作为参数传递会进行拷贝的，所以要重新接收

```
var mySlice = make([]int, 5, 10)
fmt.Println(len(mySlice)) //5
fmt.Println(cap(mySlice)) //10
fmt.Println(append(mySlice, 1, 2, 3)) //[0 0 0 0 0 1 2 3]
fmt.Println(mySlice) //[0 0 0 0 0]
mySlice = append(mySlice, mySlice...) // ... 打散序列
```

copy 函数，无需导入，两个切片的大小不要求相等

```
slice1 := []int{1, 2, 3, 4, 5}
slice2 := []int{6, 7, 8}
slice3 := make([]int, 5)
copy(slice3, slice1)
fmt.Println(slice3) //[1 2 3 4 5]
copy(slice1, slice2)
fmt.Println(slice1) //[6 7 8 4 5]
copy(slice2, slice3)
fmt.Println(slice2) //[1,2,3]
```

### 字典

类似于 Java 的 Map

```
//声明map类型的变量，key为string，value为int32，只是声明的话并不能直接使用，要make后才能使用
var map1 map[string]int32

//创建，重载方法，加一个int参数用于分配初始空间
var map1 = make(map[string]int32)

map1["1"] = 1
map1["2"] = 2
map1["1"] = 3
fmt.Println(map1["2"])
fmt.Println(map1) //map[1:3 2:2]

//确定是否存在键值对
value,status := map1["3"]
fmt.Println(value,status)// 0 false

Go语言提供了一个内置函数delete()，用于删除容器内的元素

delete(map1, "1234")
```

### 流程控制

Go 语言支持如下的几种流程控制语句：

- 条件语句，对应的关键字为 if、else 和 else if

- 选择语句，对应的关键字为 switch、case 和 select

- 循环语句，对应的关键字为 for 和 range，没有 while

- 跳转语句，对应的关键字为 goto

一些注意点

- if 语句不用加(),但是{}是必须要加的

- switch 语句自带 break,fallthrough 关键字，当条件满足执行完时，不跳出，执行下一条 case，无论条件正确与否

  ```
  case -1, 0, 1:
      fmt.Printf("-1, 0, 1")
  case 1:
      fmt.Println(1)
      fallthrough
  case 2:
      fmt.Println(2)
  ``

  ```

- 循环语句只有 for 没有 while 和 do-while,使用 range 可以方便的遍历数组，切片，字符串，字典

  ```
  for i := 0; i < 10; i++ {
      if i == 5 {
          fmt.Println(i)
          for i, v := range pMap {
              fmt.Println(i, v)
          }
      }
  }
  ```

### 函数

基本格式：func 函数名 (传入参数) (返回参数) {函数体}

支持不定参数：myfunc(args ...int),当作切片来取值

匿名函数：直接将函数运行{}(),或者直接将函数赋值给一个变量

interface{}:类似于 Java 的 Object，可以接受任何类型的参数

回调：已有函数类型`func f1(f Say)`,接口`func f2(f func(string))`,传实现，或者实现匿名函数

注意：..在左边代表不定参数，在右边代表打散序列

注意：如果想在外部调用该包函数，所调用的函数名必须大写

注意：在 Go 里面，函数也是一个值类型，所以可以像普通变量一样被传递或使用

注意：如果开发者只对该函数其中的某几个返回值感兴趣的话，也可以直接用下划线作为占位符来
忽略其他不关心的返回值，`_, _, lastName, _ := getName()`

```
func add(x int, y int) int {
    return x + y
}

func calc(x int, y int) (int, int) {
    return x + y, x * y
}

fmt.Println(calc(2, 3)) // 5,6
sum, multiply := calc(2, 3)
fmt.Println(sum)      //5
fmt.Println(multiply) //6
```

```
func test(par interface{}){
    fmt.Println(reflect.TypeOf(par))
    switch par.(type) {
    case int:
        fmt.Println("这是int")
    case string:
        fmt.Println("这是string")
    }
}
```

### defer

defer 关键字：使用 defer 修饰的函数。它可以再函数 return 后，函数结束前执行

一般用作资源函数调用结束后的资源回收

一个函数中可以存在多个 defer 语句，因此需要注意的是， defer 语句的调用是遵照先进后出的原则，即最后一个 defer 语句将最先被执行。

```
func main() {
    defer fmt.Println("Over")
    fmt.Println("Begin")
}
```

### 闭包

闭包是可以包含自由（未绑定到特定对象）变量的代码块，这些变量不在这个代码块内或者任何全局上下文中定义，而是在定义代码块的环境中定义。要执行的代码块（由于自由变量包含
在代码块中，所以这些自由变量以及它们引用的对象没有被释放）为自由变量提供绑定的计算环
境（作用域）。

```
// 每次调用myPrint时，会有i的值复制，然后打印，此时 i 值复制了3次，分别是1，2，3
// 由于defer是后进先出，所以执行变成3，2，1
//1 2 3 3 2 1
func main() {
    a := []int{1, 2, 3}
    for _, i := range a {
        fmt.Print(i," ")
        defer myPrint(i)
    }
}
func myPrint(i int) {
    fmt.Print(i," ")
}

// 使用闭包
// 在函数内部保存的是i的引用，当开始调用匿名函数时候，i已经增加到3了
//1 2 3 3 3 3
func main() {
    a := []int{1, 2, 3}
    for _, i := range a {
        fmt.Print(i, " ")
        defer func() {
            fmt.Print(i, " ")
        }()
    }
}
```

### 异常处理

Panic

是一个内建函数，可以中断原有的控制流程，进入一个令人恐慌的流程中。当函
数 F 调用 panic，函数 F 的执行被中断，并且 F 中的延迟函数会正常执行，然
后 F 返回到调用它的地方。在调用的地方， F 的行为就像调用了 panic。这一过
程继续向上，直到程序崩溃时的所有 goroutine 返回。
恐慌可以直接调用 panic 产生。也可以由运行时错误产生，例如访问越界的数
组。

Recover

是一个内建的函数，可以让进入令人恐慌的流程中的 goroutine 恢复过来。 recover
仅在延迟函数中有效。
在正常的执行过程中，调用 recover 会返回 nil 并且没有其他任何效果。如果
当前的 goroutine 陷入恐慌，调用 recover 可以捕获到 panic 的输入值，并且恢
复正常的执行。

自定义异常

```go
type MyError struct {
	Time    time.Time
	Message string
}

func (e *MyError) Error() string {
	return e.Time.Format(time.RFC822) + "," + e.Message
}

func q(a int) error {
	if a == 1 {
		return nil
	}
	var aE = eee.MyError{time.Now(),"666"}
	return &aE
}
```

模拟 try-catch

```go
func tryCatch(a int) int {
  var resp = 0
	// try
	func() {
		// catch
		defer func() {
			err := recover()
			if err != nil {
				resp = -1
			}
		}()
		// code
		if a == 0 {
			panic("手动模拟panic")
		} else {
			resp = a * a
		}
	}()
	return resp
}
```

### 结构体 struct

```
func main() {
var a = new(NameAge)
var b NameAge
fmt.Println(a)
fmt.Println(b)

//&{ 0} 指针
//{ 0}，由于 struct 是值类型，参数传递或者赋值时候会发生拷贝，一般使用&{}传递指针
}
```

可以对新定义的类型创建函数以便操作，可以通过两种途径：

1. 创建一个函数接受这个类型的参数。
   func doSomething(n1 _NameAge, n2 int) { /_ \*/ }
   （你可能已经猜到了）这是 函数调用。
2. 创建一个工作在这个类型上的函数（参阅在 2.1 中定义的接收方）：
   func (n1 _NameAge) doSomething(n2 int) { /_ */ }
   这是方法调用，可以类似这样使用：
   var n *NameAge
   n.doSomething(2)

1、golang 的命名需要使用驼峰命名法，且不能出现下划线

2、golang 中根据首字母的大小写来确定可以访问的权限。无论是方法名、常量、变量名还是结构体的名称，如果首字母大写，则可以被其他的包访问；如果首字母小写，则只能在本包中使用

定义结构和结构的方法

```
type S struct { i int }
func (p *S) Get() int { return p.i }
func (p *S) Put(v int) { p.i = v }
```

struct 用来自定义复杂数据结构，可以包含多个字段（属性），可以嵌套

go 中的 struct 类型理解为类，可以定义方法，和函数定义有些许区别

struct 类型是值类型

```
var book = new(Book)
var book1 Books
book1.title = "Java 程序设计"
fmt.Println(book1, book1.author == "") //{Java 程序设计 } true

var book2 = Books{"Go 程序设计", "作者不祥"}
fmt.Println(book2) //{Go 程序设计 作者不祥}

var book3 = Books{author: "作者不祥", title: "一个书名"}
fmt.Println(book3)        //{一个书名 作者不祥}
fmt.Println(book3.author) //作者不祥
fmt.Println(book3.title)  //一个书名

var book4 *Books = new(Books)
book4.author = "张三"
fmt.Println(book4.author) //张三
fmt.Println(book4)        //&{ 张三}
```

构造函数

golang 中的 struct 没有构造函数，可以伪造一个

继承

```
type Father struct {
    name   string
    fValue string
}

type Son struct {
    name string
    Father
    // father Father 起别名
}

var son = Son{"儿子", Father{"父亲", "特有值"}}

fmt.Println(son)        //{儿子 {父亲 特有值}}
fmt.Println(son.name)   //儿子
fmt.Println(son.fValue) //特有值

// 如果通过起别名，son.fValue是会出错的，必须通过son.别名.fValue来访问
```

tag

在 go 中，首字母大小写有特殊的语法含义，小写包外无法引用。由于需要和其它的系统进行数据交互，例如转成 json 格式。这个时候如果用属性名来作为键值可能不一定会符合项目要求。tag 在转换成其它数据格式的时候，会使用其中特定的字段作为键值。

在 Go 语言中，你可以给任意类型（包括内置类型，但不包括指针类型）添加相应的方法，

继承,比如拓展拓展 int 类型

```
type myInt int

func (v myInt) Less(b myInt) bool {
	return v < b
}

func main() {
	var i myInt = 100
	fmt.Println(i.Less(101))
}
```

初始化 struts

```
type Rect struct {
x, y float64
width, height float64
}

rect1 := new(Rect)
rect2 := &Rect{}
rect3 := &Rect{0, 0, 100, 200}
rect4 := &Rect{width: 100, height: 200}
```

Go 语言也提供了继承，但是采用了组合的文法，所以我们将其称为匿名组合

Go 语言的接口并不是其他语言（C++、 Java、 C#等）中所提供的接口概念
在 Go 语言出现之前，接口主要作为不同组件之间的契约存在。对契约的实现是强制的，你
必须声明你的确实现了该接口。为了实现一个接口，你需要从该接口继承

非侵入式接口,在 Go 语言中，一个类只需要实现了接口要求的所有函数，我们就说这个类实现了该接口

### 接口

```
//定义一个接口
type Callback func(x int, y int) int

//和上面的接口参数一样
func multiply(x int, y int) int {
    return x * y
}

func main() {
    //和上面的接口参数一样
    add := func(x int, y int) int {
        return x + y
    }
    calcTest(3, 5, add)
    calcTest(3, 5, multiply)
    calcTest(3, 5, func(x int, y int) int {
        return x + y
    })
}

func calcTest(x int, y int, cal Callback) {
    fmt.Println(cal(x, y))
}
```

```
// type User struct {
// 	Name string `json:"userName"`
// 	Age  int    `json:"userAge"`
// }

// var user User
// user.Name = "nick"
// user.Age = 18

// conJson, _ := json.Marshal(user)
// fmt.Println(string(conJson)) //{"userName":"nick","userAge":0}

// type Book struct {
// 	Title  string
// 	Author string
// }

//注意和函数定义的区别
// func (this *Book) init(Title string,Author string) {
//     this.Title = Title
//     this.Author = Author
// }

// 	var book1 Books
// 	book1.title = "Java 程序设计"
// 	fmt.Println(book1, book1.author == "") //{Java 程序设计 } true

// 	var book2 = Books{"Go 程序设计", "作者不祥"}
// 	fmt.Println(book2) //{Go 程序设计 作者不祥}

// 	var book3 = Books{author: "作者不祥", title: "一个书名"}
// 	fmt.Println(book3)        //{一个书名 作者不祥}
// 	fmt.Println(book3.author) //作者不祥
// 	fmt.Println(book3.title)  //一个书名

// var book4 *Books = new(Books)
// book4.author = "张三"
// fmt.Println(book4.author) //张三
// fmt.Println(book4)        //&{ 张三}
// var city string

// flag.StringVar(&city, "c", "上海", "城市中文名")
// flag.Parse()

// fmt.Println("城市是:", city)
```

### 多线程

```
func ready(w string, sec int) {
	time.Sleep(time.Duration(sec) * time.Second)
	fmt.Println(w, "is ready !")
}

func main() {
	go ready("Tea", 2)
	go ready("Coffee", 1)
    fmt.Println("I'm waiting")
    //main这里要等待下，不然main结束程序也就结束了
	time.Sleep(5 * time.Second)
}
```

### channel

> Share memory by communicating, don’t communicate by sharing memory.
> 通过通信共享内存，而不是通过共享内存而通信

在并发编程中，通信是保证数据正确的重要手段，golang 提供了 channel 的方式来实现通讯，当然他是线程安全的

下面是一个简单的无缓冲的通道。注意这是无缓冲的，在一个 goroutine 内进行，读/写/读写/写读操作都会 deadlock。正确的做法是在两个 goroutine 内进行读写，但注意一定要先启动 goroutine，不然就阻塞住了

```go
c := make(chan int)
log.Println("start")
go func() {
	time.Sleep(time.Second * 5)
	c <- 1
}()
log.Println("end", <-c)
```

下面是有缓冲的通道，在通道为空读，通道为满写的时候都是阻塞的，因此下面写入不阻塞，读取不阻塞

```go
c := make(chan int, 1)
c <- 1
i := <-c
fmt.Println(i)
```

channel 的用法很多，比如使用两个有缓冲的 channel 就能实现控制协程的数量。另外如果不知道总任务数的话可以通过 channel+WaitGroup 的方式实现

```go
// 总任务数
total := make(chan byte, len(arr))
// 并发执行数
limit := make(chan byte, 20)

defer close(total)
defer close(limit)

for _, v := range arr {
  limit <- 1

  go func(v int) {
    doSomething(v)
    <-limit
    total <- 1
  }(v)
}

for i := 0; i < len(arr); i++ {
  <-total
}
```

### 并发锁

在并发编程中，通锁来保证数据读取和写入的正确性，通过 channel 的阻塞，我们可以很容易的实现锁，同时 golang 也通过 channel 封装好了各种锁模型的实现，这些锁位于`sync`包中，常见的方法如下

```go
sync.Mutex
sync.RWMutex
sync.Once
sync.WaitGroup
```

### 阻塞主线程的几种方式

有时候需要在主线程启动多个服务，同时主线程运行到最后要阻塞住，不然启动的服务就都停掉了。注意启动的服务也要至少有一个是一致运行的，不然会被检测到 deadlock

1. 死循环,但会 100%占用 cpu,不建议使用

```
go func() {...}()
for {}
```

2. channel 阻塞

利用 channel 的阻塞，普通 channel，或者带缓冲的都能形成阻塞

```
c := make(chan int)
go func() {...}()
c <- 1
```

利用 nil channel 阻塞。当不为 channel 分配内存时，channel 就是 nil channel。nil channel 会永远阻塞对该 channel 的读、写操作。

```
var c chan struct{}
go func() {...}()
<-c
```

4.  使用锁

sync.Mutex 同步锁。一个已经锁了的锁，再锁一次会一直阻塞

```
var m sync.Mutex
m.Lock()
go func() {...}()
m.Lock()
```

一直不 Done

```
var wg sync.WaitGroup
wg.Add(1)
go func() {...}()
wg.Wait()
```

5. 空 select

```
go func() {...}()
select{}
```

6. os.Signal。系统信号量，在 go 里面也是个 channel，在收到特定的消息之前一直阻塞

```
sig := make(chan os.Signal, 2)
signal.Notify(sig, syscall.SIGTERM, syscall.SIGINT)
go func() {...}()
<-sig
```

### 异常处理

所以当 golang 中遇到 panic 时，如果不进行 recover，便会导致整个程序挂掉

利用 defer 延迟处理的 recover 进行恢复，具体例子如下

注意在 panic 之前定义 defer

```
func main() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("123456")
		}
	}()

	panic("123")

	fmt.Println("Hello World !")
}
```

### 参考

[GO 语言中文网](https://studygolang.com/)

[用 Golang 来编写 cli 程序吧，Happy~](https://blog.biezhi.me/2017/11/build-cli-with-golang.html)

[Golang 标准库中文文档](https://studygolang.com/static/pkgdoc/main.html)

[astaxie/build-web-application-with-golang](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/preface.md)
