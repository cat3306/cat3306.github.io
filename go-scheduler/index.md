# go scheduler


<!--more-->

## 系统调度

- 进程调度

最近在看一些操作系统的文章，有篇文章说进程是对cpu的抽象，细细想来确实如此。计算机的问题都可以加一个中间层来解决。程序员写的程序，经过编译器编译(编译性语言)会变成一堆低级的，难以阅读的机器指令。机器按照指令的顺序，取指执行。这个指令集合可以姑且理解为进程，然而操作系统运行着上百个进程，如何管理这些进程变得至关重要。
CPU的速度永远是其他设备无法比拟的，一个进程执行过程中可能会陷入一些比较耗时的操作，比如磁盘IO，这时候就需要把这个进程调走，让其他的进程获取CPU资源，来提升整个系统的性能。随着而来的问题是如何调度这些进程。

- 线程调度

事实上进程并不是系统调度的实体，线程才是，线程是更小的单位，线程是轻量级进程。这不难理解，一个进程会有很多操作，这些操作是可以并行进行的，有些很消耗CPU，有些是IO费时操作，如果遇到IO的操作，如果系统是调度进程的策略会把整个进程给调走，那么这个进程就直接卡住了，用户体验肯定是糟糕的。

### 线程切换

系统调度线程的依据是线程的状态，线程有三种状态(简化模型)，`Waiting`，`Runnable` `Executing`

- Waiting 等待状态，线程等待某个事件的返回，例如网络数据的返回，磁盘数据的返回，调用系统API；内存同步访问条件atomic, mutexes等
- Runnable 状态，就绪状态，获取CPU资源就可以运行
- Executing 正在执行的状态，CPU正在执行的线程

线程可以分为计算型，和IO型。也可以说是CPU密集型，IO密集型。

IO型需要外部资源，如网络数据，磁盘数据，或者是内存的同步访问原语。线程陷入IO操作后，系统调度会让线程跳到Waiting 状态

CPU密集型，做计算任务，这种类型的线程会消耗CPU，例如一个程序执行一个死循环的打印语句，可以观察他的CPU占有率非常的高

对于这两种的线程切换态度是不一样的，CPU密集型，由于一直有指令需要CPU执行。调度策略就不会考虑这种类型的进程。对于IO密集型的线程就不一样了，在等待IO资源的时候，调度策略会把当前线程换下来，让其他的线程使用CPU。

对于系统调度来说，最重要的是尽可能利用CPU，尽可能让所有的CPU核心有事情做

if you have a program that is focused on IO-Bound work, then context switches are going to be an advantage. Once a Thread moves into a Waiting state, another Thread in a Runnable state is there to take its place. This allows the core to always be doing work. This is one of the most important aspects of scheduling. Don’t allow a core to go idle if there is work (Threads in a Runnable state) to be done.

翻译过来就是说，如果程序陷入IO时，上下文切换将是个优点






