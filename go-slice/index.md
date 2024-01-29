# go slice


<!--more-->

# 切片

go的切片是动态数组，所谓的动态是指切片的容量（底层数组长度）随着元素的增加而扩充，由于数组的长度一经确定就不能更改，所以扩充的原理很自然就想到`malloc` 一个新的数组空间，将原有的数组元素拷贝到新数组中。
切片有四种申明方式：

```go
list:=make([]int,0,10)

var list2 []int

list3:=list[0:1]

list4:=[]int{1,2,3}
```

`make` 关键字的第二个参数是切片长度，第三个参数是容量。估计初始化容量可以减少扩容带来的性能损耗。

# 数据结构

在谈数据结构之前，谈谈几个go切片的特性

## slice的一些特性

1. 下面输出是什么？

```go
package main

import "fmt"

func main() {
	list := make([]int, 0)
	add(list)
	fmt.Println("main func:", list)
	add1(&list)
	fmt.Println("main func:", list)
}
func add(list []int) {
	list = append(list, 1, 2, 3)
	fmt.Println("add func:", list)
}

func add1(list *[]int) {
	*list = append(*list, 1, 2, 3)
	fmt.Println("add1 func:", list)
}
```

{{% admonition type="info" title="答案" details="true" %}} 

```
add func: [1 2 3]
main add: []
add1 func: &[1 2 3]
main add1 [1 2 3]
```

{{% /admonition %}}

{{% admonition type="info" title="分析" details="true" %}} 

 因为go 的函数参数是值传递方式，切片底层是个结构体，由于切片的容量不够，发生了扩容，底层的数组地址已经改变了，但是调用者无法感知这种改变

如果传入的是个切片指针，调用者是能感知扩容的变化

{{% /admonition %}}	



2. 下面输出是什么？

```go
func main() {
	list := []int{1, 2, 3}
	list1 := list[:2]
	list1 = append(list1, 9)
	fmt.Println(list, list1)
	list1 = append(list1, 10)
	fmt.Println(list, list1)
}
```

{{% admonition type="info" title="答案" details="true" %}} 

```
[1 2 9] [1 2 9]
[1 2 9] [1 2 9 10]
```

{{% /admonition %}}

{{% admonition type="info" title="分析" details="true" %}} 

再不发生扩容的情况下，list1 截取list部分元素，它们共享同一个数组，当`list1 = append(list1, 9)` 会直接覆盖原来的切片，新切片如果发生扩容` list1 = append(list1, 10)`，则不影响原来的底层数组

{{% /admonition %}}	

3. 下面输出是什么？

```go
func main() {
	list := []int{1, 2, 3}
	list1 := list[:2]
	fmt.Println(len(list), cap(list))
	fmt.Println(len(list1), cap(list1))
	list2 := list[1:]
	fmt.Println(len(list2), cap(list2))
}
```

{{% admonition type="info" title="答案" details="true" %}} 

```
3 3
2 3
2 2
```

{{% /admonition %}}

{{% admonition type="info" title="分析" details="true" %}} 

当新切片是从其他切片截取的，新切片的容量会随着左下标增加而减少，如果`list2 := list[3:]` cap(list3) =0

{{% /admonition %}}	

## slice的数据结构

`/usr/local/go/src/reflect/value.go`

- `Data` 是指向数组的指针
- `Len` 是切片的长度
- `Cap` 是当前切片的容量，即`Data`数组的长度

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

{{% figure class="center" src="/img/golang/8.png" alt="img" %}}

## 图解上面的特性

### 1



{{% figure class="center" src="/img/golang/9.png" alt="img" %}}

### 2



{{% figure class="center" src="/img/golang/10.png" alt="img" %}}

### 3

{{% figure class="center" src="/img/golang/11.png" alt="img" %}}

# 追加与扩容

可以使用`append`关键字向切片中追加元素，在追加元素过程中如果底层数组容量不够会触发扩容机制

对于`list=append(list,1,2,3)`，会有下面代码步骤:

获取旧的长度，容量，如果新的长度大于容量，则进行扩容`growslice`

`*(ptr+len) = 1` 等一系列操作，给新的切片赋值

```go
// slice = append(slice, 1, 2, 3)
a := &slice
ptr, len, cap := slice
newlen := len + 3
if uint(newlen) > uint(cap) {
   newptr, len, newcap = growslice(slice, newlen)
   vardef(a)
   *a.cap = newcap
   *a.ptr = newptr
}
newlen = len + 3
*a.len = newlen
*(ptr+len) = 1
*(ptr+len+1) = 2
*(ptr+len+2) = 3
```

{{% figure class="center" src="/img/golang/12.png" alt="img" %}}

## 扩容逻辑

- 如果期望的容量大于当前容量的两倍，新的容量就是期望的容量
- 如果当前的切片的长度小于1024，新的容量翻倍
- 如果当前的切片的长度大于1024，就会每次增加容量的25%，直到新容量大于期望容量

```go
// /usr/local/go/src/runtime/slice.go
newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.cap < 1024 {
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

## 内存对齐

关于内存对齐是怎么回事，专门有篇文章探讨[内存对齐](../../op-system/memory-alignment/)

上面的步骤只是初步计算所需要的容量，还需要内存对齐计算精确的内存大小。如果切片的元素所占有的字节位1，8(`sys.PtrSize`)，2的幂数，需要内存对齐，过程如下

```go
var overflow bool
	var lenmem, newlenmem, capmem uintptr
	// Specialize for common values of et.size.
	// For 1 we don't need any division/multiplication.
	// For sys.PtrSize, compiler will optimize division/multiplication into a shift by a constant.
	// For powers of 2, use a variable shift.
	switch {
	case et.size == 1:
		lenmem = uintptr(old.len)
		newlenmem = uintptr(cap)
		capmem = roundupsize(uintptr(newcap))
		overflow = uintptr(newcap) > maxAlloc
		newcap = int(capmem)
	case et.size == sys.PtrSize:
		lenmem = uintptr(old.len) * sys.PtrSize
		newlenmem = uintptr(cap) * sys.PtrSize
		capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
		overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
		newcap = int(capmem / sys.PtrSize)
	case isPowerOfTwo(et.size):
		var shift uintptr
		if sys.PtrSize == 8 {
			// Mask shift for better code generation.
			shift = uintptr(sys.Ctz64(uint64(et.size))) & 63
		} else {
			shift = uintptr(sys.Ctz32(uint32(et.size))) & 31
		}
		lenmem = uintptr(old.len) << shift
		newlenmem = uintptr(cap) << shift
		capmem = roundupsize(uintptr(newcap) << shift)
		overflow = uintptr(newcap) > (maxAlloc >> shift)
		newcap = int(capmem >> shift)
	default:
		lenmem = uintptr(old.len) * et.size
		newlenmem = uintptr(cap) * et.size
		capmem, overflow = math.MulUintptr(et.size, uintptr(newcap))
		capmem = roundupsize(capmem)
		newcap = int(capmem / et.size)
	}
```

`roundupsize(uintptr(newcap) * sys.PtrSize)`会将将要申请的内存向上取整得到capmem，新的容量为	`newcap = int(capmem / sys.PtrSize)`

实际怎样，我们用`dlv`调试下

### 调试



```go
func main() {
	var arr []int64
	fmt.Println(cap(arr))
	arr = append(arr, 1, 2, 3, 4, 5)
	fmt.Println(cap(arr))
}
```

1. ````
   cat3306/gosrclearn/slice                                                                                                                                                                 
   ▶ dlv debug
   Type 'help' for list of commands.
   (dlv) 
   
   
   ````

2. ```
   ▶ dlv debug
   Type 'help' for list of commands.
   (dlv) break runtime.growslice
   Breakpoint 1 set at 0x1047fea for runtime.growslice() /usr/local/go/src/runtime/slice.go:162
   (dlv) 
   ```

3. ```
   (dlv) continue
   > runtime.growslice() /usr/local/go/src/runtime/slice.go:162 (hits goroutine(1):5 total:9) (PC: 0x1047fea)
   Warning: debugging optimized function
      157: // NOT to the new requested capacity.
      158: // This is for codegen convenience. The old slice's length is used immediately
      159: // to calculate where to write new values during an append.
      160: // TODO: When the old backend is gone, reconsider this decision.
      161: // The SSA backend might prefer the new length or to return only ptr/cap and save stack space.
   => 162: func growslice(et *_type, old slice, cap int) slice {
      163:         if raceenabled {
      164:                 callerpc := getcallerpc()
      165:                 racereadrangepc(old.array, uintptr(old.len*int(et.size)), callerpc, funcPC(growslice))
      166:         }
      167:         if msanenabled {
   ```

4. ```
   (dlv) print old
   runtime.slice {array: unsafe.Pointer(0x0), len: 0, cap: 0}
   (dlv) print cap
   5
   (dlv) 
   ```

5. ```
   > runtime.growslice() /usr/local/go/src/runtime/slice.go:219 (PC: 0x1048085)
   Warning: debugging optimized function
      214:                 newcap = int(capmem)
      215:         case et.size == sys.PtrSize:
      216:                 lenmem = uintptr(old.len) * sys.PtrSize
      217:                 newlenmem = uintptr(cap) * sys.PtrSize
      218:                 capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
   => 219:                 overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
      220:                 newcap = int(capmem / sys.PtrSize)
      221:         case isPowerOfTwo(et.size):
      222:                 var shift uintptr
      223:                 if sys.PtrSize == 8 {
      224:                         // Mask shift for better code generation.
   (dlv) print capmem
   48
   (dlv) 
   ```

6. ```
   (dlv) 
   > runtime.growslice() /usr/local/go/src/runtime/slice.go:255 (PC: 0x10481eb)
   Warning: debugging optimized function
      250:         //
      251:         // func main() {
      252:         //   s = append(s, d, d, d, d)
      253:         //   print(len(s), "\n")
      254:         // }
   => 255:         if overflow || capmem > maxAlloc {
      256:                 panic(errorString("growslice: cap out of range"))
      257:         }
      258: 
      259:         var p unsafe.Pointer
      260:         if et.ptrdata == 0 {
   (dlv) print newcap
   6
   ```

   由上面细节可知，预期容量是`40(5*8)`，但是由于内存对齐的逻辑，`capmem`是48，所以新的容量长度为`6`

# 拷贝切片

`copy(dst, src []Type)`

```go
// /usr/local/go/src/runtime/slice.go
func slicecopy(toPtr unsafe.Pointer, toLen int, fromPtr unsafe.Pointer, fromLen int, width uintptr) int {
	if fromLen == 0 || toLen == 0 {
		return 0
	}

	n := fromLen
	if toLen < n {
		n = toLen
	}

	if width == 0 {
		return n
	}

	size := uintptr(n) * width
	if raceenabled {
		callerpc := getcallerpc()
		pc := funcPC(slicecopy)
		racereadrangepc(fromPtr, size, callerpc, pc)
		racewriterangepc(toPtr, size, callerpc, pc)
	}
	if msanenabled {
		msanread(fromPtr, size)
		msanwrite(toPtr, size)
	}

	if size == 1 { // common case worth about 2x to do here
		// TODO: is this still worth it with new memmove impl?
		*(*byte)(toPtr) = *(*byte)(fromPtr) // known to be a byte pointer
	} else {
		memmove(toPtr, fromPtr, size)
	}
	return n
}
```

# 延伸阅读

- [Arrays, slices (and strings): The mechanics of ‘append’](https://blog.golang.org/slices)
- [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)
- [Array vs Slice: accessing speed](https://stackoverflow.com/questions/30525184/array-vs-slice-accessing-speed)


