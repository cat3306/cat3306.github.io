# go dlv


# dlv 

`dlv` 是`golang` 的调试工具

[github](https://github.com/go-delve/delve)

[简易教程](https://chai2010.cn/advanced-go-programming-book/ch3-asm/ch3-09-debug.html)

简易例子

```go
func main() {
	slice := make([]int, 0)
	for i := 1; i < 10; i++ {
		slice = append(slice, i)
	}
}
```

`cd` 代码根目录

```bash
$ dlv deug
Type 'help' for list of commands.
(dlv) 
```

```
(dlv) help
The following commands are available:

Running the program:
    call ------------------------ Resumes process, injecting a function call (EXPERIMENTAL!!!)
    continue (alias: c) --------- Run until breakpoint or program termination.
    next (alias: n) ------------- Step over to next source line.
    rebuild --------------------- Rebuild the target executable and restarts it. It does not work if the executable was not built by delve.
    restart (alias: r) ---------- Restart process.
    step (alias: s) ------------- Single step through program.
    step-instruction (alias: si)  Single step a single cpu instruction.
    stepout (alias: so) --------- Step out of the current function.

Manipulating breakpoints:
    break (alias: b) ------- Sets a breakpoint.
    breakpoints (alias: bp)  Print out info for active breakpoints.
    clear ------------------ Deletes breakpoint.
    clearall --------------- Deletes multiple breakpoints.
    condition (alias: cond)  Set breakpoint condition.
    on --------------------- Executes a command when a breakpoint is hit.
    toggle ----------------- Toggles on or off a breakpoint.
    trace (alias: t) ------- Set tracepoint.
    watch ------------------ Set watchpoint.

Viewing program variables and memory:
    args ----------------- Print function arguments.
    display -------------- Print value of an expression every time the program stops.
    examinemem (alias: x)  Examine raw memory at the given address.
    locals --------------- Print local variables.
    print (alias: p) ----- Evaluate an expression.
    regs ----------------- Print contents of CPU registers.
    set ------------------ Changes the value of a variable.
    vars ----------------- Print package variables.
    whatis --------------- Prints type of an expression.

Listing and switching between threads and goroutines:
    goroutine (alias: gr) -- Shows or changes current goroutine
    goroutines (alias: grs)  List program goroutines.
    thread (alias: tr) ----- Switch to the specified thread.
    threads ---------------- Print out info for every traced thread.

Viewing the call stack and selecting frames:
    deferred --------- Executes command in the context of a deferred call.
    down ------------- Move the current frame down.
    frame ------------ Set the current frame, or execute command on a different frame.
    stack (alias: bt)  Print stack trace.
    up --------------- Move the current frame up.

Other commands:
    config --------------------- Changes configuration parameters.
    disassemble (alias: disass)  Disassembler.
    dump ----------------------- Creates a core dump from the current process state
    edit (alias: ed) ----------- Open where you are in $DELVE_EDITOR or $EDITOR
    exit (alias: quit | q) ----- Exit the debugger.
    funcs ---------------------- Print list of functions.
    help (alias: h) ------------ Prints the help message.
    libraries ------------------ List loaded dynamic libraries
    list (alias: ls | l) ------- Show source code.
    source --------------------- Executes a file containing a list of delve commands
    sources -------------------- Print list of source files.
    transcript ----------------- Appends command output to a file.
    types ---------------------- Print list of types

Type help followed by a command for full documentation.
```

## `break`

设置一个断点

```
(dlv) break main.main
Breakpoint 1 set at 0x1058eea for main.main() ./main.go:3
```

## `breakpoints `

查看设置的断点

```
(dlv) breakpoints
Breakpoint runtime-fatal-throw (enabled) at 0x102dae0 for runtime.throw() /usr/local/go/src/runtime/panic.go:1188 (0)
Breakpoint unrecovered-panic (enabled) at 0x102de40 for runtime.fatalpanic() /usr/local/go/src/runtime/panic.go:1271 (0)
        print runtime.curg._panic.arg
Breakpoint 1 (enabled) at 0x1058eea for main.main() ./main.go:3 (0)
```

## `vars`

查看全部包级的变量。

因为最终的目标程序可能含有大量的全局变量，我们可以通过一个正则参数选择想查看的全局变量：

```
(dlv) vars main
runtime.main_init_done = chan bool nil
runtime.mainStarted = false
```

`continue` 执行下一个断点

```
(dlv) continue
> main.main() ./main.go:3 (hits goroutine(1):1 total:1) (PC: 0x1058eea)
     1: package main
     2: 
=>   3: func main() {
     4:         slice := make([]int, 0)
     5:         for i := 1; i < 10; i++ {
     6:                 slice = append(slice, i)
     7:         }
     8: }
(dlv) 

```

## `next` 

或者 `n`单步执行， `=>`表示所运行的位置

````
(dlv) n 
> main.main() ./main.go:5 (PC: 0x1058f0d)
     1: package main
     2: 
     3: func main() {
     4:         slice := make([]int, 0)
=>   5:         for i := 1; i < 10; i++ {
     6:                 slice = append(slice, i)
     7:         }
     8: }

````

## `locals` 

打印局部变量

```
(dlv) locals
slice = []int len: 2, cap: 2, [...]
i = 3
```

## break && condition

可以使用 `break` 和 `condition`组合命令，在循环内部条件断点，例如下面这个操作

```
(dlv) break main.go:6
Breakpoint 2 set at 0x1058f24 for main.main() ./main.go:6
(dlv) condition 2 i==7
(dlv) continue
> main.main() ./main.go:6 (hits goroutine(1):1 total:1) (PC: 0x1058f24)
     1: package main
     2: 
     3: func main() {
     4:         slice := make([]int, 0)
     5:         for i := 1; i < 10; i++ {
=>   6:                 slice = append(slice, i)
     7:         }
     8: }
(dlv) locals
slice = []int len: 6, cap: 8, [...]
i = 7
(dlv) 
```

`break main.go:6`根据行数设置一个断点，并且在`i==7`时生效

`condition 2 i==7` 2的意思是设置断点的索引(第几个断点 根据"Breakpoint 2 set at 0x1058f24 for main.main() ./main.go:6" 可知是2)

locals 一下显示 slice = []int len: 6, cap: 8, [...] i = 7

## `print ` 

打印局部变量

```
(dlv) print slice
[]int len: 6, cap: 8, [1,2,3,4,5,6]
```

 ## stack 

查看当前执行函数的栈帧信息：

```
(dlv) stack
0  0x0000000001058f24 in main.main
   at ./main.go:6
1  0x00000000010300b3 in runtime.main
   at /usr/local/go/src/runtime/proc.go:255
2  0x0000000001056261 in runtime.goexit
   at /usr/local/go/src/runtime/asm_amd64.s:1581
```

 ## goroutine or goroutines

goroutine 信息

```
(dlv) goroutine
Thread 4125012 at ./main.go:6
Goroutine 1:
        Runtime: ./main.go:6 main.main (0x1058f24)
        User: ./main.go:6 main.main (0x1058f24)
        Go: <autogenerated>:1 runtime.newproc (0x1058369)
        Start: /usr/local/go/src/runtime/proc.go:145 runtime.main (0x102fec0)

* Goroutine 1 - User: ./main.go:6 main.main (0x1058f24) (thread 4125012)
  Goroutine 2 - User: /usr/local/go/src/runtime/proc.go:367 runtime.gopark (0x10304d2) [force gc (idle)]
  Goroutine 3 - User: /usr/local/go/src/runtime/proc.go:367 runtime.gopark (0x10304d2) [GC sweep wait]
  Goroutine 4 - User: /usr/local/go/src/runtime/proc.go:367 runtime.gopark (0x10304d2) [GC scavenge wait]
[4 goroutines]
```

# 调试服务

已知一个服务已经在运行，可以用`dlv attach pid`来调试 ，例如

`gameserver` 是一个`tcp`服务，监听`8840`端口

```
▶ ./gameserver -c conf/conf.json
2022-10-13 09:22:41.593 INFO engine/server.go:29 game Server is listening on:8840
```

查看`gameserver`的`pid`

```bash
$ ps -ef | grep gameserver
501 49913 34703   0  9:22AM ttys012    0:00.03 ./gameserver -c conf/conf.json
```

`pid` 是49913

```bash
▶ dlv attach 49913    
Type 'help' for list of commands.
(dlv)
```

至此就可以愉快的调试网络服务了，以客户端发送一个鉴权包作为了例子

## 例子

```
(dlv) break OnTraffic
Command failed: Location "OnTraffic" ambiguous: github.com/cat3306/gameserver/engine.(*Server).OnTraffic, github.com/panjf2000/gnet/v2.(*BuiltinEventEngine).OnTraffic…
(dlv) break github.com/cat3306/gameserver/engine.(*Server).OnTraffic
Breakpoint 1 set at 0x1282d2f for github.com/cat3306/gameserver/engine.(*Server).OnTraffic() /Users/joker/code/go/src/github.com/cat3306/gameserver/engine/server.go:32
```

`OnTraffic` 有网络数据时回调的函数，这时候运行客户端，并且发送一个鉴权的数据包，客户端阻塞住了

`dlv`这边执行`continue`，可以看到`=>`指向`OnTraffic`,表示服务运行到这里了

```
(dlv) continue
> github.com/cat3306/gameserver/engine.(*Server).OnTraffic() /Users/joker/code/go/src/github.com/cat3306/gameserver/engine/server.go:32 (hits goroutine(25):1 total:1) (PC: 0x1282d2f)
Warning: debugging optimized function
    27: func (s *Server) OnBoot(e gnet.Engine) (action gnet.Action) {
    28:         s.eng = e
    29:         glog.Logger.Sugar().Infof("game Server is listening on:%d", conf.GameConfig.Port)
    30:         return
    31: }
=>  32: func (s *Server) OnTraffic(c gnet.Conn) gnet.Action {
    33:         defer func() {
    34:                 err := recover()
    35:                 if err != nil {
    36:                         glog.Logger.Sugar().Errorf("OnTraffic panic %v", err)
    37:                 }
(dlv) 
```

`netx` or `n`单步运行

```
> github.com/cat3306/gameserver/engine.(*Server).OnTraffic() /Users/joker/code/go/src/github.com/cat3306/gameserver/engine/server.go:40 (PC: 0x1282d9b)
Warning: debugging optimized function
    35:                 if err != nil {
    36:                         glog.Logger.Sugar().Errorf("OnTraffic panic %v", err)
    37:                 }
    38:         }()
    39:         s.eng.CountConnections()
    40:         context, err := protocol.Decode(c)
 => 41:         if err != nil {
    42:                 glog.Logger.Sugar().Warnf("OnTraffic err:%s", err.Error())
    43:                 return gnet.None
    44:         }
    45:         if context == nil {

```

打印`context`

`print context`

```
(dlv) print context
*github.com/cat3306/gameserver/protocol.Context {
        Payload: []uint8 len: 552, cap: 65526, [123,34,67,105,112,104,101,114,84,101,120,116,34,58,34,79,84,70,105,68,102,56,69,102,75,109,52,75,83,81,80,117,56,76,101,98,43,98,103,82,43,78,119,97,120,75,81,56,70,47,79,65,55,54,111,66,100,55,89,49,79,121,85,122,...+488 more],
        CodeType: Json (2),
        Proto: 2105448169,
        Conn: github.com/panjf2000/gnet/v2.Conn(*github.com/panjf2000/gnet/v2.conn) *{
                ctx: interface {} nil,
                peer: golang.org/x/sys/unix.Sockaddr(*golang.org/x/sys/unix.SockaddrInet6) ...,
                localAddr: net.Addr(*net.TCPAddr) ...,
                remoteAddr: net.Addr(*net.TCPAddr) ...,
                loop: *(*"github.com/panjf2000/gnet/v2.eventloop")(0xc00015acf0),
                outboundBuffer: *(*"github.com/panjf2000/gnet/v2/pkg/buffer/elastic.Buffer")(0xc000242cd0),
                pollAttachment: *(*"github.com/panjf2000/gnet/v2/internal/netpoll.PollAttachment")(0xc000241d10),
                inboundBuffer: (*"github.com/panjf2000/gnet/v2/pkg/buffer/elastic.RingBuffer")(0xc0001b0418),
                buffer: []uint8 len: 0, cap: 64974, [],
                fd: 19,
                isDatagram: false,
                opened: true,
                id: "TljczpztN",
                property: map[string]string [],
                propertyLock: (*sync.RWMutex)(0xc0001b0460),},
        connMgr: *github.com/cat3306/gameserver/protocol.ConnManager nil,}
```

`print`这个命令非常强大,可以打印具体的字段，比如

```
(dlv) print context.Proto
2105448169
(dlv) print context.CodeType
Json (2)
(dlv) print context.Conn.id
"TljczpztN"
(dlv) print context.Conn.remoteAddr
net.Addr(*net.TCPAddr) *{
        IP: net.IP len: 16, cap: 16, [0,0,0,0,0,0,0,0,0,0,255,255,127,0,0,1],
        Port: 54783,
        Zone: "",}
(dlv) 

```

设置另一个断点，然后`continue`进入执行`handler`的函数

```
(dlv) break ExeHandler
Breakpoint 3 set at 0x12829f2 for github.com/cat3306/gameserver/engine.(*HandlerManager).ExeHandler() /Users/joker/code/go/src/github.com/cat3306/gameserver/engine/handlermgr.go:190

(dlv) continue
> github.com/cat3306/gameserver/engine.(*HandlerManager).ExeHandler() /Users/joker/code/go/src/github.com/cat3306/gameserver/engine/handlermgr.go:190 (hits goroutine(25):1 total:1) (PC: 0x12829f2)
Warning: debugging optimized function
   185:                 f(ctx, nil)
   186:                 return nil
   187:         }
   188:         return HandlerNotFound
   189: }
=> 190: func (h *HandlerManager) ExeHandler(auth bool, ctx *protocol.Context) {
   191:         err := h.exeSyncHandler(auth, ctx)
   192:         if !errors.Is(err, HandlerNotFound) {
   193:                 return
   194:         }
   195:         err = h.exeAsyncHandler(auth, ctx)
```

中间过程和前面一样，直接跳到`ClientAuth` 函数，这是真正执行鉴权的逻辑

```
(dlv) break ClientAuth
Breakpoint 4 set at 0x127c652 for github.com/cat3306/gameserver/router.(*ClientAuth).ClientAuth() /Users/joker/code/go/src/github.com/cat3306/gameserver/router/auth.go:37
(dlv) continue
> github.com/cat3306/gameserver/router.(*ClientAuth).ClientAuth() /Users/joker/code/go/src/github.com/cat3306/gameserver/router/auth.go:37 (hits goroutine(25):1 total:1) (PC: 0x127c652)
Warning: debugging optimized function
    32: var req struct {
    33:         CipherText []byte `json:"CipherText"`
    34:         Text       string `json:"Text"`
    35: }
    36: 
=>  37: func (c *ClientAuth) ClientAuth(ctx *protocol.Context, v interface{}) {
    38:         err := ctx.Bind(&req)
    39:         if err != nil {
    40:                 glog.Logger.Sugar().Errorf("param err:%s", err.Error())
    41:         }
    42:         //glog.Logger.Sugar().Errorf("req.CipherText:%s,c.rawPrivateKey:%s", req.CipherText, c.rawPrivateKey)
(dlv) 

```

一步步运行

````
(dlv) n
> github.com/cat3306/gameserver/router.(*ClientAuth).ClientAuth() /Users/joker/code/go/src/github.com/cat3306/gameserver/router/auth.go:39 (PC: 0x127c68f)
Warning: debugging optimized function
    34:         Text       string `json:"Text"`
    35: }
    36: 
    37: func (c *ClientAuth) ClientAuth(ctx *protocol.Context, v interface{}) {
    38:         err := ctx.Bind(&req)
=>  39:         if err != nil {
    40:                 glog.Logger.Sugar().Errorf("param err:%s", err.Error())
    41:         }
    42:         //glog.Logger.Sugar().Errorf("req.CipherText:%s,c.rawPrivateKey:%s", req.CipherText, c.rawPrivateKey)
    43:         text := cryptoutil.RsaDecrypt(req.CipherText, c.rawPrivateKey)
    44:         if string(text) != req.Text {
(dlv) 

````

`ctx.Bind(&req)` 是将字节流映射到结构上，类似于`gin context.Bind()`，打印req，会发现客户端发来两个字段

一个是被公钥加了密的密文二进制流`CipherText`，一个是明文`Text`其值为`life is short`

```
(dlv) print req
struct { CipherText []uint8 "json:\"CipherText\""; Text string "json:\"Text\"" } {
        CipherText: []uint8 len: 384, cap: 384, [57,49,98,13,255,4,124,169,184,41,36,15,187,194,222,111,230,224,71,227,112,107,18,144,240,95,206,3,190,168,5,222,216,212,236,148,204,52,225,73,181,250,211,246,199,214,42,29,173,72,121,43,235,162,22,72,61,18,45,24,130,202,208,84,...+320 more],
        Text: "life is short",}
(dlv) 
```

鉴权的逻辑也很简单，服务器用自己的私钥去解客户端发过来的密文，解出来的明文和客户端发的明文一样，说明鉴权成功，如果不一样则鉴权失败

鉴权成功后，会给`Conn` 写一个`auth` 的`property`

```
> github.com/cat3306/gameserver/router.(*ClientAuth).ClientAuth() /Users/joker/code/go/src/github.com/cat3306/gameserver/router/auth.go:50 (PC: 0x127c8d0)
Warning: debugging optimized function
    45:                 glog.Logger.Sugar().Errorf("认证失败!")
    46:                 ctx.Send(JsonRspErr("auth failed"))
    47:                 time.Sleep(time.Second)
    48:         } else {
    49:                 ctx.Conn.SetProperty(protocol.Auth, "ok")
=>  50:                 ctx.Send(JsonRspOK("auth ok"))
    51:         }
    52: }
(dlv) print ctx.Conn.property
map[string]string [
        "auth": "ok", 
]
(dlv) 
```

至此，一个鉴权的全部过程都详细`debug`了

不得不说`dlv`这个调试工具非常强大






