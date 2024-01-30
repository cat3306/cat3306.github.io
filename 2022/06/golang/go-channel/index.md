# go channel


# channel

<!--more-->

`channel`被设计用来`goroutine`之间通讯的。`go`在共享内存的设计理念是：不要通过共享内存的方式进行通信，而是应该通过通信的方式共享内存。

# 数据结构与原理

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

|   字段    |                           含义                           |
| :-------: | :------------------------------------------------------: |
|    buf    |                     指向缓存区的指针                     |
|  qcount   |                   缓存区已有的元素个数                   |
| dataqsize |                      缓存区数组容量                      |
| elemsize  |                         元素大小                         |
| elemtype  |                         元素类型                         |
|   sendx   | ch<-x ,goroutine发送数据到缓存区，填充数据的下标，写操作 |
|   recvx   |      x:=<-ch,goroutine向channel读数据的下标，读操作      |
|   recvq   |                      消费者阻塞队列                      |
|   sendq   |                      生产者阻塞队列                      |
|   lock    |              锁，用于维持缓存区数据的完整性              |
|  closed   |             c.closed != 0 表示channel已关闭              |

**`channel`分为有缓存和无缓存两种**。对于有缓存来说，需要缓存区来存储数据，缓存区是一个环形数组，sendx，recvx下标超过或等于数组长度会被置成0。

````go
// /usr/local/go/src/runtime/chan.go 
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
  		.....
		if c.recvx == c.dataqsiz {
			 c.recvx = 0
		}
  		.....
}
````



## 有缓存channel

{{% figure class="center" src="/img/golang/2.png" alt="img" %}}

当有`goroutine`向`channel`发送数据时

{{% figure class="center" src="/img/golang/3.png" alt="img" %}}

如果这时候再向`channel`发送一个数据，`recvx`=`sendx`表示为缓存区为空或者已满

而判断`channel`的缓存区是否已满，源码是这样判断的

```go
// /usr/local/go/src/runtime/chan.go 
func full(c *hchan) bool {
	// c.dataqsiz is immutable (never written after the channel is created)
	// so it is safe to read at any time during channel operation.
	if c.dataqsiz == 0 {
		// Assumes that a pointer read is relaxed-atomic.
		return c.recvq.first == nil
	}
	// Assumes that a uint read is relaxed-atomic.
	return c.qcount == c.dataqsiz
}
```

至于缓存区满了之后需不需要阻塞是由`block`参数决定的

````go
// /usr/local/go/src/runtime/chan.go 
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
		....
		if !block && c.closed == 0 && full(c) {
				return false
		}
		....
}
````

怎么样才能让`channel`无阻塞呢，需要`select`语句中`default`关键字

```go
func main() {
	ch := make(chan int, 10)
	go func() {
		for i := 0; i < 10; i++ {
			ch <- i
		}
		select {
		case ch <- 1:
		default:
			fmt.Println("no block")
		}
	}()

	time.Sleep(time.Minute)
}
```

所谓的阻塞，是阻塞当前`goroutine`,不管是从`channel`中取数据，还是从`channel`发送数据，都有可能阻塞。所以`channel`的数据结构里面有两个等待队列：`recvq`,`sendq`。用来记录阻塞的`goroutine`。等待队列是`sudog`类型的双向列表，`sudog`记录了哪个`goroutine`在等待，等待哪个`channel`

```go
// /usr/local/go/src/runtime/chan.go 
type waitq struct {
	first *sudog
	last  *sudog
}

type sudog struct {

	g *g

	next *sudog
	prev *sudog
	elem unsafe.Pointer


	acquiretime int64
	releasetime int64
	ticket      uint32

	isSelect bool

	success bool

	parent   *sudog // semaRoot binary tree
	waitlink *sudog // g.waiting list or semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}
```

当满足条件时，要立刻唤醒等待的`goroutine`

| 字段 |      含义       |
| :--: | :-------------: |
|  g   | 等待的goroutine |
| elem |   元素的地址    |

当有`goroutine`读走一个数据的时候：v:=<-ch

{{% figure class="center" src="/img/golang/4.png" alt="img" %}}

## 无缓冲channel 

这些字段都是零值

|   字段    |  值  |
| :-------: | :--: |
|    buf    | nil  |
|  qcount   |  0   |
| dataqsize |  0   |
|   sendx   |  0   |
|   recvx   |  0   |

工作的只有`recvq`,`sendq`，当有`goroutine`发送数据时，会直接把数据指针赋值给`recvq`的第一个元素，如果`recvq`为空，阻塞当前`goroutine`，并加入到`sendq`；当有`goroutine`接收数据时，会直接从`sendq`中取出队首元素的数据

{{% figure class="center" src="/img/golang/5.png" alt="img" %}}

## 无数据channel 

有时候遇到的场景是，`goroutine`之间的通讯不涉及数据，但是需要有触发时机。这个时候，`channel`可以定义无数据的`channel` ：ch:=make(chan struct{},5)，此时`buf`字段是`nil`，其他的字段正常


