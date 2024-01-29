# 八皇问题


<!--more-->

八皇问题(Eight queens),问题表述为：在8×8格的国际象棋上摆放8个皇后，使其不能互相攻击，即任意两个皇后都不能处于同一行、同一列或同一斜线上，问有多少种摆法。高斯认为有76种方案。1854年在柏林的象棋杂志上不同的作者发表了40种不同的解，后来有人用图论的方法解出92种结果。如果经过±90度、±180度旋转，和对角线对称变换的摆法看成一类，共有42类。计算机发明后，有多种计算机语言可以编程解决此问题(摘自百度)。

```go
func TestEightQueen(t *testing.T) {
	queen := []int{-1, -1, -1, -1, -1, -1, -1, -1}
	row := 0
	size := len(queen)
	for row >= 0 && row < size {
		pos := search(row, queen, size)
		if pos < 0 {
			queen[row] = -1
			row--
		} else {
			queen[row] = pos
			row++
		}
	}
  if row == -1 {
		fmt.Println("未找到方案")
		return
	}
	draw(queen)
}
func draw(queen []int) {
	for i := 0; i < len(queen); i++ {
		for j := 0; j < len(queen); j++ {
			if j == queen[i] {
				fmt.Printf("* ")
			} else {
				fmt.Printf("- ")
			}
		}
		fmt.Println()
	}
}
func search(row int, queen []int, size int) int {
	start := queen[row] + 1
	for column := start; column < size; column++ {
		if getTarget(column, queen, row) {
			return column
		}
	}
	return -1
}
func getTarget(column int, queen []int, row int) bool {
	for i := 1; i <= row; i++ {
		if queen[row-i] == column || queen[row-i] == column-i || queen[row-i] == column+i {

			return false
		}
	}
	return true
}

=== RUN   TestEightQueen
* - - - - - - - 
- - - - * - - - 
- - - - - - - * 
- - - - - * - - 
- - * - - - - - 
- - - - - - * - 
- * - - - - - - 
- - - * - - - - 
--- PASS: TestEightQueen (0.00s)
PASS
```

