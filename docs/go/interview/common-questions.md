# 常见面试题
> 2017年6月份，[诺唯](https://www.zhihu.com/people/noaway)在知乎专栏发表文章[golang 面试题](https://zhuanlan.zhihu.com/p/26972862)，我发现自己部分内容回答不上来，基础知识点不牢固，所以结合Golang在线文档，重新梳理下相关知识点。

重点是查漏补缺，不要陷入奇技淫巧类内容。

## 1 写出下面代码输出内容
```
package main

import (
	"fmt"
)

func main() {
	defer_call()
}

func defer_call() {
	defer func() { fmt.Println("打印前") }()
	defer func() { fmt.Println("打印中") }()
	defer func() { fmt.Println("打印后") }()

	panic("触发异常")
}
```

**解析**:考察知识点`defer`和`panic`。

- defer：在函数返回`return`前执行defer后面的函数。函数体中存在多个defer情况下，按照**LIFO**后进先出的原则执行。参考链接：[Effective Go--defer](https://golang.org/doc/effective_go.html#defer)。
- panic：执行到panic时，函数F的执行立即停止，但是defer正常执行。然后F把panic返回给上层调用者，上层调用函数和F执行同样的步骤，直到所有的goroutine终止。参考链接：[golang builtin pkg](https://golang.org/pkg/builtin/#panic)

## 2 以下代码有什么问题，说明原因
```
type student struct {
	Name string
	Age  int
}

func pase_student() {
	m := make(map[string]*student)
	stus := []student{
		{Name: "zhou", Age: 24},
		{Name: "li", Age: 23},
		{Name: "wang", Age: 22},
	}
	for _, stu := range stus {
		m[stu.Name] = &stu
	}
}
```
**解析**：for...range中stu变量地址已经确定，后面循环体中的stu使用的是同一个变量，`m[stu.Name] = &stu`中取stu的地址，for循环结束stu是{Name: "wang", Age: 22}的地址。参考链接：[聊聊Go中的Range关键字](https://xiaozhou.net/something-about-range-of-go-2016-04-10.html)

## 3 下面的代码会输出什么，并说明原因
```
func main() {
	runtime.GOMAXPROCS(1)
	wg := sync.WaitGroup{}
	wg.Add(20)
	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println("i: ", i)
			wg.Done()
		}()
	}
	for i := 0; i < 10; i++ {
		go func(i int) {
			fmt.Println("i: ", i)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

输出：**随机输出，没有固定顺序**
```
-9-[10][10][10][10][10][10][10][10][10][10]-0--1--2--3--4--5--6--7--8-

-9-[10]-0--1--2--3--4--5--6--7--8-[10][10][10][10][10][10][10][10][10]
```
这里涉及**data race**问题，需要深入了解，[data race](https://golang.org/doc/articles/race_detector.html)

## 4 下面代码会输出什么？
```
type People struct{}

func (p *People) ShowA() {
	fmt.Println("showA")
	p.ShowB()
}
func (p *People) ShowB() {
	fmt.Println("showB")
}

type Teacher struct {
	People
}

func (t *Teacher) ShowB() {
	fmt.Println("teacher showB")
}

func main() {
	t := Teacher{}
	t.ShowA()
}
```
**output**：
```
showA
showB
```
**解析：**Golang中没有继承和子类的概念，但是有组合，详情参见[Effective Go--embedding](https://golang.org/doc/effective_go.html#embedding):
>There's an important way in which embedding differs from subclassing. When we embed a type, the methods of that type become methods of the outer type, but when they are invoked the receiver of the method is the inner type, not the outer one. In our example, when the Read method of a bufio.ReadWriter is invoked, it has exactly the same effect as the forwarding method written out above; the receiver is the reader field of the ReadWriter, not the ReadWriter itself.

## 5 下面代码会触发异常吗？请详细说明
```
func main() {
	runtime.GOMAXPROCS(1)
	int_chan := make(chan int, 1)
	string_chan := make(chan string, 1)
	int_chan <- 1
	string_chan <- "hello"
	select {
	case value := <-int_chan:
		fmt.Println(value)
	case value := <-string_chan:
		panic(value)
	}
}
```
**解析：**随机触发，考查[select](https://tour.golang.org/concurrency/5)的用法。select等待channel，如果channel有信号，就执行对应的语句。如果多个channel就绪，就**随机执行**。

## 6 下面代码输出什么？
```
func calc(index string, a, b int) int {
	ret := a + b
	fmt.Println(index, a, b, ret)
	return ret
}

func main() {
	a := 1
	b := 2
	defer calc("1", a, calc("10", a, b))
	a = 0
	defer calc("2", a, calc("20", a, b))
	b = 1
}
```
结果
```
10 1 2 3
20 0 2 2
2 0 2 2
1 1 3 4
```
**解析：** `defer calc("1", a, calc("10", a, b))`，里面的函数先执行，输出`10 1 2 3`,然后defer带着参数入栈。`defer calc("2", a, calc("20", a, b))`同理。所以执行顺序是：
```
calc("10", a, b) a=1,b=2
calc("20", a, b) a=0,b=2
defer calc("2", a, 2) a=0
defer calc("1", a, 3)
```

## 7 请写出以下输入内容
```
func main() {
	s := make([]int, 5)
	s = append(s, 1, 2, 3)
	fmt.Println(s)
}
```
**解析：**{0 0 0 0 0 1 2 3}，考察make和append的用法。
- [make](https://golang.org/pkg/builtin/#make)：只用于channel，slice，map的初始化。
- [append](https://golang.org/pkg/builtin/#append)：如果slice的cap>len，添加到slice后面，如果cap=len，重新分配slice，调整cap，添加到后面返回新的slice。

## 8 下面的代码有什么问题?
```
type UserAges struct {
	ages map[string]int
	sync.Mutex
}

func (ua *UserAges) Add(name string, age int) {
	ua.Lock()
	defer ua.Unlock()
	ua.ages[name] = age
}

func (ua *UserAges) Get(name string) int {
	if age, ok := ua.ages[name]; ok {
		return age
	}
	return -1
}
```
**解析：**Get操作没有加锁，map是非并发安全的。会报panic错误。

## 9 下面的迭代会有什么问题？
```
func (set *threadSafeSet) Iter() <-chan interface{} {
	ch := make(chan interface{})
	go func() {
		set.RLock()

		for elem := range set.s {
			ch <- elem
		}

		close(ch)
		set.RUnlock()

	}()
	return ch
}
```
**解析：**内部channel阻塞，默认初始化channel时没有缓冲区，需要等待接收者读取后才能继续写入。
>`ch := make(chan interface{})` 和 `ch := make(chan interface{}, 1)`是不一样的。

无缓冲的 不仅仅是只能向 ch 通道放一个值，而是一直要有人接收，那么ch <- elem才会继续下去，要不然就一直阻塞着，也就是说有接收者才去放，没有接收者就阻塞。

而缓冲为1则即使没有接收者也不会阻塞，因为缓冲大小是1只有当 放第二个值的时候 第一个还没被人拿走，这时候才会阻塞 

## 10 以下代码能编译过去吗？为什么？
```
package main

import (
	"fmt"
)

type People interface {
	Speak(string) string
}

type Stduent struct{}

func (stu *Stduent) Speak(think string) (talk string) {
	if think == "bitch" {
		talk = "You are a good boy"
	} else {
		talk = "hi"
	}
	return
}

func main() {
	var peo People = Stduent{}
	think := "bitch"
	fmt.Println(peo.Speak(think))
}
```
**解析：**不能，Stduent没有实现interface的方法，peo的Speak接收者是指针，不是值类型。*Student的指针有提供该方法，但该方法并不属于结构类型Student的方法。因为struct是值类型。

## 11 以下代码打印出来什么内容，说出为什么
```
package main

import (
	"fmt"
)

type People interface {
	Show()
}

type Student struct{}

func (stu *Student) Show() {

}

func live() People {
	var stu *Student
	return stu
}

func main() {
	if live() == nil {
		fmt.Println("AAAAAAA")
	} else {
		fmt.Println("BBBBBBB")
	}
}
```
**解析：**interface底层包含两个域：类型和存储的值（type， data）。`live()`函数返回的接口被包装成People类型，底层存储为(*People, nil)，所以 `live() == nil`为false，只有(nil, nil)时，==nil才为true。 参考：[golang: 详解interface和nil](https://my.oschina.net/goal/blog/194233)

# 参考
- []()