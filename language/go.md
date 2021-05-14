# go 基本法

## package

引入包`import "xxx"`，和python类似

```go
//因式分解式地引入包
import (
	"fmt"
    "xxx"
)
```

通过`包名.xxx`使用包

可以给包取别名`import alias “actual name”`

通过可以使用时省略包名`import . "pkg"` 

包的匿名导入https://yar999.gitbook.io/gopl-zh/ch10/ch10-05



## 声明

四种类型的声明语句：var、const、type和func；

- go有个神奇的要求，声明了就一定要使用

- go没有实现引用，只有类似引用的操作
- go不支持隐式类型转换



go可以用这种因式分解式的声明

```go
const (
    AbsoluteZeroC Celsius = -273.15 // 绝对零度
    FreezingC     Celsius = 0       // 结冰点温度
    BoilingC      Celsius = 100     // 沸水温度
)
```



### 变量

```go
var 变量名字 类型 = 表达式
var	len	int	= 1

//其中类型可省略（可推断类型），赋值也可省略（保证赋初值），但是不能同时省略

//一些使用例子
var i, j, k int                 // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
var f, err = os.Open(name) // os.Open returns a file and an error
```



局部变量可以用`:=`声明并赋值

```go
v := 1 //声明一个int 1 变量
```



go中只有var++和var--，不能放前面



### 类型

就是cpp的typedef

```go
type 类型名字 底层类型

type Celsius float64    // 摄氏温度
type Fahrenheit float64 // 华氏温度
```



### 函数

```go
func function_name( [parameter list] ) [return_types] {
   函数体
}

//e.x.
func swap(x, y string) (string, string) {
   return y, x
}
```





## 循环

```go
for init; condition; post {} //	e.x   for i:=0;i<len;i++ {}

for condition {}

for {} //无限循环

//for-each
for key, value := range oldMap {
    newMap[key] = value
}

//不需要index的for-each，即cpp的for-each
for _, value := range oldMap {
    Println(value)
}
```



## 数组

和内置数组差不多

```go
var variable_name [SIZE] variable_type

var arr[4]int // 0 0 0 0
var arr[4]int = [4]int{1,2} //1 2 0 0
arr := [4]int{} // 0 0 0 0
arr := [...]int{0,1,2} // 0 1 2

//奇葩方式
r := [...]int{99: -1} //定义了一个含有100个元素的数组r，最后一个元素被初始化为-1，其它元素都是用0初始化
```



### 比较

可以直接用`==`来比较两个数组，只有当两个数组的所有元素都是相等的时候数组才是相等的。不相等比较运算符!=遵循同样的规则。



## 切片slice

```go
var identifier []type
var slice1 []type = make([]type, len)
var slice1 []type = make([]type, len, cap) 

s :=[]int{0,1}//s := [len]int{} 和数组有二义性不允许

s := make([]int, len) //len可以是变量

var numbers []int //这样的 slice == nil，这也是slice唯一合法的比较操作
```



### 底层实现！！！

切片，一开始看感觉和c++的vector很像，但实际上还是有区别的，slice的底层实现比vector简单得多，slice更像是对数组的一层封装而vector是对内存空间操作

切片内部结构：

```go
struct Slice
{   
    byte*    array;       // actual data
    uintgo    len;        // number of elements
    uintgo    cap;        // allocated number of elements
};
//所以unsafe.Sizeof(切片)是 24
```

当把 slice 作为参数，本身传递的是值，但因为 **byte\*  array**指针指向的还是原底层数组，所以对参数的修改也会影响到外部。

但如果对 slice 本身做 append，而且导致 slice 进行了扩容，实际扩容的是函数内复制的一份切片，对于函数外面的切片没有变化。

> 细品上面这句话，如需要扩容则会开辟新的内存并且copy原来的arr，这时append返回的slice的底层数组和原来的不是同一个
>
> 如果不需要扩容那么还是在原有的底层数组进行操作





### len和cap函数

和vector的size和capacity类似，**len切片<=cap切片<=len底层数组**

```go
fmt.Printf("len=%d cap=%d slice=%v\n",len(x),cap(x),x) //打印一个空切片  len=0 cap=0 slice=[]
```



### 切片操作

可以通过切片操作来**引用**数组/切片

```go
s := arr[s:l] //将 arr 中[s,l)的元素创建为一个新的切片
s := arr[:l] //[0,l)
s := arr[s:]//[s, end)

s := arr[:] //相当于s引用arr
```

<img src="https://gblobscdn.gitbook.com/assets%2F-M7bMYfjF3ccPwZetA8L%2Fsync%2Fa8d2956b4780f5f13544239d328c54913495a6ff.png?alt=media"  />

<font color="red">注意，对于slice，切片操作返回一个原始字节系列的子序列，底层都是共享之前的底层数组，因此切片操作对应常量时间复杂度</font>

所以上图中一旦修改了Q2或者summer，mouths也会被改变



### append和copy

```go
func main() {
   var numbers []int
   printSlice(numbers) //len=0 cap=0 slice=[]

   /* 允许追加空切片 */
   numbers = append(numbers, 0)
   printSlice(numbers) //len=1 cap=1 slice=[0]

   /* 向切片添加一个元素 */
   numbers = append(numbers, 1)
   printSlice(numbers) //len=2 cap=2 slice=[0 1]

   /* 同时添加多个元素 */
   numbers = append(numbers, 2,3,4)
   printSlice(numbers) // len=5 cap=6 slice=[0 1 2 3 4]

   /* 创建切片 numbers1 是之前切片的两倍容量*/
   numbers1 := make([]int, len(numbers), (cap(numbers))*2)

   /* 拷贝 numbers 的内容到 numbers1 */
   copy(numbers1,numbers)
   printSlice(numbers1)  //len=5 cap=12 slice=[0 1 2 3 4]
}

func printSlice(x []int){
   fmt.Printf("len=%d cap=%d slice=%v\n",len(x),cap(x),x)
}
```



**append切片**

```go
s = append(s, t...) //t should be slice
```



### 比较

slice唯一合法的比较操作是和nil比较

如果你需要测试一个slice是否是空的，使用len(s) == 0来判断，而不应该用s == nil来判断，因为一个零值的slice等于nil。



### 扩容

可以通过查看$GOROOT/src/runtime/slice.go源码，其中扩容相关代码如下：

```go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
    newcap = cap
} else {
    if old.len < 1024 {
        newcap = doublecap
    } else {
        // Check 0 < newcap to detect overflow
        // and prevent an infinite loop.
        for 0 < newcap && newcap < cap {
            newcap += newcap / 4
        }
        // Set newcap to the requested cap when
        // the newcap calculation overflowed.
        if newcap <= 0 {
            newcap = cap
        }
    }
}
```

从上面的代码可以看出以下内容：

- 首先判断，如果新申请容量（cap）大于2倍的旧容量（old.cap），最终容量（newcap）就是新申请的容量（cap）。
- 否则判断，如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍，即（newcap=doublecap），
- 否则判断，如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的1/4，即（newcap=old.cap,for {newcap += newcap/4}）直到最终容量（newcap）大于等于新申请的容量(cap)，即（newcap >= cap）
- 如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量（cap）。

需要注意的是，切片扩容还会根据切片中元素的类型不同而做不同的处理，比如`int`和`string`类型的处理方式就不一样。



## Map

```go
/* 声明变量，默认 map 是 nil */
var map_variable map[key_data_type]value_data_type

/* 使用 make 函数 */
map_variable := make(map[key_data_type]value_data_type)

/* 声明并初始化一个空map，这个map不等于nil ！ */
var m map[string]int = map[string]int{}
```

声明并初始化

```go
ages := map[string]int{
    "alice":   31,
    "charlie": 34,
}

//等价形式
ages := make(map[string]int)
ages["alice"] = 31
ages["charlie"] = 34
```



### 插入

通过`[]`操作符插入

如果不初始化 map，那么就会创建一个 nil map。nil map 不能用来存放键值对

```go
ages["carol"] = 21 // panic: assignment to entry in nil map
```



### 删除

如果要删除的k-v不存在，那么什么也不发生

```go
delete(ages, "alice") // remove element ages["alice"]
```



### 查找

```go
value := map[key]	//如果key不存在，那么将得到value对应类型的零值

//但是有时通过零值也无法判断，这时可以通过
val, ok := map[key]
if ok{
    //key 存在
}else{
    //key不存在
}
```

遍历

```go
for key, value := range m {
    fmt.Println("Key:", key, "Value:", value)
}
```



### 比较

```go
var ages map[string]int
fmt.Println(ages == nil)    // "true"
fmt.Println(len(ages) == 0) // "true"
```



## 结构体

```go
type my_struct struct{
    member_name  val_type
    ...
}

//声明一个struct变量
var ms my_struct
//声明并初始化
variable_name := structure_variable_type {value1, value2...valuen}
或
variable_name := structure_variable_type { key1: value1, key2: value2..., keyn: valuen}
```



- 结构体成员类型不能是结构体本身类型，因为一个聚合的值不能包含它自身（该限制同样适应于数组）但是S类型的结构体可以包含`*S`指针类型的成员
- 结构体作为参数传递值

### 结构体指针

go的结构体指针`.`相当于c的`->`

```go
var employeeOfTheMonth *Employee = &dilbert
employeeOfTheMonth.Position += " (proactive team player)" 
```



### 成员属性

**结构体中成员的首字母大小写问题**

-  首字母大写相当于 public。
-  首字母小写相当于 private。

**注意:** 这个 public 和 private 是相对于包（go 文件首行的 package 后面跟的包名）来说的。



当要将结构体对象转换为 JSON 时，对象中的成员首字母必须是大写，才能正常转换为 JSON，不是大写的json中就没有这个字段了



## JSON

懒得记了：https://yar999.gitbook.io/gopl-zh/ch4/ch4-05



## 函数

在go中，函数也是一种值，可以赋值给变量、作为参数或者作为函数返回值



### defer

被defer修饰的函数调用，会被延迟到调用者执行完毕后（无论是return正常还是panic异常结束）才执行；当然如果函数在defer语句执行前就退出了，defer语句及修饰的调用就不会执行

可以借助defer实现类似RAII的机制

```go
//自动关闭文件
package ioutil
func ReadFile(filename string) ([]byte, error) {
    f, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer f.Close() //f.Close()会在return ReadAll(f)执行完后才执行
    return ReadAll(f)
}

//自动释放锁
var mu sync.Mutex
var m = make(map[string]int)
func lookup(key string) int {
    mu.Lock()
    defer mu.Unlock()
    return m[key]
}
```



#### defer与匿名函数

defer一个匿名函数，可以实现修改返回值的功能

```go
func triple(x int) (result int) { //x=4
    defer func() { result += x }()//修改了返回值
    return double(x) //double(4)=8，本应返回8
}
fmt.Println(triple(4)) // "12"
```



### panic异常

运行时错误会引起panic异常，或者直接调用内置的panic函数也会引发panic异常；

panic和错误error不同，会引起程序的奔溃，一般用于严重错误；

在Go的panic机制中，defer函数的调用在释放堆栈信息之前。；

#### 主动调用

```go
panic(fmt.Sprintf("invalid suit %q", s))
panic(err) //此时recover的返回值等于err
```

#### Recover捕获异常

如果在deferred的函数中调用了内置函数recover，并且定义该defer语句的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil。

```go
func Parse(input string) (s *Syntax, err error) {
    defer func() {
        if p := recover(); p != nil {
            err = fmt.Errorf("internal error: %v", p)
        }
    }()
    // ...parser...
}
//Parse函数发生异常时，recover会使函数从panic中恢复，这时可以设置返回值，否则会按返回值类型的默认零值返回
```



黑魔法，不包含return语句但能返回一个非零值的函数

```go
func Nonreturn() (val int) {
    defer func(){
        switch p:=recover(); p{
        case nil:   
            val = 1 //Nonreturn func return 1
        case "error a":
            val = 2 //Nonreturn func return 2
        default:
            //Nonreturn func return 0
        }
    }()
    panic(nil)
}
```



## 方法

理解：go没有面向对象，但是提供了类似成员函数的东西，可以为一个类型定义方法，那么这个类型的变量就可以调用这个方法

```go
package geometry
import "math"

type Point struct{ X, Y float64 }

// traditional function
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// 为Point type定义了一个名为Distance的方法
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// call
p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // "5", function call
fmt.Println(p.Distance(q))  // "5", method call


//注意Point类型中member和method的命名，go不允许它们重名，也就是不能再定义一个名为X的方法
```



### 基于指针对象的方法

当调用一个函数时，会对其每一个参数值进行拷贝，如果一个函数需要更新一个变量，或者函数的其中一个参数实在太大我们希望能够避免进行这种默认的拷贝，这种情况下我们就需要用到指针了。对应到我们这里用来更新接收器的对象的方法，<u>当这个接受者变量本身比较大时，我们就可以用其指针而不是对象来声明方法</u>，如下：

```go
func (p *Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}

// call
r := &Point{1, 2}
r.ScaleBy(2) //调用者需要是一个指针
// 但是go的语法糖允许我们用变量直接调用
p := Point{1, 2}
p.ScaleBy(2) //编译器会隐式用&p去调用方法（要求p是左值）
```

一般会约定如果Point这个类有一个指针作为接收器的方法，那么所有Point的方法都必须有一个指针接收器，即使是那些并不需要这个指针接收器的函数。



**译注：** 作者这里说的比较绕，其实有两点：

1. 不管你的method的receiver是指针类型还是非指针类型，都是可以通过指针/非指针类型进行调用的，编译器会帮你做类型转换。
2. 在声明一个method的receiver该是指针还是非指针类型时，你需要考虑两方面的内部，第一方面是这个对象本身是不是特别大，如果声明为非指针变量时，调用会产生一次拷贝；第二方面是如果你用指针类型作为receiver，那么你一定要注意，这种指针类型指向的始终是一块内存地址，就算你对其进行了拷贝。熟悉C或者C艹的人这里应该很快能明白。



### 通过嵌入结构体来扩展类型

也算是个语法糖吧这个，嵌入类型的member也是直接的member，可以直接access

```go
type Point struct{ X, Y float64 }

type ColoredPoint struct {
    Point
    Color color.RGBA
}

var cp ColoredPoint
cp.X = 1
fmt.Println(cp.Point.X) // "1"
cp.Point.Y = 2
fmt.Println(cp.Y) // "2"
```



### 方法值和方法表达式

方法值，字面意思方法也是一个值一个变量，像c++里的function对象

```go
distanceFromP := p.Distance        // method value
distanceFromP(q) //call method
```



方法表达式

当T是一个类型时，方法表达式可能会写作T.f或者(*T).f，会返回一个函数"值"，这种函数会将其第一个参数用作接收器（简单理解的话就是方法需要对象的this指针），所以可以用通常(即不写选择器)的方式来对其进行调用

```go
distance := Point.Distance   // method expression
fmt.Println(distance(p, q))  // "5"
fmt.Printf("%T\n", distance) // "func(Point, Point) float64"
```



### 封装

**结构体成员和方法的首字母大小写问题**

-  首字母大写相当于 public。
-  首字母小写相当于 private。

**注意:** 这个 public 和 private 是相对于包来说的。

这种基于名字的手段使得在语言中最小的封装单元是package，而不是像其它语言一样的类型。一个struct类型的字段对同一个包的所有代码都有可见性，无论你的代码是写在一个函数还是一个方法里。



## 接口

### 接口类型

接口类型具体描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。

```go
// 接口内嵌
type ReadWriter interface {
    Reader
    Writer
}
//等价于
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```



### 实现接口

```go
// 定义一个接口
type Writer interface {
    Write(p []byte) (n int, err error)
    ....
}

//让一个类型实现这个接口
type ByteCounter int

func (c *ByteCounter) Write(p []byte) (int, error) {
    *c += ByteCounter(len(p)) // convert int to ByteCounter
    return len(p), nil
}

func .....

// 接口类型可以作为函数参数/返回值的类型 （Java：实现了接口的对象可以作为接口类型的参数传入）
func Fprintf(w io.Writer, format string, args ...interface{}) (int, error){
    w.Write()//在某个地方调用接口的方法
}
```



### 空接口类型 interface{}

可以把任何值赋给空接口

```go
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)

// 利用空接口类型来对输入参数类型推断
func sqlQuote(x interface{}) string {
    switch x := x.(type) {
    case nil:
        return "NULL"
    case int, uint:
        return fmt.Sprintf("%d", x) // x has type interface{} here.
    case bool:
        if x {
            return "TRUE"
        }
        return "FALSE"
    case string:
        return sqlQuoteString(x) // (not shown)
    default:
        panic(fmt.Sprintf("unexpected type %T: %v", x, x))
    }
}
```



### 接口值

概念上讲一个接口的值，接口值，由两个部分组成，一个具体的类型和那个类型的值。它们被称为接口的动态类型和动态值。

<img src="https://gblobscdn.gitbook.com/assets%2F-M7bMYfjF3ccPwZetA8L%2Fsync%2Fe3d88d19ce72032e54c24aed2b607d4562de01df.png?alt=media"  />

一个接口值基于它的动态类型被描述为空或非空，所以这是一个空的接口值。你可以通过使用w==nil或者w!=nil来判读接口值是否为空。调用一个空接口值上的任意方法都会产生panic



将一个*os.File类型的值赋给变量w:`w = os.Stdout`

<img src="https://gblobscdn.gitbook.com/assets%2F-M7bMYfjF3ccPwZetA8L%2Fsync%2Fb10c60aab1bf5ef2f1a952fc82f1f4b6a00cdfdf.png?alt=media"  />



一个接口值可以持有任意大的动态值。`var x interface{} = time.Now()`

<img src="https://gblobscdn.gitbook.com/assets%2F-M7bMYfjF3ccPwZetA8L%2Fsync%2F26320cc36883fa7f2684a6c7a63cc9dbee6fca44.png?alt=media"  />

#### 接口值比较

接口值可以使用＝＝和！＝来进行比较。两个接口值相等仅当它们都是nil值或者它们的动态类型相同并且动态值也根据这个动态类型的＝＝操作相等。因为接口值是可比较的，所以它们可以用在map的键或者作为switch语句的操作数。

然而，如果两个接口值的动态类型相同，但是这个动态类型是不可比较的（比如切片），将它们进行比较就会失败并且panic



### 类型断言

https://blog.csdn.net/u012807459/article/details/30050391

- [ ] 要转换成接口类型才能进行断言，是因为要利用接口值的动态类型？





## Go协程

个人记：协程类似线程，但是它是在用户空间创建、调度、销毁的，拥有线程的特点但是又避免了线程调度的开销



### 使用协程

```go
f()    // call f(); wait for it to return
go f() // create a new goroutine that calls f(); don't wait
```



### Channels

一个channels是一个通信机制，它可以让一个goroutine通过它给另一个goroutine发送值信息。每个channel都有一个特殊的类型，也就是channels可发送数据的类型。一个可以发送int类型数据的channel一般写为chan int。



#### 创建

```go
ch := make(chan int) // ch has type 'chan int'

chan T // 声明一个双向通道
chan<- T // 声明一个只能用于发送的通道
<-chan T // 声明一个只能用于接收的通道

ch = make(chan int)    // unbuffered channel
ch = make(chan int, 0) // unbuffered channel
ch = make(chan int, 3) // buffered channel with capacity 3
```



#### 关闭

关闭channel，随后对基于该channel的任何发送操作都将导致panic异常。对一个已经<u>被close过的channel之行接收操作依然可以接受到</u>之前已经成功发送的数据；如果channel中已经没有数据的话讲产生一个零值的数据。

```go
close(ch)
```



#### 发送&接收

```go
ch <- x  // a send statement
x = <-ch // a receive expression in an assignment statement
<-ch     // a receive statement; result is discarded
```



#### 注意

- channels的底层是循环数组，所以对它的复制类似引用，它的零值是`nil`
- 两个相同类型的channel可以使用==运算符比较。如果两个channel引用的是相通的对象，那么比较的结果为真。一个channel也可以和nil进行比较。





### 协程泄漏

下面这段程序，当一个协程通过channel发送一个错误给主协程，主协程直接返回了，这时没有协程从channel中接收数据了，如果有其他的协程往channel里面写数据就会永远地阻塞下去，并且永远都不会退出。这种情况叫做goroutine泄露。

```go
// makeThumbnails4 makes thumbnails for the specified files in parallel.
// It returns an error if any step failed.
func makeThumbnails4(filenames []string) error {
    errors := make(chan error)

    for _, f := range filenames {
        go func(f string) {
            _, err := thumbnail.ImageFile(f)
            errors <- err
        }(f)
    }

    for range filenames {
        if err := <-errors; err != nil {
            return err // NOTE: incorrect: goroutine leak!
        }
    }
    return nil
}
```



### 循环快照问题

单独的变量`f`是被所有的匿名函数值所共享，且会被连续的循环迭代所更新的。当新的goroutine开始执行字面函数时，for循环可能已经更新了f并且开始了另一轮的迭代或者(更有可能的)已经结束了整个循环，所以当这些goroutine开始读取f的值时，它们所看到的值已经是slice的最后一个元素了。所以应该将`f`作为参数传给匿名函数。

```go
for _, f := range filenames {
    go func() {
        thumbnail.ImageFile(f) // NOTE: incorrect!
        // ...
    }()
}
```



### 基于select的多路复用

想象一下我们的程序需要同时从多个channels读取数据，但是当我们阻塞在一个channels时即使另一个有数据了也无能为力，如果同时只会有一个有数据那就会无限阻塞。另外就是网络中在一个线程监听的多个socket的问题。



```go
select {
case <-ch1: //每一个case代表一个通信操作(在某个channel上进行发送或者接收)并且会包含一些语句组成的一个语句块。
    // ...
case x := <-ch2:
    // ...use x...
case ch3 <- y:
    // ...
default:
    // ...
}
```

- select会等待case中有能够执行的case时去执行。**当条件满足时，select才会去通信并执行case之后的语句**；这时候其它通信是不会执行的。
- 一个没有任何case的select语句写作select{}，会永远地等待下去。
- 如果**多个case同时就绪时，select会随机地选择一个执行**，这样来保证每一个channel都有平等的被select的机会。



## sync.Mutex互斥锁

```go
mu sync.Mutex // guards balance

func Balance() int {
    mu.Lock()
    b := balance
    mu.Unlock()
    return b
}
// 简洁化
func Balance() int {
    mu.Lock()
    defer mu.Unlock()
    return balance
}
```

- go里没有重入锁



## sync.RWMutex读写锁

```go
var mu sync.RWMutex
var balance int
// 调用了RLock和RUnlock方法来获取和释放一个读取或者共享锁
func Balance() int {
    mu.RLock() // readers lock
    defer mu.RUnlock()
    return balance
}

// 调用mu.Lock和mu.Unlock方法来获取和释放一个写或互斥锁
```



## sync.Once初始化

实现一个并发安全的单例模式

```go
var loadIconsOnce sync.Once
var icons map[string]image.Image
// Concurrency-safe.
func Icon(name string) image.Image {
    loadIconsOnce.Do(loadIcons)//仅执行初始化函数loadIcons一次
    return icons[name]
}
```



## 竞争条件检测

即使我们小心到不能再小心，但在并发程序中犯错还是太容易了。幸运的是，Go的runtime和工具链为我们装备了一个复杂但好用的动态分析工具，竞争检查器(the race detector)。

只要在go build，go run或者go test命令后面加上-race的flag，就会使编译器创建一个你的应用的“修改”版或者一个附带了能够记录所有运行期对共享变量访问工具的test，并且会记录下每一个读或者写共享变量的goroutine的身份信息。另外，修改版的程序会记录下所有的同步事件，比如go语句，channel操作，以及对(*sync.Mutex).Lock，(*sync.WaitGroup).Wait等等的调用。(完整的同步事件集合是在The Go Memory Model文档中有说明，该文档是和语言文档放在一起的。译注：https://golang.org/ref/mem)

竞争检查器会检查这些事件，会寻找在哪一个goroutine中出现了这样的case，例如其读或者写了一个共享变量，这个共享变量是被另一个goroutine在没有进行干预同步操作便直接写入的。这种情况也就表明了是对一个共享变量的并发访问，即数据竞争。这个工具会打印一份报告，内容包含变量身份，读取和写入的goroutine中活跃的函数的调用栈。这些信息在定位问题时通常很有用。9.7节中会有一个竞争检查器的实战样例。

竞争检查器会报告所有的已经发生的数据竞争。然而，它只能检测到运行时的竞争条件；并不能证明之后不会发生数据竞争。所以为了使结果尽量正确，请保证你的测试并发地覆盖到了你到包。

由于需要额外的记录，因此构建时加了竞争检测的程序跑起来会慢一些，且需要更大的内存，即时是这样，这些代价对于很多生产环境的工作来说还是可以接受的。对于一些偶发的竞争条件来说，让竞争检查器来干活可以节省无数日夜的debugging。(译注：多少服务端C和C艹程序员为此尽折腰)



