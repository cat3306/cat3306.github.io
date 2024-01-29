# go defer


# defer

<!--more-->

`defer` 一词是延期，推迟的意思。在`go`中经常被用于关闭文件描述符、关闭数据库连接以及解锁资源。一般操作是紧挨着申请资源的下一句声明例如:

````go
func OpenFile() error {
	file, err := os.OpenFile("/var/log/go_log/test.log", os.O_APPEND|os.O_CREATE|os.O_RDWR, 0777)
	if err != nil {
		return err
	}
	defer file.Close()
	....
}
````

可以看出`defer`用于一些扫尾工作，在函数`OpenFile() `退出前，会执行`file.Close()`，从代码简洁的角度，`defer`比手动释放资源更加优雅

````go
func OpenFile() error {
	file, err := os.OpenFile("/var/log/go_log/test.log", os.O_APPEND|os.O_CREATE|os.O_RDWR, 0777)
	if err != nil {
		return err
	}
	if err := do1(); err != nil {
		file.Close()
		return err
	}
	if err := do2(); err != nil {
		file.Close()
		return err
	}
	....
}
````

# 现象

- 函数中多次调用`defer`，函数返回之前，`defer`的调用顺序是倒序调用，满足先入后出的栈原理

  ```go
  package main
  
  import "fmt"
  
  func main() {
  	LoopDefer()
  }
  func LoopDefer() {
  	for i := 0; i < 5; i++ {
  			defer fmt.Println(i)
  	}
  }
  
  /**
  ▶ go run main.go
  4
  3
  2
  1
  0
  */
  ```

- `defer` 传入的是一个带有参数的函数，会预先计算参数。

  ```go
  package main
  
  import "fmt"
  
  func main() {
  	var i int
  
  	defer fmt.Println(i + Add(Add(1, 1), Add(2, 2)))
  	i++
  }
  func Add(i, j int) int {
  	return i + j
  }
  /**
  ▶ go run main.go
  6
  */
  ```

  上面的例子足以说明一切，在真正的调用`fmt.Println(...)`之前，已经把`i + Add(Add(1, 1), Add(2, 2))`算好了(6)，如果用匿名函数包裹一下，情况就不一样。

  ```go
  package main
  
  import "fmt"
  
  func main() {
  	var i int
  
  	defer func() {
  		fmt.Println(i + Add(Add(1, 1), Add(2, 2)))
  	}()
  	i++
  }
  func Add(i, j int) int {
  	return i + j
  }
  /**
  ▶ go run main.go
  7
  */
  ```

  匿名函数包裹后，`i + Add(Add(1, 1), Add(2, 2))`，是延迟调用的，在`i++`后执行

## 数据结构		

以上的一些`defer`特性，和它的数据结构和实现原理有关，[_defer](https://github.com/golang/go/blob/41d8e61a6b9d8f9db912626eb2bbc535e929fefc/src/runtime/runtime2.go#L907)

```go
type _defer struct {
	siz     int32 //参数和结果的内存大小
	started bool
	heap    bool
	// openDefer indicates that this _defer is for a frame with open-coded
	// defers. We have only one defer record for the entire frame (which may
	// currently have 0, 1, or more defers active).
	openDefer bool 		 // 表示当前 defer 是否经过开放编码的优化
	sp        uintptr  // 栈指针
	pc        uintptr  // 调用方的程序计数器
	fn        *funcval // 传入的函数
	_panic    *_panic  // 触发延迟调用的结构体，可能为空
	link      *_defer  // 延迟调用链表
  /*
  _defer--->_defer--->_defer--->_defer
 
  */

	// If openDefer is true, the fields below record values about the stack
	// frame and associated function that has the open-coded defer(s). sp
	// above will be the sp for the frame, and pc will be address of the
	// deferreturn call in the function.
	fd   unsafe.Pointer // funcdata for the function associated with the frame
	varp uintptr        // value of varp for the stack frame
	// framepc is the current pc associated with the stack frame. Together,
	// with sp above (which is the sp associated with the stack frame),
	// framepc/sp can be used as pc/sp pair to continue a stack trace via
	// gentraceback().
	framepc uintptr
}
```


