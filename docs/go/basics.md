# 基础

# 逃逸分析
编译代码时，编译器根据分析，判断将变量分配在`栈`或`堆`上。

## 一般情况
函数定义中，一般将局部变量和参数分配到栈上（stack frame）上。但是，如果编译器不能确定在函数返回（return）时，变量是否被引用（reference），分配到堆上；如果局部变量非常大，也应分配在堆上。

如果对变量取地址（\*和&操作），则有可能分配在堆上。此外，还需要进行逃逸分析（escape analytic），判断return后变量是否被引用，不引用分配到栈上，引用分配到堆上。

上面的分析总结自[Go FAQ-Heap and Stack](https://golang.org/doc/faq#stack_or_heap)

# vet
用于检测代码中工程师常犯的错误：
* 错误的printf格式
* 错误的构建tag
* 在闭包中使用错误的range循环变量
* 无用的赋值操作
* 无法到达的代码
* 错误使用mutex
* ...

```golang
go vet pkg
```

# Data race
数据竞争：当多个goroutine并发访问同一个变量，并且至少有一个goroutine对变量进行写操作时，就会发生数据竞争。

go提供了一个内置的数据竞争检测工具：
```golang
$ go test -race mypkg    // to test the package
$ go run -race mysrc.go  // to run the source file
$ go build -race mycmd   // to build the command
$ go install -race mypkg // to install the package
```

# 垃圾回收

# 结构体
如果结构体的全部成员都是可以比较的，那么结构体也是可以比较的，那样的话两个结构体将可以使用==或!=运算符进行比较。

可比较的结构体类型和其他可比较的类型一样，可以用于map的key类型。

## struct{}
如果结构体没有任何成员的话就是空结构体，写作struct{}。它的大小为0，也不包含任何信息，但是有时候依然是有价值的。有些Go语言程序员用map来模拟set数据结构时，用它来代替map中布尔类型的value，只是强调key的重要性，但是因为节约的空间有限，而且语法比较复杂，所以我们通常会避免这样的用法。

##  结构体嵌入和匿名成员
Go语言有一个特性让我们只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员。匿名成员的数据类型必须是命名的类型或指向一个命名的类型的指针。

匿名类型可以被主类型通过`.`直接访问


# 复合数据类型
## map
但是map中的元素并不是一个变量，因此我们不能对map的元素进行取址操作：
```go
_ = &ages["bob"] // compile error: cannot take address of map element
```
禁止对map元素取址的原因是map可能随着元素数量的增长而重新分配更大的内存空间，从而可能导致之前的地址无效。

Map的迭代顺序是不确定的，并且不同的哈希函数实现可能导致不同的遍历顺序。在实践中，遍历的顺序是随机的，每一次遍历的顺序都不相同。这是故意的，每次都使用随机的遍历顺序可以强制要求程序不会依赖具体的哈希函数实现。如果要按顺序遍历key/value对，我们必须显式地对key进行排序。

# defer panic和recovery
如果在deferred函数中调用了内置函数recover，并且定义该defer语句的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil。

虽然把对panic的处理都集中在一个包下，有助于简化对复杂和不可以预料问题的处理，但作为被广泛遵守的规范，你不应该试图去恢复其他包引起的panic

# interface
接口类型是对其它类型行为的抽象和概括；因为接口类型不会和特定的实现细节绑定在一起，通过这种抽象的方式我们可以让我们的函数更加灵活和更具有适应能力。

接口类型具体描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。


# goroutine和channel

# 反射