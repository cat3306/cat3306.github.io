# go cli


<!--more-->

# 官网

https://golang.google.cn/

# 文档

https://golang.google.cn/cmd/go/

# 常见问题

https://golang.google.cn/doc/faq

# 工具库

https://github.com/golang/tools/

## Go install

https://go.dev/dl/

1. **Remove any previous Go installation**

   ```bash
   $ rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz
   ```

2. **Add /usr/local/go/bin to the `PATH` environment variable.**

   ```bash
   # vim $HOME/.profile
   #or
   # /etc/profile (for a system-wide installation)
   
   #Note: Changes made to a profile file may not apply until the next time you log into your computer. To apply the changes immediately, just run the shell commands directly or execute them from the profile using a command such as source $HOME/.profile.
   export PATH=$PATH:/usr/local/go/bin
   source $HOME/.profile
   # or
   source /etc/profile
   ```

3. **Verify that you've installed Go by opening a command prompt and typing the following command:**

   ```bash
   $ go version
   ```

   

# 目录结构

```bash
$ tree -L 1
.
├── bin
├── pkg
└── src
```

`src` 编写代码的根目录
`pkg`  `go module` 模式下 生成的依赖包根目录
`bin` 放着 `go install` 生成的二进制文件 ，可用于制作命令工具。由于要在系统的任何地方执行这里面的二进制，需要将 `~/go/bin` 加入系统可执行目录中 

```bash
vim ~/.bash_profile

export GOPATH=~/code/go
export GOBIN=~/code/go/bin
export PATH=$PATH:$GOBIN
```

当然也可以

```bash
go env -w GOPATH=~/code/go
go env -w GOBIN=~/code/go/bin
```

# 命令

```bash
The commands are:

        bug         start a bug report
        build       compile packages and dependencies
        clean       remove object files and cached files
        doc         show documentation for package or symbol
        env         print Go environment information
        fix         update packages to use new APIs
        fmt         gofmt (reformat) package sources
        generate    generate Go files by processing source
        get         add dependencies to current module and install them
        install     compile and install packages and dependencies
        list        list packages or modules
        mod         module maintenance
        run         compile and run Go program
        test        test packages
        tool        run specified go tool
        version     print Go version
        vet         report likely mistakes in packages

Use "go help <command>" for more information about a command.
```

可直接查阅，例如`go help build`

# 常见命令

## go build

```bash
# force rebuilding of packages that are already up-to-date.
# 强制重新构建二进制
$ go build -a 

# print the commands but do not run them
# 打印编译细节，不执行
$ go build -n

# the number of programs, such as build commands or
#	test binaries, that can be run in parallel.
#	The default is GOMAXPROCS, normally the number of CPUs available
# 编译时设置逻辑核数，以便并发编译。默认CPU逻辑核数 例如8
$ go build -p n # such as go build -p 1

# enable data race detection.
# Supported only on linux/amd64, freebsd/amd64, darwin/amd64, darwin/arm64, windows/amd64,
# linux/ppc64le and linux/arm64 (only for 48-bit VMA).
# 并发竞争检测
$ go build -race


# print the commands.
# 编译打印编译细节
$ go build -x





$ go build #无参数编译，到本目录
$ go build src1.go src2.go ... #指定源码文件编译
$ go build -o ? src1.go src2.go ... #指定二进制文件名
$ go build -o ? #指定二进制文件名
$ go build + package # 指定包名编译
$ go build -v #编译显示包名
$ go build -n #打印编译时所用到的所有命令，但不真正执行
$ go build -x #打印编译时所用到的所有命令
$ go build -a #强制重新构建
$ go build -race #检测go携程竞争
$ go build -gcflags="-N -l -S" #输出汇编

```

### 交叉编译

- Mac -> Linux

  ```bash
  $ CGO_ENABLED=0  GOOS=linux  GOARCH=amd64  go build main.go
  ```

- Mac -> Windows

  ```bash
  $ CGO_ENABLED=0 GOOS=windows  GOARCH=amd64  go  build  main.go
  ```

- Linux -> Mac

  ```bash
  $ CGO_ENABLED=0 GOOS=darwin  GOARCH=amd64  go build main.go
  ```

- Linux -> Windows

  ```bash
  $ CGO_ENABLED=0 GOOS=windows  GOARCH=amd64  go build main.go
  ```

- Windows -> Mac

  ```bash
  SET  CGO_ENABLED=0
  SET GOOS=darwin
  SET GOARCH=amd64
  go build main.go
  ```

- Windows -> Linux

  ```bash
  SET CGO_ENABLED=0
  SET GOOS=linux
  SET GOARCH=amd64
  go build main.go
  ```

### 编译传参

有时候需要将一些版本信息代入二进制中，这个时候就需要go的编译传参，分为两种情况

- 如果是`main`包的参数

```go
$ go build -ldflags "-X main.date=$(date -u +%Y%m%d.%H%M%S) -X main.version=1.0" main.go
```

```go
//例子
package main

import "fmt"

var date string
var version string
func main() {
	fmt.Println(date)
	fmt.Println(version)
}
```

- 如果是其它的包，就需要全路径了(相对于GOPATH路径)

```makefile
LDFLAGS += -X "github.com/pingcap/tidb/parser/mysql.TiDBReleaseVersion=$(shell git describe --tags --dirty --always)"
LDFLAGS += -X "github.com/pingcap/tidb/util/versioninfo.TiDBBuildTS=$(shell date -u '+%Y-%m-%d %H:%M:%S')"
LDFLAGS += -X "github.com/pingcap/tidb/util/versioninfo.TiDBGitHash=$(shell git rev-parse HEAD)"
LDFLAGS += -X "github.com/pingcap/tidb/util/versioninfo.TiDBGitBranch=$(shell git rev-parse --abbrev-ref HEAD)"
LDFLAGS += -X "github.com/pingcap/tidb/util/versioninfo.TiDBEdition=$(TIDB_EDITION)"

go build $(LDFLAGS)
```

## go run 

```bash
$ go run *.go #编译，并执行,但是不留下二进制文件。适合单文件，类似脚本执行方式
```

## go clean

清除编译后的二进制文件

```bash
$ go clean -n #打印要执行的命令，但不执行
$ go clean -x #打印要执行的命令,执行
```

## go install

  编译后的二进制放到$GOPATH/bin/

```bash
$ go install #一般的，在项目的根目录执行
```

## go get

```bash
go get github.com/davyxu/cellnet
go get -v github.com/davyxu/cellnet #显示操作流程的日志及信息
go get -u github.com/davyxu/cellne #下载丢失的包，但不会更新已经存在的包
```

## go test

```bash
#测试这个文件下的所有的测试用例
go test helloworld_test.go 

#测试这个文件下的所有的测试用例，显示详细流程
go test --count=1 -v helloworld_test.go 

#指定测试文件下的特定测试函数--常用
go test --count=1 -v -run ^TestFunc$ helloworld_test.go 

#指定测试文件下的特定测试函数(正则匹配有可能多个)--常用
go test --count=1 -v -run TestFunc helloworld_test.go 

#不使用缓存，指定测试文件下的特定测试函数(正则匹配有可能多个)--常用   
go test --count=1 -v -run TestFunc helloworld_test.go 
# 上面都是gopath方式，如果项目是go mod的，例如 go/src/github.com/golib 
go test --count=1 -v -run ^TestXxx$ github.com/cat3306/golib/ziputil
```

# 环境变量

`go env`

编译模式

```bash
export GO111MODULE=off #用GOPATH编译
export GO111MODULE=on #用go mod编译
```

### go 模块代理

[代理](https://goproxy.cn/)

````bash
$ go env -w GO111MODULE=on
$ go env -w GOPROXY=https://goproxy.cn,direct
````

有时需要访问私库

```bash
$ go env -w GOPRIVATE="github.com/cat3306/golib"
```

下载包用`ssh`方式

```bash
$ vim .gitconfig


[url "ssh://git@github.com/"]
	insteadOf = https://github.com/
```

导入fork 的包

go.mod 

```
replace github.com/panjf2000/gnet/v2 => github.com/cat3306/gnet/v2 v2.1.2
```



# docker  golang build

用docker容器的golang编译本地代码

需要设置`GOPROXY`

```bash
docker run --rm -v "$PWD":/go/src/mobile_cron -v "/Users/joker/code/go/src/cloud/mobile_service_project/mobile_base":/go/src/mobile_base -w /go/src/mobile_cron -it golang:1.17 bash -c "go env -w GOPROXY=https://goproxy.cn,direct && go build"
```


