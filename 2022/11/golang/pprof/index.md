# go pprof


### 测试代码
```go 
package main

import (
	"fmt"
	"net/http"
	_ "net/http/pprof"
	"runtime"
)

func main() {
	fmt.Println(runtime.NumCPU())
	go http.ListenAndServe("0.0.0.0:6060", nil)
	fmt.Println(fib(100))
	select {}

}
func fib(n int) int {
	if n <= 1 {
		return 1
	}
	return fib(n-1) + fib(n-2)
}

```

### go tool pprof

火焰图

``` bash
go tool pprof -http :8080 profile
```

### go test
采集内存

``` bash
go test -bench=. -memprofile=mem.prof
```

采集cpu

``` bash
 go test -bench=. -cpuprofile=cpu.prof
```
