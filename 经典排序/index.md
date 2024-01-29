# 经典排序


<!--more-->

## 归并排序

归并排序，时间复杂度(nlogn)，空间复杂度o(n),分而治之
4, 5, -1, -6, -9,-9,-10,-100

![](/img/sort/1.png)

![](/img/sort/2.png)

```go
func TestMergeSort(t *testing.T) {
	array := []int{4, 5, -1, -6, -9,-9,-10,-100}
	fmt.Println(mergeSort(array, 0, len(array)-1))

}
func mergeSort(array []int, low, high int) []int {
	mid := (high-low)/2 + low
	newArray := make([]int, 0)
	if low >= high {
		newArray = append(newArray, array[low])
		return newArray
	}

	left := mergeSort(array, low, mid)
	right := mergeSort(array, mid+1, high)
	var leftIndex, rightIndex int
	for leftIndex < len(left) && rightIndex < len(right) {
		if left[leftIndex] <= right[rightIndex] {
			newArray = append(newArray, left[leftIndex])
			leftIndex++
		} else {
			newArray = append(newArray, right[rightIndex])
			rightIndex++
		}
	}
	for leftIndex < len(left) {
		newArray = append(newArray, left[leftIndex])
		leftIndex++
	}
	for rightIndex < len(right) {
		newArray = append(newArray, right[rightIndex])
		rightIndex++
	}
	return newArray
}
```

## 冒泡排序

时间复杂度o(n*n)

```go
func TestBubbleSort(t *testing.T) {
	array := []int{1, 3, 9, -22, -9, 0, -1, -123}
	bubbleSort(array)
	fmt.Println(array)
}
func bubbleSort(array []int) {
	for i := 0; i < len(array); i++ {
		for j := i + 1; j < len(array); j++ {
			if array[i] > array[j] {
				array[i], array[j] = array[j], array[i]
			}
		}
	}
}
```

## 选择排序 ，

时间复杂度o(n*n),空间复杂度o(1)。每次选取最小(最大)的一个，与当前元素进行互换

```go
func TestSelectSort(t *testing.T) {
	array := []int{8, -5, -3, -6, 1, 9, -90}
	selectSort(array)
	fmt.Println(array)
}
func selectSort(array []int) {
	for i := 0; i < len(array); i++ {
		min := i
		for j := i + 1; j < len(array); j++ {
			if array[j] < array[min] {
				min = j
			}
		}
		if i != min {
			array[i],array[min]=array[min],array[i]
		}
	}
}
```

## 插入排序

时间复杂度o(n*n),空间复杂度o(1)。流程类似于排序扑克牌。

```go
func TestInsertSort(t *testing.T) {
	a := []int{-1, 8, -9, -10, 0, 1, 8, 7, -99}
	insertSort(a)
	fmt.Println(a)
}
func insertSort(array []int) {
	for i := 0; i < len(array); i++ {
		for j := i + 1; j > 0 && j < len(array); j-- {
			if array[j] < array[j-1] {
				array[j], array[j-1] = array[j-1], array[j]
			} else {
				break
			}
		}
	}
}
```

## 快速排序

时间复杂度n*logn。基本思想是，随机选一个key。将所有小于key的放在左边，大于等于key的放在右边。

然后对左右两边的子数列进行第一个操作。直到子数组只有一个数字为止

```go
func TestQuickSort(t *testing.T) {
	array := []int{-1, 2, 4, -6, 7, 8}
	quickSort(array, 0, len(array)-1)
	fmt.Println(array)
}
func quickSort(array []int, low, high int) {
	if low >= high {
		return
	}
	i := low
	j := high
	key := array[i]
	for i < j {
		for i < j && array[j] >= key {
			j--
		}
		if i < j {
			array[i] = array[j]
			i++
		}
		for i < j && array[i] < key {
			i++
		}
		if i < j {
			array[j] = array[i]
			j--
		}
	}
	array[i] = key
	quickSort(array, low, i-1)
	quickSort(array, i+1, high)
}

⋊> ~/g/s/j/go_learn go test --count=1 -v -run TestQuickSort sort/sort_test.go                                                                                                     10:36:54
=== RUN   TestQuickSort
[-6 -1 2 4 7 8]
--- PASS: TestQuickSort (0.00s)
PASS
ok      command-line-arguments  0.006s
```

## 堆排序

利用堆的数据结构进行排序。分为大顶堆，小顶堆，如图
![](/img/sort/3.png)

​									大顶堆
![](/img/sort/4.png)

​									小顶堆	

堆排序分为两部分

1. 构建一个堆
2. 移除顶部元素后重新维持一个堆

由于堆是一种特殊的二叉树，整体排序所消耗的时间是n*logn。具体过程如下

排序这个数组 [-9, 89, 2, 73, 88, 83, -34, -23, 8]

1. 构建堆
   ![](/img/sort/5.png)

   ![](/img/sort/6.png)
   ![](/img/sort/7.png)

   ![](/img/sort/8.png)

   ![](/img/sort/9.png)

   ![](/img/sort/10.png)

   ![](/img/sort/11.png)

   ![](/img/sort/12.png)

   ![](/img/sort/13.png)

2. 移除堆顶元素，重新维护一个新的堆

   ![](/img/sort/13.png)

   ![](/img/sort/14.png)

   ![](/img/sort/15.png)

   ![](/img/sort/16.png)

   ![](/img/sort/17.png)

   ![](/img/sort/20.png)

   ![](/img/sort/21.png)

   ![](/img/sort/12.png)

   ```java
   //java
   public class BigHeap<E extends Comparable<E>> {
       private final ArrayList<E> list = new ArrayList<>();
       public BigHeap() {
       }
     
       public BigHeap(ArrayList<E> list) {
           for (E e : list) {
               add(e);
           }
       }
     
       public void add(E e) {
           list.add(e);
           int currentIndex = list.size() - 1;
           while (currentIndex > 0) {
               int parentIndex = (currentIndex - 1) / 2;
               if (list.get(currentIndex).compareTo(list.get(parentIndex)) > 0) {
                   E tmp = list.get(currentIndex);
                   list.set(currentIndex, list.get(parentIndex));
                   list.set(parentIndex, tmp);
               } else {
                   break;
               }
               currentIndex = parentIndex;
           }
       }
   
       public E remove() {
           if (list.isEmpty()) return null;
           E e = list.get(0);
           list.set(0, list.get(list.size() - 1));
           list.remove(list.size() - 1);
           int len = list.size();
           int currentIndex = 0;
           while (currentIndex < len) {
               int leftIndex = currentIndex * 2 + 1;
               int rightIndex = currentIndex * 2 + 2;
               if (leftIndex >= len) {
                   break;
               }
               int maxIndex = leftIndex;
               if (rightIndex <= len - 1) {
                   if (list.get(rightIndex).compareTo(list.get(leftIndex)) > 0) {
                       maxIndex = rightIndex;
                   }
               }
               if (list.get(currentIndex).compareTo(list.get(maxIndex)) < 0) {
                   E tmp = list.get(currentIndex);
                   list.set(currentIndex, list.get(maxIndex));
                   list.set(maxIndex, tmp);
                   currentIndex = maxIndex;
               } else {
                   break;
               }
           }
           return e;
       }
   }
   
   public class HeapSort {
       public static <E extends Comparable<E>> void heapSort(ArrayList<E> list) {
           BigHeap<E> bigHeap = new BigHeap<>(list);
           for (int i = 0; i < list.size(); i++) {
               list.set(i, bigHeap.remove());
           }
       }
   
       public static void main(String[] args) {
           ArrayList<Integer> array = new ArrayList<>();
           array.add(-9);
           array.add(89);
           array.add(2);
           array.add(73);
           array.add(88);
           array.add(83);
           array.add(-34);
           array.add(-23);
           array.add(8);
           System.out.println(array);
           heapSort(array);
           System.out.println(array);
       }
   }
   ```

   

