# go 异步抢占调度


<!--more-->

# go 的异步抢占调度

# go 1.13

在`go 1.13 `（包括1.13）之前，并不是真正意思上的抢占。什么是真正的抢占式调度，为了探究这个问题，网上广为流传这么一段代码

```go
package main 

import "fmt"
func main() {
	go func(i int) {
		for {
			i++
			fmt.Println(i)
		}
	}(0)
	for {

	}
}
```



在`go 1.13`中这段代码的表象是，程序会持续输出，过不了多久便停止了输出，但是程序并没有终止，通过`top`命令查看会发现程序的cpu负载很高，这是因为`for{}`导致，负责打印的`goroutine`应该是在某处"卡住"了。

## 调试

本地高于`go 1.13`，可以用`GODEBUG=asyncpreemptoff=1 go run main.go`来调试。也可以在`docker`里面跑一个`go 1.13`的版本

docker 里面调试

1. ```
   $ docker pull golang:1.13
   ```

2. ```
   $ docker run -it golang:1.13
   ```

3. 在容器里面，安装vim

   ```
   root@a2ea6ba48cc9:/go# apt-get update
   
   root@a2ea6ba48cc9:/go# apt-get install vim
   ```

   

4. 还需要一个`dlv`的golang调试工具

   ```
   go env -w GO111MODULE=on
   go env -w GOPROXY=https://goproxy.cn,direct
   go get github.com/go-delve/delve/cmd/dlv@latest
   ```

   

5. 在容器里面复制上面代码

   ```
   root@a2ea6ba48cc9:/go/src# vim main.go
   ```

6. 运行

   ```
   root@a2ea6ba48cc9:/go/src# go run main.go 
   ```

   




