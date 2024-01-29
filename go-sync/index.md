# go sync


# 为什么要同步

操作系统中进程拥有自己的虚拟内存空间，彼此访问不了对方的内存区域，这是操作系统层面上的内存安全。但是由于线程是操作系统的调度单位，多个线程共享进程的内存空间，并发的访问(读写)同一块内存会出现一些诡异的问题。同步访问内存是多线程编程不可逾越的操作。对`golang`稍微熟悉一点就会知道常用的`map`并发并不安全，如果出现多个`goroutine`同时读写，程序会发生崩溃。

# 互斥锁

锁的基本思想是：同一个时刻只有一个线程处于一块代码段，这块代码段称为临界区，刚开始的时候`thread0`进入临界区前会检查一个变量`state`是否为`0`，如果是`0`代表可以进入临界区，并且将`state`置为1，处理完逻辑后，离开临界区把`state`置为0。当然对于`state`操作是原子性的，否则这是个无穷无尽的并发问题。具体思想如下:(far fa-hand-point-down fa-fw):

```go
package main

import (
	"fmt"
	"sync/atomic"
	"time"
)

var (
	state        int32
	CriticalZone = map[int]int{}
	WriteCount   int
	ReadCount    int
)

const (
	LockState   = 1
	UnLockState = 0
)

func Lock() bool {
	return atomic.CompareAndSwapInt32(&state, UnLockState, LockState)
}
func UnLock() bool {
	return atomic.CompareAndSwapInt32(&state, LockState, UnLockState)
}
func write() {
	for {
		if Lock() {
			WriteCount++
			CriticalZone[WriteCount] = WriteCount
			UnLock()
		}
	}

}
func read() {
	for {
		if Lock() {
			ReadCount++
			s := CriticalZone[ReadCount]
			fmt.Println(s)
			time.Sleep(time.Millisecond*200)
			UnLock()
		}

	}

}
func main() {
	CriticalZone = make(map[int]int)
	go write()
	go read()
	select {}
}


```

实现了非常简单的互斥锁，如果把`Lock` 和`UnLock`的部分去掉，则会`panic`

`fatal error: concurrent map read and map write`

```go

func write() {
	for {
		WriteCount++
		CriticalZone[WriteCount] = WriteCount
	}
}

func read() {
	for {
		ReadCount++
		s := CriticalZone[ReadCount]
		fmt.Println(s)
		time.Sleep(time.Millisecond * 200)
	}
}
```

# 基本原语

`golang`在`sync`包中提供了一些基本的同步原语 `sync.Mutex`,`sync.RWMutex`,`sync.WaitGroup`,`sync.Once`,`sync.Cond`

## Mutex

`sync.Mutex`的数据结构长这个样子，其中`state`表示互斥锁的状态，`sema`表示控制锁状态的信号量

```go
//usr/local/go/src/sync/mutex.go
type Mutex struct {
	state int32
	sema  uint32
}
```

`state`的枚举

```go
///usr/local/go/src/sync/mutex.go
const (
	mutexLocked = 1 << iota // mutex is locked
	mutexWoken
	mutexStarving
	mutexWaiterShift = iota
  .....
```

这些状态用二进制的位来表示

{{% figure class="center" src="/img/golang/7.png" alt="img" %}}

其中`mutexWaiterShift`的值是`3`，表示除了最低三位以外其他的位用来记录排队等待的`goroutine`个数

官方的注释中提到`mutex`有正常模式和饥饿模式(normal and starvation)

- 在正常模式中，阻塞的`goroutine`满足排队的先后顺序，即`FIFO order`，但是该模式有个问题：刚刚唤醒的`goroutine`和那些已经在`CPU`执行的`goroutine`竞争互斥锁并没有优势-----实际上也确实如此，那些拥有`CPU`的`goroutine`可能会有很多，因此唤醒的`goroutine`可能会被这个机制"饿死"，而饥饿模式正是为了解决这个问题的。如果到达的`goroutine`尝试获取互斥锁超过了`1ms`,互斥锁则会切换成饥饿模式。
- 在饥饿模式中，等待队列的第一个`goroutine`会直接获得`CPU`的执行权，那些刚刚到达的`goroutine`并不会和唤醒那个`goroutine`竞争互斥锁，也不会进入自旋状态，相反他们会直接追加到等待队列的尾部。
- 如果一个 goroutine 获得了互斥锁并且它在队列的末尾或者它等待的时间少于 1ms，那么当前的互斥锁就会切换回正常模式
- 相比于饥饿模式，正常模式能有更好的性能，但是饥饿模式能很好的解决等待队列尾部高延迟的情况



### Lock 

`sync.Mutex`对外暴露的函数非常简单 --`Lock`  `Unlock`，它实现了Locker 的接口。`Lock`是尝试持有锁的操作而`Unlock`释放锁的操作

```go
// usr/local/go/src/sync/mutex.go
// A Locker represents an object that can be locked and unlocked.
type Locker interface {
	Lock()
	Unlock()
}
```



在`golang`源码中，`Lock`长这个样子

```go
///usr/local/go/src/sync/mutex.go
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	// Slow path (outlined so that the fast path can be inlined)
	m.lockSlow()
}
```

`atomic.CompareAndSwapInt32`这是一个对`int32`类型变量的原子操作，函数名很直观--比较并交换，如果这一步是`false`(表示已经有其他的`goroutine`持有锁)，则进入`lockSlow`

```go
// usr/local/go/src/sync/mutex.go
func (m *Mutex) lockSlow() {
	var waitStartTime int64
	starving := false
	awoke := false
	iter := 0
	old := m.state
	for {
		// Don't spin in starvation mode, ownership is handed off to waiters
		// so we won't be able to acquire the mutex anyway.
		if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
			// Active spinning makes sense.
			// Try to set mutexWoken flag to inform Unlock
			// to not wake other blocked goroutines.
			if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
				awoke = true
			}
			runtime_doSpin()
			iter++
			old = m.state
			continue
		}
```

官方的注释中提到当互斥锁处于饥饿模式的时候，进来的`goroutine`不会自旋，而是将所有权交给刚刚唤醒的`goroutine`

{{< admonition tip >}}
自旋(spin)是一种多线程同步机制，其思想非常简单：循环的去尝试检查某个条件是否为真。自旋过程会一直占用`CPU`，这样的一个优势是在多核的`CPU`上，自旋可以避免`thread`/`routine`的切换，减少上下文的切换的成本。但是自旋本身是个耗`CPU`的操作，操作不当可能会损耗系统的性能。
{{< /admonition >}}

接下来是计算`state`新的状态

```go
		//usr/local/go/src/sync/mutex.go
		new := old
		// Don't try to acquire starving mutex, new arriving goroutines must queue.
		if old&mutexStarving == 0 {
			new |= mutexLocked
		}
		if old&(mutexLocked|mutexStarving) != 0 {
			new += 1 << mutexWaiterShift
		}
		// The current goroutine switches mutex to starvation mode.
		// But if the mutex is currently unlocked, don't do the switch.
		// Unlock expects that starving mutex has waiters, which will not
		// be true in this case.
		if starving && old&mutexLocked != 0 {
			new |= mutexStarving
		}
		if awoke {
			// The goroutine has been woken from sleep,
			// so we need to reset the flag in either case.
			if new&mutexWoken == 0 {
				throw("sync: inconsistent mutex state")
			}
			new &^= mutexWoken
		}
```





### Unlock

```go
// usr/local/go/src/sync/mutex.go
// Unlock unlocks m.
// It is a run-time error if m is not locked on entry to Unlock.
//
// A locked Mutex is not associated with a particular goroutine.
// It is allowed for one goroutine to lock a Mutex and then
// arrange for another goroutine to unlock it.
func (m *Mutex) Unlock() {
	if race.Enabled {
		_ = m.state
		race.Release(unsafe.Pointer(m))
	}

	// Fast path: drop lock bit.
	new := atomic.AddInt32(&m.state, -mutexLocked)
	if new != 0 {
		// Outlined slow path to allow inlining the fast path.
		// To hide unlockSlow during tracing we skip one extra frame when tracing GoUnblock.
		m.unlockSlow(new)
	}
}
```

解锁的过程要分为两种情况进行处理

- 如果`atomic.AddInt32`返回的值是`0`代表解锁成功
- 如果不等于`0`则进入慢解锁的过程:`unlockSlow`

```go
// usr/local/go/src/sync/mutex.go
func (m *Mutex) unlockSlow(new int32) {
	if (new+mutexLocked)&mutexLocked == 0 {
		throw("sync: unlock of unlocked mutex")
	}
	if new&mutexStarving == 0 {
		old := new
		for {
			if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
				return
			}
			new = (old - 1<<mutexWaiterShift) | mutexWoken
			if atomic.CompareAndSwapInt32(&m.state, old, new) {
				runtime_Semrelease(&m.sema, false, 1)
				return
			}
			old = m.state
		}
	} else {
		runtime_Semrelease(&m.sema, true, 1)
	}
}
```

- 在正常模式下
  - 如果不存在等待的`gorutine`队列或者互斥锁的`mutexLocked`，`mutexWoken`,`mutexStarving`都不为`0`直接返回
  - 如果存在等待的`gorutine`队列，会通过`runtime_Semrelease`唤醒等待的`gorutine`,并让它持有锁

- 饥饿模式下，会直接通过`runtime_Semrelease`唤醒等待的`gorutine`,并让它持有锁。

  

## RWMutex

读写互斥锁是细粒度的锁，它不限制并发读的情况，但是不能并发的读写，写写

|      |              读               |              写               |
| :--: | :---------------------------: | :---------------------------: |
|  读  |      :white_check_mark:       | :negative_squared_cross_mark: |
|  写  | :negative_squared_cross_mark: | :negative_squared_cross_mark: |

结构体

```go
type RWMutex struct {
	w           Mutex  // held if there are pending writers
	writerSem   uint32 // semaphore for writers to wait for completing readers
	readerSem   uint32 // semaphore for readers to wait for completing writers
	readerCount int32  // number of pending readers
	readerWait  int32  // number of departing readers
}
```

- w 持有互斥锁的功能
- `writerSem` 写的信号量
- `readerSem` 读的信号量
- `readerCount` 读操作个数
- `readerWait` 表示写操作被阻塞时等待的读操作个数

对外暴露的函数有`sync.RWMutex.Lock` `sync.RWMutex.Unlock` `sync.RWMutex.Rlock` `sync.RWMutex.Runlock`

- 写操作对应的操作是`sync.RWMutex.Lock` 和 `sync.RWMutex.Unlock`
- 读操作对应的操作是 `sync.RWMutex.Runlock` 和 `sync.RWMutex.Rlock`

### Lock 

```go
// /usr/local/go/src/sync/rwmutex.go
func (rw *RWMutex) Lock() {
	// First, resolve competition with other writers.
	rw.w.Lock()
	// Announce to readers there is a pending writer.
	r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
	// Wait for active readers.
	if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
		runtime_SemacquireMutex(&rw.writerSem, false, 0)
	}
}
```

这个过程有以下操作

- 复用互斥锁的功能尝试的去获取锁，如果已经有其他`goroutine`已经持有锁，则自己进入自旋或者休眠
- 当获取锁后，还会判断是否有读的情况，`atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders)+ rwmutexMaxReaders` 如果这个计算不为`0`,表示有其他的`goroutine`持有读锁，则调用`runtime_SemacquireMutex`进入休眠状态等待所有读锁所有者执行结束后释放 `writerSem` 信号量将当前`goroutine`唤醒

### Unlock



## WaitGroup



## Once

`sync.Once` 的语义是保证某个函数只执行一次，这在一些初始化全局变量中非常有用。

`once`的结构体非常简单

```go
// /usr/local/go/src/sync/once.go
type Once struct {
	done uint32
	m    Mutex
}
```

- `done` 是否已调用某函数的标志位
- `m`互斥锁

### do

如果已经调用过，即`atomic.LoadUint32(&o.done) !=0`，直接返回，否则进入`doSlow`

```go
// /usr/local/go/src/sync/once.go
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 0 {
		o.doSlow(f)
	}	
}
```

### doSlow

```go
// /usr/local/go/src/sync/once.go
func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

可以看出如果传入的是不同的`func`,只会执行第一个`func`



## Cond







# 扩展原语






