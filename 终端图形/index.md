# 终端图形


<!--more-->

```c
//c
#include <stdio.h>
#define Row 21
int arry[Row][Row];
void init_arry()
{
    for (int i = 0; i < Row; i++)
    {
        for (int j = 0; j < Row; j++)
        {
            arry[i][j] = 0;
        }
    }
}
void func()
{
    int mid = Row / 2;
    int left = mid;
    int right = mid;
    int flag = 0;
    for (int i = 0; i < Row; i++)
    {
        if (i == 0 || i == Row - 1) //
        {
            arry[i][mid] = 1;
        }
        else
        {
            if (flag)
            {
                left++;
                right--;
            }
            else
            {
                left--;
                right++;
            }
            arry[i][left] = 1;
            arry[i][right] = 1;
            if (left == 0 && right == Row - 1)
            {
                flag = 1;
            }
        }
    }
}
void print()
{
    for (int i = 0; i < Row; i++)
    {
        for (int j = 0; j < Row; j++)
        {
            if (arry[i][j] == 0)
            {
                printf(" ");
            }
            else
            {
                printf("*");
            }
        }
        printf("\n");
    }
}
int main()
{
    init_arry();
    func();
    print();
}

          *
         * *
        *   *
       *     *
      *       *
     *         *
    *           *
   *             *
  *               *
 *                 *
*                   *
 *                 *
  *               *
   *             *
    *           *
     *         *
      *       *
       *     *
        *   *
         * *
          *
```




