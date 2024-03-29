+++
title = "interface类型"
author = ["zakudriver"]
description = "golang interface类型"
date = 2020-03-09
lastmod = 2022-07-22T10:15:39+08:00
tags = ["golang"]
categories = ["code"]
draft = false
+++

## Go 语言与鸭子类型的关系 {#go-语言与鸭子类型的关系}

> If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck.
>
> 如果某个东西长得像鸭子, 像鸭子一样游泳, 像鸭子一样嘎嘎叫, 那它就可以被看成是一只鸭子.
>
> 在 Go 语言中, 如果类型的方法集完全包含接口的方法集，则可认为该类型实现了该接口.

鸭子类型是一种动态语言的风格, 在这种风格中, 一个对象有效的语义, 不是由继承自特定的类或实现特定的接口, 而是由它"当前方法和属性的集合"决定.
Go 作为一种静态语言, 通过接口实现了 鸭子类型, 实际上是 Go 的编译器在其中作了隐匿的转换工作.


## 值接收者和指针接收者的区别 {#值接收者和指针接收者的区别}

```go
package main

import "fmt"

type Person struct {
  age int
}

func (p Person) howOld() int {
  return p.age
}

func (p *Person) growUp() {
  p.age += 1
}

func main() {
  // qcrao 是值类型
  qcrao := Person{age: 18}

  // 值类型 调用接收者也是值类型的方法
  fmt.Println(qcrao.howOld())

  // 值类型 调用接收者是指针类型的方法
  qcrao.growUp()
  fmt.Println(qcrao.howOld())

  // ----------------------

  // stefno 是指针类型
  stefno := &Person{age: 100}

  // 指针类型 调用接收者是值类型的方法
  fmt.Println(stefno.howOld())

  // 指针类型 调用接收者也是指针类型的方法
  stefno.growUp()
  fmt.Println(stefno.howOld())
}
```

| -       | 值接收者                                                | 指针接收者                                                 |
|---------|-----------------------------------------------------|-------------------------------------------------------|
| 值类型调用者 | 方法会使用调用者的一个副本，类似于"传值"                | 使用值的引用来调用方法, 上例中 qcrao.growUp() 实际上是 (&amp;qcrao).growUp() |
| 指针类型调用者 | 指针被解引用为值, 上例中, stefno.howOld() 实际上是 (\*stefno).howOld() | 实际上也是"传值", 方法里的操作会影响到调用者, 类似于指针传参, 拷贝了一份指针 |


### 区别 {#区别}

如果方法的接收者是值类型, 无论调用者是对象还是对象指针, 修改的都是对象的副本, 不影响调用者; 如果方法的接收者是指针类型, 则调用者修改的是指针指向的对象本身.

使用值接收者还是指针接收者, 不是由该方法是否修改了调用者 (也就是接收者) 来决定, 而是应该基于该类型的本质.

-   如果类型具备"原始的本质", 也就是说它的成员都是由 Go 语言里内置的原始类型, 如字符串, 整型值等, 那就定义值接收者类型的方法.
    像内置的引用类型, 如 slice, map, interface, channel, 这些类型比较特殊, 声明他们的时候, 实际上是创建了一个 header, 对于他们也是直接定义值接收者类型的方法.
    这样, 调用函数时, 是直接 copy 了这些类型的 header, 而 header 本身就是为复制设计的.
-   如果类型具备非原始的本质, 不能被安全地复制, 这种类型总是应该被共享, 那就定义指针接收者的方法.
    比如 go 源码里的文件结构体 (struct File) 就不应该被复制, 应该只有一份实体.


## iface 和 eface 的区别 {#iface-和-eface-的区别}

iface 和 eface 都是 Go 中描述接口的底层结构体, 区别在于 iface 描述的接口包含方法, 而 eface 则是不包含任何方法的空接口: interface{}.


### iface {#iface}

```go
import "unsafe"

type iface struct {
  tab  *itab          // 接口类型以及实际类型
  data unsafe.Pointer // 接口具体的值, 一般而言是一个指向堆内存的指针
}

type itab struct {
  inter *interfacetype
  _type *_type
  hash  uint32 // copy of _type.hash. Used for type switches.
  _     [4]byte
  fun   [1]uintptr // variable sized. 存储的是第一个方法的函数指针，如果有更多的方法，在它之后的内存空间里继续存储.
}

type interfacetype struct {
  typ     _type
  pkgpath name
  mhdr    []imethod
}

type _type struct {
  // 类型大小
  size    uintptr
  ptrdata uintptr
  // 类型的 hash 值
  hash uint32
  // 类型的 flag，和反射相关
  tflag tflag
  // 内存对齐相关
  align      uint8
  fieldalign uint8
  // 类型的编号，有bool, slice, struct 等等等等
  kind  uint8
  equal func(unsafe.Pointer, unsafe.Pointer) bool
  // gc 相关
  gcdata    *byte
  str       nameOff
  ptrToThis typeOff
}
```

{{< figure src="/ox-hugo/2020-07-09_11-05-26_56564826-82527600-65e1-11e9-956d-d98a212bc863.png" width="400px" >}}


### eface {#eface}

```go
type eface struct {
    _type *_type
    data  unsafe.Pointer
}
```


### \_type {#type}

\_type 是描述 Go 语言中各种数据类型的结构体

```go
type _type struct {
  // 类型大小
  size    uintptr
  ptrdata uintptr
  // 类型的 hash 值
  hash uint32
  // 类型的 flag，和反射相关
  tflag tflag
  // 内存对齐相关
  align      uint8
  fieldalign uint8
  // 类型的编号，有bool, slice, struct 等等等等
  kind  uint8
  equal func(unsafe.Pointer, unsafe.Pointer) bool
  // gc 相关
  gcdata    *byte
  str       nameOff
  ptrToThis typeOff
}
```

Go 语言各种数据类型都是在 \_type 字段的基础上, 增加一些额外的字段来进行管理的:

```go
type arraytype struct {
  typ   _type
  elem  *_type
  slice *_type
  len   uintptr
}

type chantype struct {
  typ  _type
  elem *_type
  dir  uintptr
}

type slicetype struct {
  typ  _type
  elem *_type
}

type functype struct {
  typ      _type
  inCount  uint16
  outCount uint16
}

type ptrtype struct {
  typ  _type
  elem *_type
}

type structfield struct {
  name       name
  typ        *_type
  offsetAnon uintptr
}
```

这些数据类型的结构体定义, 是反射实现的基础.


## 接口的动态类型和动态值 {#接口的动态类型和动态值}

```go
import "unsafe"

type iface struct {
  tab  *itab          // 接口类型以及实际类型
  data unsafe.Pointer // 接口具体的值, 一般而言是一个指向堆内存的指针
}
```

iface 类型包含两个字段:

-   tab: 是接口表指针，指向类型信息
-   data: 是数据指针，则指向具体的数据


### 接口类型和 nil 作比较 {#接口类型和-nil-作比较}

接口值的零值是指动态类型和动态值都为 nil, 这个接口才能被认为 **接口值 == nil**.

1.  ```go
    package main

    import "fmt"

    func main() {
      var a interface{}
      fmt.Println(c == nil) // true

      var b *string
      fmt.Println(b == nil) // true

      a = b
      fmt.Println(a == nil) // false
    }
    ```
    b 赋值给 a 后, a 的动态类型为 \*string , 动态值为 nil , 所以 a == nil 为 false .
2.  ```go
    package main

    import "fmt"

    type MyError string

    func (i MyError) Error() string {
      return i
    }

    func main() {
      err := HandleError()

      fmt.Println(err == nil) // false
    }

    func HandleError() error {
      var err *MyError = nil
      return err
    }
    ```
    调用 HandleError 返回 error 接口类型, 动态类型为 \*MyError , 动态值为 nil .


#### 打印接口的动态值和类型 {#打印接口的动态值和类型}

```go
package main

import (
    "unsafe"
    "fmt"
)

type iface struct {
    itab, data uintptr
}

func main() {
    var a interface{} = nil

    var b interface{} = (*int)(nil)

    x := 5
    var c interface{} = (*int)(&x)

    ia := *(*iface)(unsafe.Pointer(&a))
    ib := *(*iface)(unsafe.Pointer(&b))
    ic := *(*iface)(unsafe.Pointer(&c))

  fmt.Println(ia) // {0 0}
  fmt.Println(ib) // {17426912 0}
  fmt.Println(ic) // {17426912 842350714568}

  fmt.Println(*(*int)(unsafe.Pointer(ic.data))) // 5
}
```

-   a 的动态类型和动态值的地址均为 0, 也就是 nil;
-   b 的动态类型和 c 的动态类型一致, 都是 \*int;
-   c 的动态值为 5.


### 编译器自动检测类型是否实现接口 {#编译器自动检测类型是否实现接口}

```go
var _ io.Writer = (*myWriter)(nil)
```

编译器会由此检查 \*myWriter 类型是否实现了 io.Writer 接口.

```go
package main

import "io"

type myWriter string

func (w *myWriter) Write(p []byte) (n int, err error) {
  return
}

func main() {
  // 检查 *myWriter 类型是否实现了 io.Writer 接口
  var _ io.Writer = (*myWriter)(nil)

  // 检查 myWriter 类型是否实现了 io.Writer 接口
  var _ io.Writer = myWriter{}
}
```

```shell
src/main.go:15:6: cannot use myWriter literal (type myWriter) as type io.Writer in assignment:
    myWriter does not implement io.Writer (missing Write method)
```

myWriter 没用实现 io.Writer


## 接口类型的赋值 (构造) 和断言 {#接口类型的赋值--构造--和断言}


### 赋值 {#赋值}

针对不同类型有以下函数:

> convT2E16, convT2I16
> convT2E32, convT2I32
> convT2E64, convT2I64
> convT2Estring, convT2Istring
> convT2Eslice, convT2Islice
> convT2Enoptr, convT2Inoptr

```go
func convT2I(tab *itab, elem unsafe.Pointer) (i iface) {
  t := tab._type
  if raceenabled {
    raceReadObjectPC(t, elem, getcallerpc(), funcPC(convT2I))
  }
  if msanenabled {
    msanread(elem, t.size)
  }
  x := mallocgc(t.size, t, true)
  typedmemmove(t, x, elem)
  i.tab = tab
  i.data = x
  return
}
```

把 tab 赋给了 iface 的 tab 字段; data 部分则是在堆上申请了一块内存, 然后将 elem 指向的数据拷贝过去.


### 断言 {#断言}

```go
func assertI2I(inter *interfacetype, i iface) (r iface) {
  tab := i.tab
  if tab == nil {
    // explicit conversions require non-nil interface value.
    panic(&TypeAssertionError{nil, nil, &inter.typ, ""})
  }
  if tab.inter == inter {
    r.tab = tab
    r.data = i.data
    return
  }
  r.tab = getitab(inter, tab._type, false)
  r.data = i.data
  return
}

func assertI2I2(inter *interfacetype, i iface) (r iface, b bool) {
  tab := i.tab
  if tab == nil {
    return
  }
  if tab.inter != inter {
    tab = getitab(inter, tab._type, true)
    if tab == nil {
      return
    }
  }
  r.tab = tab
  r.data = i.data
  b = true
  return
}

func assertE2I(inter *interfacetype, e eface) (r iface) {
  t := e._type
  if t == nil {
    // explicit conversions require non-nil interface value.
    panic(&TypeAssertionError{nil, nil, &inter.typ, ""})
  }
  r.tab = getitab(inter, t, false)
  r.data = e.data
  return
}

func assertE2I2(inter *interfacetype, e eface) (r iface, b bool) {
  t := e._type
  if t == nil {
    return
  }
  tab := getitab(inter, t, true)
  if tab == nil {
    return
  }
  r.tab = tab
  r.data = e.data
  b = true
  return
}
```

判断需断言的变量 (iface) 是否满足接口类型 (interfacetype).

assertI2I 对应 接口断言返回一个参数:

```go
package main

import (
  "errors"
  "fmt"
)

func main(args) {
  var a interface{} = errors.New("error")

  err := a.(error)
  fmt.Println(err.Error())
}
```

assertI2I2 则对应返回两个参数的情况:

```go
package main

import (
  "errors"
  "fmt"
)

func main(args) {
  var a interface{} = errors.New("error")

  if err, ok := a.(error); ok {
    fmt.Println(err.Error())
  }
}
```

都在编译阶段编译器判断.


### 打印接口类型的hash值 {#打印接口类型的hash值}

```go
package main

import (
  "fmt"
  "unsafe"
)

type iface struct {
  tab  *itab
  data unsafe.Pointer
}

type itab struct {
  inter uintptr
  _type uintptr
  hash  uint32
  _     [4]byte
  fun   [1]uintptr
}

func main() {
  p := Person(Student{age: 18})

  iface := (*iface)(unsafe.Pointer(&p))
  fmt.Printf("iface.tab.hash = %#x\n", iface.tab.hash) // iface.tab.hash = 0xd4209fda
}
```


## 类型转换和断言的区别 {#类型转换和断言的区别}


### 类型转换 {#类型转换}

Go 语言中不允许隐式类型转换, 也就是说 = 两边, 不允许出现类型不相同的变量.
类型转换前后的两个类型必须相互兼容.

> &lt;结果类型&gt; := &lt;目标类型&gt; ( &lt;表达式&gt; )

```go
package main

import "fmt"

func main() {
  var i int = 9

  var f float64
  f = float64(i)
  fmt.Printf("%T, %v\n", f, f)

  f = 10.8
  a := int(f)
  fmt.Printf("%T, %v\n", a, a)

  // s := []int(i)
}
```


### 断言 {#断言}

空接口 interface{} 没有定义任何函数, 因此 Go 中所有类型都实现了空接口.
当一个函数的形参是 interface{}, 那么在函数中, 需要对形参进行断言, 从而得到它的真实类型.

> &lt;目标类型的值&gt;，&lt;布尔参数&gt; := &lt;表达式&gt;.( 目标类型 ) // 安全类型断言
> &lt;目标类型的值&gt; := &lt;表达式&gt;.( 目标类型 )　　        //非安全类型断言

```go
package main

import "fmt"

type Student struct {
  Name string
  Age  int
}

func main() {
  var i interface{} = new(Student)
  s := i.(*Student)

  fmt.Println(s)
}
```

switch 形式断言

```go
package main

import "fmt"

type Student struct {
  Name string
  Age  int
}

func main() {
  var i interface{}


  judge(i)
}

func judge(v interface{}) {
  fmt.Printf("%p %v\n", &v, v)

  switch v := v.(type) {
  case nil:
    fmt.Printf("%p %v\n", &v, v)
    fmt.Printf("nil type[%T] %v\n", v, v)

  case Student:
    fmt.Printf("%p %v\n", &v, v)
    fmt.Printf("Student type[%T] %v\n", v, v)

  case *Student:
    fmt.Printf("%p %v\n", &v, v)
    fmt.Printf("*Student type[%T] %v\n", v, v)

  default:
    fmt.Printf("%p %v\n", &v, v)
    fmt.Printf("unknow\n")
  }
}

```


### fmt.Println 函数 {#fmt-dot-println-函数}

fmt.Println 函数的参数是 interface{}. 对于内置类型, 函数内部会用穷举法, 得出它的真实类型, 然后转换为字符串打印.
而对于自定义类型, 首先确定该类型是否实现了 String() 方法. 如果实现了, 则直接打印输出 String() 方法的结果; 否则, 会通过反射来遍历对象的成员进行打印.

因为 Student 结构体没有实现 String() 方法, 所以 fmt.Println 会利用反射挨个打印成员变量:

```go
package main

import "fmt"

type Student struct {
  Name string
  Age  int
}

func main() {
  s := Student{
    Name: "zzz",
    Age:  18,
  }

  fmt.Println(s) // {zzz 18}
}
```

增加一个 String() 方法的实现:

```go
import "fmt"

func (s Student) String() string {
  return fmt.Sprintf("[Name: %s], [Age: %d]", s.Name, s.Age) // [Name: zzz], [Age: 18]
}
```

修改 String() 方法:

```go
import "fmt"

func (s *Student) String() string {
  return fmt.Sprintf("[Name: %s], [Age: %d]", s.Name, s.Age) // {zzz 18}
}
```

打印结果并没用调用 String() ,因为:

> 类型 T 只有接受者是 T 的方法; 而类型 \*T 拥有接受者是 T 和 \*T 的方法. 语法上 T 能直接调 \*T 的方法仅仅是 Go 的语法糖.

要调用 String() 需要:

```go
fmt.Println(&s)
```


## 接口转换的原理 {#接口转换的原理}

类型有 m 个方法, 某接口有 n 个方法, 则很容易知道这种判定的时间复杂度为 O(mn); Go 会对方法集的函数按照函数名的字典序进行排序, 所以实际的时间复杂度为 O(m+n).

```go
package main

import "fmt"

type coder interface {
  code()
  run()
}

type runner interface {
  run()
}

type Gopher struct {
  language string
}

func (g Gopher) code() {
  return
}

func (g Gopher) run() {
  return
}

func main() {
  var c coder = Gopher{}

  var r runner
  r = c
  fmt.Println(c, r)
}
```

Gopher 类型同时满足 coder 接口和 runner 接口.

convI2I 函数将一个 interface 转换成 另一个 interface .

```go
func convI2I(inter *interfacetype, i iface) (r iface) {
  tab := i.tab
  if tab == nil {
    return
  }
  if tab.inter == inter {
    r.tab = tab
    r.data = i.data
    return
  }
  r.tab = getitab(inter, tab._type, false)
  r.data = i.data
  return
}
```

inter 表示要转成的接口类型, i 表示一个实体类型.
如果要转换的接口类型和实体类型的接口类型相同就直接返回; 否则就用调用 getitab 函数去匹配满转方法集的接口.

```go
import "unsafe"

func getitab(inter *interfacetype, typ *_type, canfail bool) *itab {
  if len(inter.mhdr) == 0 {
    throw("internal error - misuse of itab")
  }

  // easy case
  if typ.tflag&tflagUncommon == 0 {
    if canfail {
      return nil
    }
    name := inter.typ.nameOff(inter.mhdr[0].name)
    panic(&TypeAssertionError{nil, typ, &inter.typ, name.name()})
  }

  var m *itab

  // First, look in the existing table to see if we can find the itab we need.
  // This is by far the most common case, so do it without locks.
  // Use atomic to ensure we see any previous writes done by the thread
  // that updates the itabTable field (with atomic.Storep in itabAdd).
  t := (*itabTableType)(atomic.Loadp(unsafe.Pointer(&itabTable)))
  if m = t.find(inter, typ); m != nil {
    goto finish
  }

  // Not found.  Grab the lock and try again.
  lock(&itabLock)
  if m = itabTable.find(inter, typ); m != nil {
    unlock(&itabLock)
    goto finish
  }

  // Entry doesn't exist yet. Make a new entry & add it.
  m = (*itab)(persistentalloc(unsafe.Sizeof(itab{})+uintptr(len(inter.mhdr)-1)*sys.PtrSize, 0, &memstats.other_sys))
  m.inter = inter
  m._type = typ
  // The hash is used in type switches. However, compiler statically generates itab's
  // for all interface/type pairs used in switches (which are added to itabTable
  // in itabsinit). The dynamically-generated itab's never participate in type switches,
  // and thus the hash is irrelevant.
  // Note: m.hash is _not_ the hash used for the runtime itabTable hash table.
  m.hash = 0
  m.init()
  itabAdd(m)
  unlock(&itabLock)
finish:
  if m.fun[0] != 0 {
    return m
  }
  if canfail {
    return nil
  }
  // this can only happen if the conversion
  // was already done once using the , ok form
  // and we have a cached negative result.
  // The cached result doesn't record which
  // interface function was missing, so initialize
  // the itab again to get the missing function name.
  panic(&TypeAssertionError{concrete: typ, asserted: &inter.typ, missingMethod: m.init()})
}
```

getitab 函数会根据 interfacetype 和 \_type 去全局的 itab 哈希表中查找, 如果能找到, 则直接返回; 否则, 会根据给定的 interfacetype 和 \_type 新生成一个 itab, 并插入到 itab 哈希表, 这样下一次就可以直接拿到 itab.

这里查找了两次, 并且第二次上锁了, 这是因为如果第一次没找到, 在第二次仍然没有找到相应的 itab 的情况下, 需要新生成一个, 并且写入哈希表, 因此需要加锁.
这样, 其他协程在查找相同的 itab 并且也没有找到时, 第二次查找时, 会被挂住, 之后, 就会查到第一个协程写入哈希表的 itab.

itabAdd 函数会把 itab 写入到全局itabTable

```go
import "unsafe"

func itabAdd(m *itab) {
  // Bugs can lead to calling this while mallocing is set,
  // typically because this is called while panicing.
  // Crash reliably, rather than only when we need to grow
  // the hash table.
  if getg().m.mallocing != 0 {
    throw("malloc deadlock")
  }

  t := itabTable
  if t.count >= 3*(t.size/4) { // 75% load factor
    // Grow hash table.
    // t2 = new(itabTableType) + some additional entries
    // We lie and tell malloc we want pointer-free memory because
    // all the pointed-to values are not in the heap.
    t2 := (*itabTableType)(mallocgc((2+2*t.size)*sys.PtrSize, nil, true))
    t2.size = t.size * 2

    // Copy over entries.
    // Note: while copying, other threads may look for an itab and
    // fail to find it. That's ok, they will then try to get the itab lock
    // and as a consequence wait until this copying is complete.
    iterate_itabs(t2.add)
    if t2.count != t.count {
      throw("mismatched count during itab table copy")
    }
    // Publish new hash table. Use an atomic write: see comment in getitab.
    atomicstorep(unsafe.Pointer(&itabTable), unsafe.Pointer(t2))
    // Adopt the new table as our own.
    t = itabTable
    // Note: the old table can be GC'ed here.
  }
  t.add(m)
}

func iterate_itabs(fn func(*itab)) {
  // Note: only runs during stop the world or with itabLock held,
  // so no other locks/atomics needed.
  t := itabTable
  for i := uintptr(0); i < t.size; i++ {
    m := *(**itab)(add(unsafe.Pointer(&t.entries), i*sys.PtrSize))
    if m != nil {
      fn(m)
    }
  }
}
```


## 如何用 interface 实现多态 {#如何用-interface-实现多态}

多态是一种运行期的行为, 它有以下几个特点:

1.  一种类型具有多种类型的能力
2.  允许不同的对象对同一消息做出灵活的反应
3.  以一种通用的方式对待个使用的对象
4.  非动态语言必须通过继承和接口的方式来实现

<!--listend-->

```go
package main

import "fmt"

func main() {
  s := Student{age: 18}
  whatJob(&s)

  growUp(&s)
  fmt.Println(s)

  p := Programmer{age: 100}
  whatJob(p)

  growUp(p)
  fmt.Println(p)
}

func whatJob(p Person) {
  p.job()
}

func growUp(p Person) {
  p.growUp()
}

type Person interface {
  job()
  growUp()
}

type Student struct {
  age int
}

func (p Student) job() {
  fmt.Println("I am a student.")
  return
}

func (p *Student) growUp() {
  p.age += 1
  return
}

type Programmer struct {
  age int
}

func (p Programmer) job() {
  fmt.Println("I am a programmer.")
  return
}

func (p Programmer) growUp() {
  p.age += 10
  return
}
```


## Go 接口与 C++ 接口有何异同 {#go-接口与-c-plus-plus-接口有何异同}

接口定义了一种规范, 描述了类的行为和功能, 而不做具体实现.

C++ 的接口是使用抽象类来实现的, 如果类中至少有一个函数被声明为纯虚函数, 则这个类就是抽象类.
纯虚函数是通过在声明中使用 "= 0" 来指定的. 例如:

```c++
class Shape {
public:
  // 纯虚函数
  virtual double getArea() = 0;

private:
  string name; // 名称
};
```

> 设计抽象类的目的, 是为了给其他类提供一个可以继承的适当的基类. 抽象类不能被用于实例化对象, 它只能作为接口使用.
> 派生类需要明确地声明它继承自基类, 并且需要实现基类中所有的纯虚函数.
>
> C++ 定义接口的方式称为“侵入式”, 而 Go 采用的是 “非侵入式”, 不需要显式声明, 只需要实现接口定义的函数, 编译器自动会识别.
>
> C++ 和 Go 在定义接口方式上的不同, 也导致了底层实现上的不同. C++ 通过虚函数表来实现基类调用派生类的函数;
> 而 Go 通过 itab 中的 fun 字段来实现接口变量调用实体类型的函数. C++ 中的虚函数表是在编译期生成的; 而 Go 的 itab 中的 fun 字段是在运行期间动态生成的.
> 原因在于, Go 中实体类型可能会无意中实现 N 多接口, 很多接口并不是本来需要的, 所以不能为类型实现的所有接口都生成一个 itab, 这也是“非侵入式”带来的影响;
> 这在 C++ 中是不存在的, 因为派生需要显示声明它继承自哪个基类.
