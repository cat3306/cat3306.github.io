# 第二章算法分析


<!--more-->

## 最大子序列和

- 立方复杂度

````c
#include<stdio.h>
int MaxSubSeqSum(const int A[],int N){
    int thisSum,maxSum,i,j,k;
    maxSum=0;
    for(i=0;i<N;i++){
        for(j=i;j<N;j++){
            thisSum=0;
            for(k=i;k<=j;k++){
                thisSum+=A[k];
            }
            if (thisSum>maxSum){
                maxSum=thisSum;
            }
        }
    }
    return maxSum;
}
int main(){
    int a[6] ={-2,11,-4,13,-5,-2};
    int max=MaxSubSeqSum(a,6);
    printf("%d\n",max);
}
````

<!--more-->

- 平方复杂度

```c
#include <stdio.h>
int MaxSubSeqSum(const int A[], int len) {
  int maxSum = -100;
  for (int i = 0; i < len; i++) {
    int thisSum = 0;
    for (int j = i; j < len; j++) {
      thisSum += A[j];
      if (thisSum > maxSum) {
        maxSum = thisSum;
      }
    }
  }
  return maxSum;
}
int main() {
  int a[6] = {-2, 11, -4, 13, -5, -2};
  printf("%d\n", MaxSubSeqSum(a, 6));
  return 0;
}
```

- NlogN 复杂度

```c
#include <stdio.h>
int max3(int a, int b, int c) {
  int max = a > b ? a : b;
  return max > c ? max : c;
}
int MaxSubSeqSum(const int A[], int left, int right) {
  if (left == right) {
    return A[left] > 0 ? A[left] : 0;
  }
  int mid = (right - left) / 2 + left;
  MaxSubSeqSum(A, left, mid);
  MaxSubSeqSum(A, mid + 1, right);
  int sumLeft = 0;
  int sumRight = 0;
  int maxLeft = 0;
  int maxRight = 0;
  for (int i = mid; i >= left; i--) {
    sumLeft += A[i];
    if (sumLeft > maxLeft)
      maxLeft = sumLeft;
  }
  for (int i = mid + 1; i <= right; i++) {
    sumRight += A[i];
    if (sumRight > maxRight)
      maxRight = sumRight;
  }
  return max3(maxLeft, maxRight, maxLeft + maxRight);
}

int main() {
  int A[6] = {-2, 11, -4, 13, -5, -2};
  printf("%d\n", MaxSubSeqSum(A, 0, 6 - 1));
  return 0;
}
```

最后O(n)复杂度

```c
#include <stdio.h>
int MaxSubSum(const int A[], int len) {
  int maxSum, sum;
  maxSum = sum = 0;
  for (int i = 0; i < len; i++) {
    sum += A[i];
    if (sum > maxSum)
      maxSum = sum;
    else if (sum < 0)
      sum = 0;
  }
  return maxSum;
}
int main() {
  int A[6] = {-2, 11, -4, 13, -5, -2};
  printf("%d\n", MaxSubSum(A, 6));
  return 0;
}
```

## 二分法

```c
#include <stdio.h>
int BinarySearch(const int A[], int len, int target) {
  int left = 0;
  int right = len - 1;

  while (left <= right) {
    int mid = (right - left) / 2 + left;
    if (A[mid] == target) {
      return 1;
    } else if (mid < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return 0;
}
int main() {
  int A[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
  printf("%d\n", BinarySearch(A, 10, 10));
  return 0;
}
```

