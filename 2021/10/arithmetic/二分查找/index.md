# 二分查找


<!--more-->


  ```go
a := []int{0, 1, 2, 3, 4, 5, 6}

func binarySearch(array []int, target int) (bool, int) {
	min := 0
	max := len(array) - 1
	for min <= max {
		mid := (max-min)/2 + min
		if target > array[mid] {
			min = mid + 1
		} else if target < array[mid] {
			max = mid - 1
		} else {
			return true,target
		}
	}
	return false,0
}
  ```



  ```java
public class BinarySearch {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        list.add(0);
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);
        list.add(6);
        System.out.println(binarySearch(list, 9));
    }

    public static <E extends Comparable<E>> E binarySearch(ArrayList<E> list, E target) {
        if (list == null || target == null) return null;
        var min = 0;
        var max = list.size() - 1;
        while (min <= max) {
            var mid = min + (max - min) / 2;
            if (target.compareTo(list.get(mid)) > 0) {
                min = mid + 1;
            } else if (target.compareTo(list.get(mid)) == 0) {
                return target;
            } else {
                max = mid - 1;
            }
        }
        return null;
    }
}
  ```

