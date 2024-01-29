# 正则表达式


<!--more-->

## 引入问题

为什么需要正则表达式。随着工作的深入，你会发现随时随地都有可能需要正则表达来提高工作效率。试想你需要过滤你想要的日志内容来排查某个bug，你会怎么做？命令grep 后面可以是一个正则表达，用于过滤匹配规则的日志类容。另外go的测试命令也有正则的影子

```go
go test -v -run \^Test1\$ test_test.go 
//运行Test1 的测试用例
```

当要有规则地替换大量文本类容，使用正则表达是个很好的选择。比如下面这个，查出以下电话号码所对应的用户信息。最直接方法就是拼接裸sql，可是需要处理每个电话号码，如13575453505变成 "13575453505", 每个电话号码需要引号，和一个逗号。最最最笨的方法就是一个个加，浪费时间。这时候用正则可以几秒搞定 ,替换时需要使用到捕获组，可以用()表示,并且$n进行引用，例如([0-9]),的引用是$1

(^[0-9]{11}\$)
"\$1",

## 语法

1. 123+abc，可以匹配1233abc，123abc，123333abc，+号表示前面的字符至少出现有一个

   ```go
   func Test2(t *testing.T) {
   	re, err := regexp.Compile("123+abc")
   	if err!=nil{
   		fmt.Println(err)
   		return
   	}
   	fmt.Println(re.MatchString("1233abc"))
   	fmt.Println(re.MatchString("123abc"))
   	fmt.Println(re.MatchString("123333abc"))
   	fmt.Println(re.MatchString("12abc"))
   }
   ⋊> ~/g/s/j/g/regular_expression go test --count=1 -v -run \^Test2\$ test_test.go
   true
   true
   true
   false
   ```

2. 123*abc，可以匹配1233abc，123abc，123333abc，12abc, *号表示前面的字符出现有0个，1个，或多个

   ```go
   func Test3(t *testing.T) {
   	re, err := regexp.Compile("123*abc")
   	if err!=nil{
   		fmt.Println(err)
   		return
   	}
   	fmt.Println(re.MatchString("1233abc"))
   	fmt.Println(re.MatchString("123abc"))
   	fmt.Println(re.MatchString("123333abc"))
   	fmt.Println(re.MatchString("12abc"))
   }
   ⋊> ~/g/s/j/g/regular_expression go test --count=1 -v -run \^Test3\$ test_test.go
   true
   true
   true
   true
   ```

3. 123?abc，可以匹配123abc，12abc，?号代表前面的字符最多能出现一次(0次，1次)

   ```go
   func Test4(t *testing.T) {
   	re, err := regexp.Compile("123?abc")
   	if err!=nil{
   		fmt.Println(err)
   		return
   	}
   	fmt.Println(re.MatchString("1233abc"))
   	fmt.Println(re.MatchString("123abc"))
   	fmt.Println(re.MatchString("123333abc"))
   	fmt.Println(re.MatchString("12abc"))
   }
   ⋊> ~/g/s/j/g/regular_expression go test --count=1 -v -run \^Test4\$ test_test.go
   false
   true
   false
   true
   ```

4. 匹配[...]中所有的字符,例如[hw]匹配 "hello,world"

   ```go
   func Test5(t *testing.T) {
   	re, err := regexp.Compile("[hw]")
   	if err!=nil{
   		fmt.Println(err)
   		return
   	}
   	fmt.Println(re.MatchString("hello,world"))
   	fmt.Println(re.MatchString("huawei"))
   	fmt.Println(re.MatchString("uaei"))
   }
   
   ⋊> ~/g/s/j/g/regular_expression go test --count=1 -v -run \^Test5\$ test_test.go                                                                                                           15:16:56
   === RUN   Test5
   true
   true
   false
   --- PASS: Test5 (0.00s)
   PASS
   ok      command-line-arguments  0.007s
   ```

5. [^....]匹配除了....的字符

   ```go
   func Test6(t *testing.T) {
   	re, err := regexp.Compile("[^a]")
   	if err != nil {
   		fmt.Println(err)
   		return
   	}
   	fmt.Println(re.MatchString("a"))
   	fmt.Println(re.MatchString("bc"))
   	fmt.Println(re.MatchString("abc"))
   	fmt.Println(re.MatchString("hw"))
   }
   ⋊> ~/g/s/j/g/regular_expression go test --count=1 -v -run \^Test6\$ test_test.go                                                                                                          15:27:59
   === RUN   Test6
   false
   true
   true
   true
   --- PASS: Test6 (0.00s)
   PASS
   ok      command-line-arguments  0.006s
   ```

6. ^$,^是开始匹配，$结束匹配,可用于准确匹配

   ```go
   func Test7(t *testing.T) {
   	re, err := regexp.Compile("^[a$bc]")
   	if err != nil {
   		fmt.Println(err)
   		return
   	}
   	fmt.Println(re.MatchString("abc"))
   	fmt.Println(re.MatchString("abcssdda"))
   	fmt.Println(re.MatchString("afbgch"))
   	fmt.Println(re.MatchString("va"))
   }
   ⋊> ~/g/s/j/g/regular_expression go test --count=1 -v -run \^Test7\$ test_test.go                                                                                                           16:10:32
   === RUN   Test7
   true
   false
   false
   false
   --- PASS: Test7 (0.00s)
   PASS
   ok      command-line-arguments  0.008s
   ```

7. [A-Z]代表一个区间，匹配所有大写字母,[a-z]匹配所有的小写字母

   ```go
   func Test8(t *testing.T) {
   	re, err := regexp.Compile("[a-z]")
   	if err != nil {
   		fmt.Println(err)
   		return
   	}
   	fmt.Println(re.MatchString("abc"))
   	fmt.Println(re.MatchString("abcssdda"))
   	fmt.Println(re.MatchString("afbgch"))
   	fmt.Println(re.MatchString("va"))
   	fmt.Println(re.MatchString("aA"))
   }
   ⋊> ~/g/s/j/g/regular_expression go test --count=1 -v -run \^Test8\$ test_test.go                                                                                                           16:14:30
   === RUN   Test8
   true
   true
   true
   true
   false
   --- PASS: Test8 (0.00s)
   PASS
   ok      command-line-arguments  0.006s
   ```

8. 匹配所有。\s 是匹配所有空白符，包括换行，\S 非空白符，包括换行

   ```bash
   func Test9(t *testing.T) {
   	re, err := regexp.Compile("[\\s]")
   	if err != nil {
   		fmt.Println(err)
   		return
   	}
   	fmt.Println(re.MatchString("abc  "))
   	fmt.Println(re.MatchString("abcssdda\n"))
   	fmt.Println(re.MatchString("afbgch"))
   	fmt.Println(re.MatchString("va"))
   	fmt.Println(re.MatchString("A"))
   }
   ⋊> ~/g/s/j/g/regular_expression go test -v -run \^Test9\$ test_test.go                                                                                                            16:17:54
   === RUN   Test9
   true
   true
   false
   false
   false
   --- PASS: Test9 (0.00s)
   PASS
   ok      command-line-arguments  0.012s
   ```

   ```go
   func Test9(t *testing.T) {
   	re, err := regexp.Compile("[\\S]")
   	if err != nil {
   		fmt.Println(err)
   		return
   	}
   	fmt.Println(re.MatchString("\n"))
   	fmt.Println(re.MatchString("   "))
   	fmt.Println(re.MatchString("abcssdda\n"))
   	fmt.Println(re.MatchString("afbgch"))
   	fmt.Println(re.MatchString("va"))
   	fmt.Println(re.MatchString("A"))
   }
   ⋊> ~/g/s/j/g/regular_expression go test --count=1 -v -run \^Test9\$ test_test.go                                                                                                            16:19:34
   === RUN   Test9
   false
   false
   true
   true
   true
   true
   --- PASS: Test9 (0.00s)
   PASS
   ok      command-line-arguments  0.006s
   ```

9. \w 匹配字母、数字、下划线。等价于 [A-Za-z0-9_]

10. {n}  n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o

    ```go
    func Test10(t *testing.T) {
    	re, err := regexp.Compile("jav(a){2}")
    	if err != nil {
    		fmt.Println(err)
    		return
    	}
    	fmt.Println(re.MatchString("java"))
    	fmt.Println(re.MatchString("javaa"))
    	fmt.Println(re.MatchString("javaaa"))
    	fmt.Println(re.MatchString("jav"))
    }
    ⋊> ~/g/s/j/g/regular_expression go test  --count=1 -v -run \^Test10\$ test_test.go                                                                                                18:57:33
    === RUN   Test10
    false
    true
    false
    false
    --- PASS: Test10 (0.00s)
    PASS
    ok      command-line-arguments  0.006s
    ```

11. {n,m} m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。例如，"o{1,3}" 将匹配 "fooooood" 中的前三个 o。'o{0,1}' 等价于 'o?'。请注意在逗号和两个数之间不能有空格。

    ```go
    func Test11(t *testing.T) {
    	re, err := regexp.Compile("jav(a){2,3}$")
    	if err != nil {
    		fmt.Println(err)
    		return
    	}
    	fmt.Println(re.MatchString("java"))
    	fmt.Println(re.MatchString("javaa"))
    	fmt.Println(re.MatchString("javaaa"))
    	fmt.Println(re.MatchString("javaaaa"))
    	fmt.Println(re.MatchString("jav"))
    }
    ⋊> ~/g/s/j/g/regular_expression go test  --count=1 -v -run \^Test11\$ test_test.go                                13:46:57
    === RUN   Test11
    false
    true
    true
    false
    false
    --- PASS: Test11 (0.00s)
    PASS
    ok      command-line-arguments  0.007s
    ```

12. {n,} n 是一个非负整数。至少匹配n 次。例如，'o{2,}' 不能匹配 "Bob" 中的 'o'，但能匹配 "foooood" 中的所有 o。'o{1,}' 等价于 'o+'。'o{0,}' 则等价于 'o*'

    ```go
    func Test12(t *testing.T) {
    	re, err := regexp.Compile("[a-z]{5,}")
    	if err != nil {
    		fmt.Println(err)
    		return
    	}
    	fmt.Println(re.MatchString("ja"))
    	fmt.Println(re.MatchString("javaa"))
    	fmt.Println(re.MatchString("javaaa"))
    	fmt.Println(re.MatchString("javaaaa"))
    	fmt.Println(re.MatchString("av"))
    	fmt.Println(re.MatchString("j"))
    }
    ⋊> ~/g/s/j/g/regular_expression go test  --count=1 -v -run \^Test12\$ test_test.go                                13:56:54
    === RUN   Test12
    false
    true
    true
    true
    false
    false
    --- PASS: Test12 (0.00s)
    PASS
    ok      command-line-arguments  0.007s
    ```

13. | 两边的字母或子式都可匹配匹配，可以理解或

    ```go
    func Test16(t *testing.T) {
    	re, err := regexp.Compile("^(jav)$|^(cor)$")
    	if err != nil {
    		fmt.Println(err)
    		return
    	}
    	fmt.Println(re.MatchString("jav"))
    	fmt.Println(re.MatchString("cor"))
    	re, err = regexp.Compile("b|corn")
    	if err != nil {
    		fmt.Println(err)
    		return
    	}
    
    	fmt.Println(re.MatchString("born"))
    	fmt.Println(re.MatchString("corn"))
    }
    ⋊> ~/g/s/j/g/regular_expression go test  --count=1 -v -run \^Test16\$ test_test.go                                11:02:03
    === RUN   Test16
    true
    true
    true
    true
    --- PASS: Test16 (0.00s)
    PASS
    ok      command-line-arguments  0.009s
    
    ```

14. \b匹配一个单词的边界

- ##### 特殊字符,如果需要匹配下面特殊字符本身，需要转义字符转义 \

|  $   |      匹配输入字符的结束位置     |
| :---:| ----------------------------|
|  ()  |  标记一个子表达式的开始结束位置  |
|  *   |  匹配前面的子表达式0个，或多个   |
|  +   |    匹配前面的子表达式最少一个    |
|  .   | 匹配除换行符 \n 之外的任何单字符 |
|  [   |    标记一个中括号表达式的开始    |
|  ?   |    匹配前面的子表达式最多一个    |
|  \   |      将下个字符变成特殊字符      |
|  ^   |      匹配输入字符的开始位置      |
|  {   |      标记限定符表达式的开始      |
|  \|  |            选择性匹配           |
|  /b  |        匹配一个单词的边界        |

- ##### 修饰符

|  i   | ignore - 不区分大小写 |
| :---: | --------------------- |
|  g   | global - 全局匹配     |
|  m   | multi line -多行匹配  |

## 常用正则表示

- 邮箱

  1. 支持英文，多级域名 只允许英文字母、数字、下划线、英文句号、以及中划线组成

     ```
     ^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$
     ```

  2. 支持中文、字母、数字，域名只允许英文域名

     ```bash
     ^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$
     ```

- 电话号码

  ```
  ^1(?:3\d|4[4-9]|5[0-35-9]|6[67]|7[013-8]|8\d|9\d)\d{8}$
  ```

- 匹配一行字符串

  ````
  (^\S{1,}$)
  需求是 下面随机字符串，需要替换成 "Ericwang100",
  Ericwang100
  FSRunner
  VK李康
  尚文欢shangwh
  lowkeyd
  stonezou
  ````

- 匹配每一行并且加上引号

   ``` bash
   \b(\w+)\b
   "$1",
   aaaaa
   asdasdsad
   asdasdsa
   ```

