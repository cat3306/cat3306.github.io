# go map


<!--more-->

# 原理

`map`是最常见的数据结构之一，在不同的编程语言中的申明和用法或多或少有点不同，但在底层原理上他们是一样的。数组访问某个元素是通过下标来获取，时间复杂度为O(1)。而`map`可以让`key`通过`hash`函数映射成类似数组下标，从而达到`key`具有索引功能。`map`查找某个`value`，理论上是O(1)的复杂度。一个性能优秀的`map`有两个关键因素-----哈希函数和解决冲突的方法

一个完美的`hash`的函数，必然是将不同`key`映射到不同的索引上。但世界上并不存在这种完美的`hash`函数，因为输入的范围远远大于输出范围。`md5`是一种将二进制（`binary`）映射到32位的字符串中（128bit）,可以表示`2^128`种可能。但是输入范围是无穷大的，必然存在某两种二进制的`md5`一模一样，这种现象就是`hash`碰撞

`good hash`

{{% figure class="center" src="/img/golang/6.png" alt="img" %}}

`bad hash`

{{% figure class="center" src="/img/golang/13.png" alt="img" %}}



# 碰撞解决

`hash`碰撞会产生二义性，解决的办法一般是**开放寻址法**和**拉链法**

## 开放寻址法

其思想非常简单：底层有个数组`array`用于存放键值对，`key`通过某种`hash`函数映射成hash数值并与数组的长度取模（确保索引不可越界），当经过上述操作过后会得到一个`index`，用`index`取出`value`并和目标`value`做对比，如果冲突了会向后寻找下一个空闲的位置

```go
index:=hash("love") % len(array)
```



{{% figure class="center" src="/img/golang/15.png" alt="img" %}}

如图所示：写入数据的时候，当`key3`的和`key1`，`key2`发生冲突，`key3`会被写入`key2`的后面空闲的地方。读数据的时候，根据`hash`函数得到`hash`值再与数组长度取模得到`index`，发现已经被占领了，这时候继续比较后移，直到数组的末尾或者找到目标元素

### 装载因子

所谓的装载因子是指：元素个数与数组的大小的比值，这个比值越大其性能越差，平均比较次数会随着装载因子的增大而线性增加，当达到`100%`时，`map`会退化成线性表，O(n)的复杂度。

## 拉链法

相比于**开放寻址法**，**拉链法**是常见的实现方式。实现方式是底层用数组+链表，也有的用的数组+红黑树比如`java`。

{{% figure class="center" src="/img/golang/14.png" alt="img" %}}

如图所示：
`key6`和`value6`写入过程，先通过`hash`函数得到一个`hash`值然后与数组的长度取模

```go
index := hash("Key6") % len(array)
```

`index`比如为2，选择了2号桶后，就遍历后面的链表

1. 找到相同的键值的键值对-更新对应的值
2. 没有找到相同键值的键值对-在链表后插入新的键值对

`key7`的读取过程，经过`hash`过后命中了1号桶，这时候会遍历链表并进行键的比较，正好查找到`key7`位于链表的第二个位置

# map 数据结构

## hmap

```go
// /usr/local/go/src/runtime/map.go
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/reflectdata/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	extra *mapextra // optional fields
}
```

`mapextra`

```go
type mapextra struct {
	// If both key and elem do not contain pointers and are inline, then we mark bucket
	// type as containing no pointers. This avoids scanning such maps.
	// However, bmap.overflow is a pointer. In order to keep overflow buckets
	// alive, we store pointers to all overflow buckets in hmap.extra.overflow and hmap.extra.oldoverflow.
	// overflow and oldoverflow are only used if key and elem do not contain pointers.
	// overflow contains overflow buckets for hmap.buckets.
	// oldoverflow contains overflow buckets for hmap.oldbuckets.
	// The indirection allows to store a pointer to the slice in hiter.
	overflow    *[]*bmap
	oldoverflow *[]*bmap

	// nextOverflow holds a pointer to a free overflow bucket.
	nextOverflow *bmap
}
```

1. `count` `map`中元素个数
2. `B` 可以推出`buckets`的数量，`len(buckets) == 2^B`
3. `hash0` 随机数种子
4. `oldbuckets` 用于map扩容时存储之前的`buckets`，它的大小是当前`buckets`的一半

{{% figure class="center" src="/img/golang/16.png" alt="img" %}}

如图所示，`bucket`的基本单位是`bmap`，如果单个桶已经装满，这个时候会使用溢出桶，蓝色的是溢出桶，绿色的是正常桶，两个类型的桶在内存空间是连续的

## bmap

`bmap`的结构长这个样子，一个`bmap`可以存储8个键值对，`overflow`指向溢出的下一个`bmap`，`topbits`存储了键的哈希的高8位以此减少访问键值对的次数

```
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr
}
```

{{% figure class="center" src="/img/golang/17.png" alt="img" %}}

# map 读操作

`map`有两种读的方式

```go
v:=hash[key]
v,ok:=hash[key]
```

对应的源码

```go
// /usr/local/go/src/runtime/map.go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	if raceenabled && h != nil {
		callerpc := getcallerpc()
		pc := abi.FuncPCABIInternal(mapaccess1)
		racereadpc(unsafe.Pointer(h), callerpc, pc)
		raceReadObjectPC(t.key, key, callerpc, pc)
	}
	if msanenabled && h != nil {
		msanread(key, t.key.size)
	}
	if asanenabled && h != nil {
		asanread(key, t.key.size)
	}
	if h == nil || h.count == 0 {
		if t.hashMightPanic() {
			t.hasher(key, 0) // see issue 23734
		}
		return unsafe.Pointer(&zeroVal[0])
	}
	if h.flags&hashWriting != 0 {
		throw("concurrent map read and map write")
	}
	hash := t.hasher(key, uintptr(h.hash0))
	m := bucketMask(h.B)
	b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
	if c := h.oldbuckets; c != nil {
		if !h.sameSizeGrow() {
			// There used to be half as many buckets; mask down one more power of two.
			m >>= 1
		}
		oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
		if !evacuated(oldb) {
			b = oldb
		}
	}
	top := tophash(hash)
bucketloop:
	for ; b != nil; b = b.overflow(t) {
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			if t.indirectkey() {
				k = *((*unsafe.Pointer)(k))
			}
			if t.key.equal(key, k) {
				e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
				if t.indirectelem() {
					e = *((*unsafe.Pointer)(e))
				}
				return e
			}
		}
	}
	return unsafe.Pointer(&zeroVal[0])
}
```

- 以`key`，`hash0`为参数，算出`key`的`hash` 

  `hash := t.hasher(key, uintptr(h.hash0))`

- 根据`hash`和`bucketMask`计算落入哪个桶，add是指针位移操作

  ```go
  m := bucketMask(h.B)
  b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
  ```

- 计算`hash`的高8位，用于快速比较

  ```go
  top := tophash(hash)
  ```

- `bucketloop` 是查找键值对的主逻辑

  ```go
  bucketloop:
  	for ; b != nil; b = b.overflow(t) {
  		for i := uintptr(0); i < bucketCnt; i++ {
  			if b.tophash[i] != top {
  				if b.tophash[i] == emptyRest {
  					break bucketloop
  				}
  				continue
  			}
  			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
  			if t.indirectkey() {
  				k = *((*unsafe.Pointer)(k))
  			}
  			if t.key.equal(key, k) {
  				e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
  				if t.indirectelem() {
  					e = *((*unsafe.Pointer)(e))
  				}
  				return e
  			}
  		}
  	}
  ```

  1. 两层循环，第一层遍历桶的溢出桶，第二层遍历桶的8个`tophash`

  2. 通过比较目标`topshash`和桶内的`tophash`快速筛选

  3. 命中某个`tophash`后，通过偏移量取出对应的`key`：

     `k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))`

  4. 比较`key`与目标`key`是否相等，如果相等根据偏移量取出对应的`value`，如果不相等则进入下次循环，直到跳出循环

     ```go
     e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
     ```

  5. 跳出循环，返回对应类型的零值，表示`key`不存在
     ```go
     return unsafe.Pointer(&zeroVal[0])
     ```

对于有`bool`值的读取方式，对应的源码，只是在之前的基础上加上了bool返回值，其他毫无差别。这种有bool返回值的取值方式可以更加精准的判断目标`key`是否是真的存在，如果仅仅判断零值有可能产生二义性：key本身存在，恰好值是零值

```go
// /usr/local/go/src/runtime/map.go
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool) {
	if raceenabled && h != nil {
		callerpc := getcallerpc()
		pc := abi.FuncPCABIInternal(mapaccess2)
		racereadpc(unsafe.Pointer(h), callerpc, pc)
		raceReadObjectPC(t.key, key, callerpc, pc)
	}
	if msanenabled && h != nil {
		msanread(key, t.key.size)
	}
	if asanenabled && h != nil {
		asanread(key, t.key.size)
	}
	if h == nil || h.count == 0 {
		if t.hashMightPanic() {
			t.hasher(key, 0) // see issue 23734
		}
		return unsafe.Pointer(&zeroVal[0]), false
	}
	if h.flags&hashWriting != 0 {
		throw("concurrent map read and map write")
	}
	hash := t.hasher(key, uintptr(h.hash0))
	m := bucketMask(h.B)
	b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
	if c := h.oldbuckets; c != nil {
		if !h.sameSizeGrow() {
			// There used to be half as many buckets; mask down one more power of two.
			m >>= 1
		}
		oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
		if !evacuated(oldb) {
			b = oldb
		}
	}
	top := tophash(hash)
bucketloop:
	for ; b != nil; b = b.overflow(t) {
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			if t.indirectkey() {
				k = *((*unsafe.Pointer)(k))
			}
			if t.key.equal(key, k) {
				e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
				if t.indirectelem() {
					e = *((*unsafe.Pointer)(e))
				}
				return e, true
			}
		}
	}
	return unsafe.Pointer(&zeroVal[0]), false
}
```

# map 写操作

写操作在语法上形如:`myMap["name"]="joker"`，对应的源代码`func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer`（`/usr/local/go/src/runtime/map.go`），这个函数非常长，逐个分析

1. 将传入的`key`计算`hash`值，再计算落入哪个桶中，计算`tophash`

```go
if h.flags&hashWriting != 0 {
		throw("concurrent map writes")
	}
	hash := t.hasher(key, uintptr(h.hash0))

.........

b := (*bmap)(add(h.buckets, bucket*uintptr(t.bucketsize)))
	top := tophash(hash)
```

2. 依旧是两层循环，第一层遍历溢出桶，第二层遍历单个桶的8个`tophash`

````go
bucketloop:
	for {
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if isEmpty(b.tophash[i]) && inserti == nil {
					inserti = &b.tophash[i]
					insertk = add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
					elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
				}
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			if t.indirectkey() {
				k = *((*unsafe.Pointer)(k))
			}
			if !t.key.equal(key, k) {
				continue
			}
			// already have a mapping for key. Update it.
			if t.needkeyupdate() {
				typedmemmove(t.key, k, key)
			}
			elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
			goto done
		}
		ovf := b.overflow(t)
		if ovf == nil {
			break
		}
		b = ovf
	}

````



# map 的扩容







# map 的删除

# map 的优化

## 内存泄露



`go` 的`map`随着长时间的系统运行，其占用的内存只增不减。其原因是因为在map中的`bucket`会随着容量不足进行扩容，但不会因为`delete`操作而缩小容量，而本身的`bucket`需要占用内存的

假设初始化100万元素的`map`，元素全部删除后，看看最后占多少内存。装载因子(`loadfactor`)最大为6.5，则`B`的值为18

计算装载因子

```go
fmt.Println(math.Ceil(math.Log2(float64(1000000) / 6.5)))
```



```go
package main

import (
	"fmt"
	"runtime"
)

const N = 128

func randBytes() [N]byte {
	return [N]byte{}
}

func printAlloc() {
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
	fmt.Printf("%d MB\n", m.Alloc/1024/1024)
}

func main() {
	n := 1_000_000
	m := make(map[int][N]byte, 0)
	printAlloc()

	for i := 0; i < n; i++ {
		m[i] = randBytes()
	}
	printAlloc()

	for i := 0; i < n; i++ {
		delete(m, i)
	}

	runtime.GC()
	printAlloc()
  // 避免m被GC掉
	runtime.KeepAlive(m)
}


```

跑出来的结果

```bash
▶ ./t     
0 MB
461 MB
293 MB
```

可以看见当所有元素被删除后，依然占用`293 MB`的内存，这实际上是`map`本身的内存。

啥元素都没有的`map`

```go
package main

import (
	"fmt"
	"runtime"
)

const N = 128

func printAlloc() {
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
	fmt.Printf("%d MB\n", m.Alloc/1024/1024)
}

func main() {
	n := 1_000_000
	m := make(map[int][N]byte, n)
	printAlloc()

	runtime.KeepAlive(m)
}
```

结果

```bash
▶ go run main.go
293 MB
```

但是为啥在写入100w个元素后的`map`占用的内存为`461 MB`，这是因为在写入过程中扩容了。当`value`的大小 <= 128时，value是直接存入bucket中的。如果初始化`map`的时候，指定容量大小，写入后的内存与初始化的内存一样

```go
package main

import (
	"fmt"
	"runtime"
)

const N = 128

func randBytes() [N]byte {
	return [N]byte{}
}

func printAlloc() {
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
	fmt.Printf("%d MB\n", m.Alloc/1024/1024)
}

func main() {
	n := 1_000_000
  // 指定初始化容量
	m := make(map[int][N]byte, n)
	printAlloc()

	for i := 0; i < n; i++ {
		m[i] = randBytes()
	}
	printAlloc()

	for i := 0; i < n; i++ {
		delete(m, i)
	}

	runtime.GC()
	printAlloc()
  // 避免m被GC掉
	runtime.KeepAlive(m)
}

```

结果

```bash
▶ go run main.go
293 MB
293 MB
293 MB

```

当 `value` 大小超过 128字节 后，bucket 不会直接放 `value`，转而变成一个指针。将 `N` 设为 `129`，结果：

```bash
0 MB
197 MB
38 MB
```

这和把`value`改成指针类型一样

```go
func randBytes() *[N]byte {
	return &[N]byte{}
}

	m := make(map[int]*[N]byte, 0)
	
		for i := 0; i < n; i++ {
		m[i] = randBytes()
	}

```



小结：

1. `map`本身的`bucket`数量只增不减，`bucket`本身会占有部分内存，是"内存泄露"的根本原因
2. 当`value`小于128字节，尽量用指针类型进行`map`的存储，尤其是大容量`map`，极其重要
3. 和`slice`一样尽量指定`map`的初始容量，可以避免扩容带来的性能损耗和减少内存占有量









# 










