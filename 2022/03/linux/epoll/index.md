# epoll


<!--more-->

# 概述

服务器上的进程可能有上千条连接，如何快速的知道哪条连接来了网络数据，正是`epoll`要解决的问题。在早期，linux内核采用的`select` 和 `poll`。结论是在连接数很多，活跃的连接少的情况下`epoll`有高效的性能优势，原因之一是`epoll`唤醒进程后，只需要查看就绪（有数据）的`socket`，而不是遍历整个socket集合。多路复用指的是进程的复用，本文重点描述`epoll`原理[^1]

[^1]: https://zhuanlan.zhihu.com/p/487497556 本文借鉴了这篇文章，由于处于学习阶段有很多相似的地方



# Socket 数据结构

当服务器执行`accept`后会阻塞，等待客户端的连接请求。如果客户端发起连接，三次握手后，服务器会创建一个新的`socket`，用于客户端和服务器的通信。本着linux一切皆为文件的原则，`socket` 也是一种文件描述符。并且把它加入当前进程的打开文件列表中

{{% figure class="center" src="/img/linux/socket.png" title="socket" alt="img" %}}

`socket` 内核对象具体的细节如图所示

{{% figure class="center" src="/img/linux/socket_detail.png" title="socket_detail" alt="img" %}}

## 创建 Socket

用户（用户程序）调用`accept`系统调用后，当接收到连接时会创建`socket`对象。其源码位于[net/socket.c](https://github.com/torvalds/linux/blob/787af64d05cd528aac9ad16752d11bb1c6061bb9/net/socket.c#L1859)截取部分代码所示:

````c
SYSCALL_DEFINE4(accept4, int, fd, struct sockaddr __user *, upeer_sockaddr,
        int __user *, upeer_addrlen, int, flags)
{
    struct socket *sock, *newsock;

    //根据 fd 查找到监听的 socket
    sock = sockfd_lookup_light(fd, &err, &fput_needed);

    // 申请并初始化新的 socket
    newsock = sock_alloc();
    newsock->type = sock->type;
    newsock->ops = sock->ops;

    //申请新的 file 对象，并设置到新 socket 上
    newfile = sock_alloc_file(newsock, flags, sock->sk->sk_prot_creator->name);
    ......

    //接收连接
    err = sock->ops->accept(sock, newsock, sock->file->f_flags);

    //添加新文件到当前进程的打开文件列表
    fd_install(newfd, newfile);
````

上述代码中可以看出，先找出一个listen状态的`socket`，随即`malloc` 出一个新的socket对象，拷贝`type`，`ops`（指针拷贝）。由前面的`socket`数据结构可知`ops`是`struct proto_ops inet_stream_ops`。如图所示：

{{% figure class="center" src="/img/linux/socket_ops.png" title="socket_ops" alt="img" %}}

其中 `inet_stream_ops` 起源码位于[net/ipv4/af_inet.c](https://github.com/torvalds/linux/blob/787af64d05cd528aac9ad16752d11bb1c6061bb9/net/ipv4/af_inet.c#L1020)，截取部分代码如下

```
const struct proto_ops inet_stream_ops = {
	.family		   = PF_INET,
	.owner		   = THIS_MODULE,
	.release	   = inet_release,
	.bind		   = inet_bind,
	.connect	   = inet_stream_connect,
	.socketpair	   = sock_no_socketpair,
	.accept		   = inet_accept,
	.getname	   = inet_getname,
	.poll		   = tcp_poll,
	......
```

## 创建file对象

前面提到的`socket`数据结构中，有个重要字段`file`。在`accept`方法里面会调用`sock_alloc_file`来初始化，然后赋值给新的socket对象，如图所示：

{{% figure class="center" src="/img/linux/socket_file.png" title="socket_file" alt="img" %}}

`sock_alloc_file` 源码如下：

```c
struct file *sock_alloc_file(struct socket *sock, int flags,
    const char *dname)
{
    struct file *file;
    file = alloc_file(&path, FMODE_READ | FMODE_WRITE,
            &socket_file_ops);
    ......
    sock->file = file;
}
```

`sock_alloc_file` 里面回调用` alloc_file`。注意在 alloc_file 方法中，把 socket_file_ops 函数集合一并赋到了新 file->f_op 里了。其源码位于[fs/file_table.c](https://github.com/torvalds/linux/blob/787af64d05cd528aac9ad16752d11bb1c6061bb9/fs/file_table.c#L224)

```c

struct file *alloc_file(struct path *path, fmode_t mode,
        const struct file_operations *fop)
{
    struct file *file;
    file->f_op = fop;
    ......
}
```

可以看出，在`accept`里创建的新`socket` 的`file->f_op->poll` 函数指向的是 `sock_poll`。另外file对象内部有个socket指针，指向socket对象。

## 接收连接

`socket`结构中除了`file`，还有一个核心的成员`sock`。其源码位于[include/linux/net.h](https://github.com/torvalds/linux/blob/787af64d05cd528aac9ad16752d11bb1c6061bb9/include/linux/net.h#L114)。部分源码如下：

```c
struct socket {
	socket_state		state;

	short			type;

	unsigned long		flags;

	struct file		*file;
	struct sock		*sk;
	const struct proto_ops	*ops;

	struct socket_wq	wq;
};
```

`sock` 是`socket`的核心对象。发送队列、接收队列、等待队列等核心数据结构都位于此
在`accept` 函数中，其源码位于[net/socket.c](https://github.com/torvalds/linux/blob/787af64d05cd528aac9ad16752d11bb1c6061bb9/net/socket.c#L1859)，部分源码如下

```c
SYSCALL_DEFINE4(accept4, ...)
    ...
    //1.3 接收连接
    err = sock->ops->accept(sock, newsock, sock->file->f_flags);
}
```

`sock->ops->accept` 对应的方法是 `inet_accept`。它执行的时候会从握手队列里直接获取创建好的 sock。 sock 对象的完整创建过程涉及到三次握手。 struct sock 初始化过程中用到的一个函数：

```c
void sock_init_data(struct socket *sock, struct sock *sk)
{
    sk->sk_wq   =   NULL;
    sk->sk_data_ready   =   sock_def_readable;
}
```

**在这里把 sock 对象的 sk_data_ready 函数指针设置为 sock_def_readable**。后面还会提到这个sock_def_readable

## 加入当前进程的打开文件列表中

当`file`,`socket`,`sock`等对象初始化完毕后，还需要将其挂到当前进程的打开文件列表中

```c
void fd_install(unsigned int fd, struct file *file)
{
    __fd_install(current->files, fd, file);
}

void __fd_install(struct files_struct *files, unsigned int fd,
        struct file *file)
{
    ...
    fdt = files_fdtable(files);
    BUG_ON(fdt->fd[fd] != NULL);
    rcu_assign_pointer(fdt->fd[fd], file);
}
```



# epoll数据结构

`epoll` 暴露出来的API非常简单，`epoll_create`,`epoll_ctl`,`epoll_wait`

- epoll_create：创建一个epoll对象
- epoll_ctl：管理epoll_item，epoll_item与socket关联
- epoll_wait：等待管理的socket IO事件

和`socket`一样，调用`epoll_create`创建出来的`event_poll`会和当前进程打开的文件列表关联。

{{% figure class="center" src="/img/linux/eventpoll.png" title="eventpoll" alt="img" %}}

`epoll_create` 其源代码位于[fs/eventpoll.c](https://github.com/torvalds/linux/blob/787af64d05cd528aac9ad16752d11bb1c6061bb9/fs/eventpoll.c#L2008)，其中`struct eventpoll `的定义也在这个源文件中。部分代码如下所示：

```c
SYSCALL_DEFINE1(epoll_create1, int, flags)
{
    struct eventpoll *ep = NULL;

    //创建eventpoll 对象
    error = ep_alloc(&ep);
}
```

```c
struct eventpoll {

    //sys_epoll_wait用到的等待队列
    wait_queue_head_t wq;

    //接收就绪的描述符都会放到这里
    struct list_head rdllist;

    //每个epoll对象中都有一颗红黑树
    struct rb_root rbr;

    ......
}
```

- **wq**：等待进程队列双向链表。软中断数据就绪的时候会通过 `wq` 来找到阻塞在 `epoll` 对象上的用户进程
- **rbr**：红黑树。通过这棵树来管理用户进程下添加的`socket`连接
- **rdllist**: 就绪`socket`描述符双向链表。当有的连接就绪的时候，内核会把就绪的连接放到 `rdllist` 链表里。这样进程只需要判断链表就能找出就绪进程，而不用去遍历整棵树

`eventpoll`被创建出来，还需要初始化， 其源代码位于[fs/eventpoll.c](https://github.com/torvalds/linux/blob/787af64d05cd528aac9ad16752d11bb1c6061bb9/fs/eventpoll.c#L936) ，部分代码如下所示：

```c
static int ep_alloc(struct eventpoll **pep)
{
    struct eventpoll *ep;

    //申请 epollevent 内存
    ep = kzalloc(sizeof(*ep), GFP_KERNEL);

    //初始化等待队列头
    init_waitqueue_head(&ep->wq);

    //初始化就绪列表
    INIT_LIST_HEAD(&ep->rdllist);

    //初始化红黑树指针
    ep->rbr = RB_ROOT;

    ......
}
```

## epoll_ctl 添加socket

当`epoll_ctl`注册每个`socket`的时候，会做三件事情

1. 分配一个红黑树节点对象`epitem`
2. 添加等待事件到`socket`的等待队列中，其回调函数是`ep_poll_callback`
3. 将`epitem`插入 `rdllist`红黑树中

通过`epoll_ctl` 添加两个`socket`后，结构如下图
{{% figure class="center" src="/img/linux/epoll_ctl.png" title="epoll_ctl" alt="img" %}}

`epoll_ctl` 添加`socket `的源码位于[fs/eventpoll.c](https://github.com/torvalds/linux/blob/e8b767f5e04097aaedcd6e06e2270f9fe5282696/fs/eventpoll.c#L2183)，部分代码如下所示：

```c
SYSCALL_DEFINE4(epoll_ctl, int, epfd, int, op, int, fd,
        struct epoll_event __user *, event)
{
    struct eventpoll *ep;
    struct file *file, *tfile;

    //根据 epfd 找到 eventpoll 内核对象
    file = fget(epfd);
    ep = file->private_data;

    //根据 socket 句柄号， 找到其 file 内核对象
    tfile = fget(fd);

    switch (op) {
    case EPOLL_CTL_ADD:
        if (!epi) {
            epds.events |= POLLERR | POLLHUP;
            error = ep_insert(ep, &epds, tfile, fd);
        } else
            error = -EEXIST;
        clear_tfile_check_list();
        break;
}
```

`epoll_ctl` 添加操作，会走到这个`EPOLL_CTL_ADD` case，重点看下`ep_insert`函数，其源码位于[fs/eventpoll.c](https://github.com/torvalds/linux/blob/e8b767f5e04097aaedcd6e06e2270f9fe5282696/fs/eventpoll.c#L1444)，部分代码如下所示：

```c
static int ep_insert(struct eventpoll *ep,
                struct epoll_event *event,
                struct file *tfile, int fd)
{
    //3.1 分配并初始化 epitem
    //分配一个epi对象
    struct epitem *epi;
    if (!(epi = kmem_cache_alloc(epi_cache, GFP_KERNEL)))
        return -ENOMEM;

    //对分配的epi进行初始化
    //epi->ffd中存了句柄号和struct file对象地址
    INIT_LIST_HEAD(&epi->pwqlist);
    epi->ep = ep;
    ep_set_ffd(&epi->ffd, tfile, fd);

    //3.2 设置 socket 等待队列
    //定义并初始化 ep_pqueue 对象
    struct ep_pqueue epq;
    epq.epi = epi;
    init_poll_funcptr(&epq.pt, ep_ptable_queue_proc);

    //调用 ep_ptable_queue_proc 注册回调函数
    //实际注入的函数为 ep_poll_callback
    revents = ep_item_poll(epi, &epq.pt);

    ......
    //3.3 将epi插入到 eventpoll 对象中的红黑树中
    ep_rbtree_insert(ep, epi);
    ......
}
```

## epitem 初始化

`epitem` 数据结构 位于[struct epitem ](https://github.com/torvalds/linux/blob/e8b767f5e04097aaedcd6e06e2270f9fe5282696/fs/eventpoll.c#L136)，部分代码如下所示：

```c
struct epitem {

    //红黑树节点
    struct rb_node rbn;

    //socket文件描述符信息
    struct epoll_filefd ffd;

    //所归属的 eventpoll 对象
    struct eventpoll *ep;

    //等待队列
    struct list_head pwqlist;
}
```

对`epitem`部分初始化，包括关联一个`eventpoll`对象，关联`socket` 的`file`字段

{{% figure class="center" src="/img/linux/epitem.png" title="epitem" alt="img" %}}
关联`file`字段代码位于[ep_set_ffd](https://github.com/torvalds/linux/blob/e8b767f5e04097aaedcd6e06e2270f9fe5282696/fs/eventpoll.c#L339)

```c
/* Setup the structure that is used as key for the RB tree */
static inline void ep_set_ffd(struct epoll_filefd *ffd,
			      struct file *file, int fd)
{
	ffd->file = file;
	ffd->fd = fd;
}
```

## 设置socket等待队列

{{% figure class="center" src="/img/linux/ep_poll_callback.png" title="ep_poll_callback" alt="img" %}}

`epitem`初始化后，接下来就设置`socket等待队列`。并把`ep_poll_callback`设置成回调函数

```c
static inline unsigned int ep_item_poll(struct epitem *epi, poll_table *pt)
{
    pt->_key = epi->event.events;

    return epi->ffd.file->f_op->poll(epi->ffd.file, pt) & epi->event.events;
}
```

结合`socket`的结构图中可以看出，`sock->ops->poll `其实指向的是 `tcp_poll`。

```c
unsigned int tcp_poll(struct file *file, struct socket *sock, poll_table *wait)
{
    struct sock *sk = sock->sk;

    sock_poll_wait(file, sk_sleep(sk), wait);
}
```

`sock_poll_wait` 第二个参数是调用`sk_sleep`获得的，其实就是`socket`的等待队列

```c
static inline wait_queue_head_t *sk_sleep(struct sock *sk)
{
    BUILD_BUG_ON(offsetof(struct socket_wq, wait) != 0);
    return &rcu_dereference_raw(sk->sk_wq)->wait;
}

```

进入`sock_poll_wait`

```c
static inline void sock_poll_wait(struct file *filp,
        wait_queue_head_t *wait_address, poll_table *p)
{
    poll_wait(filp, wait_address, p);
}

static inline void poll_wait(struct file * filp, wait_queue_head_t * wait_address, poll_table *p)
{
    if (p && p->_qproc && wait_address)
        p->_qproc(filp, wait_address, p);
}
```

这里的 `qproc` 是个函数指针，它在前面的 `init_poll_funcptr` 调用时被设置成了 `ep_ptable_queue_proc` 函数。

```c
static int ep_insert(...)
{
    ...
    init_poll_funcptr(&epq.pt, ep_ptable_queue_proc);
    ...
}

//file: include/linux/poll.h
static inline void init_poll_funcptr(poll_table *pt,
    poll_queue_proc qproc)
{
    pt->_qproc = qproc;
    pt->_key   = ~0UL; /* all events enabled */
}
```

 **在 ep_ptable_queue_proc 函数中，新建了一个等待队列项，并注册其回调函数为` ep_poll_callback `函数。然后再将这个等待项添加到 `socket` 的等待队列中**。

```c
static void ep_ptable_queue_proc(struct file *file, wait_queue_head_t *whead,
                 poll_table *pt)
{
    struct eppoll_entry *pwq;
    f (epi->nwait >= 0 && (pwq = kmem_cache_alloc(pwq_cache, GFP_KERNEL))) {
                //初始化回调方法
                init_waitqueue_func_entry(&pwq->wait, ep_poll_callback);

                //将ep_poll_callback放入socket的等待队列whead（注意不是epoll的等待队列）
                add_wait_queue(whead, &pwq->wait);

        }
```

在`epoll`之前，由于需要在数据就绪的时候唤醒用户进程，所以等待对象项的 `private`  会设置成当前用户进程描述符 `current`。 而我们今天的 `socket` 是交给 epoll 来管理的，不需要在一个 socket 就绪的时候就唤醒进程，所以这里的 `q->private` 没有用就设置成了 `NULL`

```c
static inline void init_waitqueue_func_entry(
    wait_queue_t *q, wait_queue_func_t func)
{
    q->flags = 0;
    q->private = NULL;

    //ep_poll_callback 注册到 wait_queue_t对象上
    //有数据到达的时候调用 q->func
    q->func = func;
}
```

可以看出，软中断将数据拷贝到`socket` 的接收队列后，会通过`ep_poll_callback`回调函数通知`epoll`对象

## 插入红黑树

分配完`epitem` 对象后，会把它插入到红黑树中

{{% figure class="center" src="/img/linux/rbr.png" title="rbr" alt="img" %}}

为什么要红黑树呢 ？



## epoll_wait等待接收

当`epoll_wait`被调用的时候，会判断`eventpoll->rdllist` 链表里有没有数据，如果有数据就返回，没有数据就创建一个等待队列的元素，然后添加到`eventpoll`等待队列上。其过程大致如下

{{% figure class="center" src="/img/linux/epoll_wait.png" title="epoll_wait" alt="img" %}}
**值得注意的是`epoll_ctl` 添加 `socket` 时也创建了等待队列项。不同的是这里的等待队列项是挂在 `epoll` 对象上的，而前者是挂在 socket 对象上的。**

其源代码如下：

```c
//file: fs/eventpoll.c
SYSCALL_DEFINE4(epoll_wait, int, epfd, struct epoll_event __user *, events,
        int, maxevents, int, timeout)
{
    ...
    error = ep_poll(ep, events, maxevents, timeout);
}

static int ep_poll(struct eventpoll *ep, struct epoll_event __user *events,
             int maxevents, long timeout)
{
    wait_queue_t wait;
    ......

fetch_events:
    //4.1 判断就绪队列上有没有事件就绪
    if (!ep_events_available(ep)) {

        //4.2 定义等待事件并关联当前进程
        init_waitqueue_entry(&wait, current);

        //4.3 把新 waitqueue 添加到 epoll->wq 链表里
        __add_wait_queue_exclusive(&ep->wq, &wait);

        for (;;) {
            ...
            //4.4 让出CPU 主动进入睡眠状态
            if (!schedule_hrtimeout_range(to, slack, HRTIMER_MODE_ABS))
                timed_out = 1;
            ...
}
```

判断就绪队列有没有就绪的`socket`，代码如下所示

```c
static inline int ep_events_available(struct eventpoll *ep)
{
    return !list_empty(&ep->rdllist) || ep->ovflist != EP_UNACTIVE_PTR;
}
```

阻塞当前进程，将创建出来的待队列对象与当前进程关联，设置唤醒回调函数

```c
static inline void init_waitqueue_entry(wait_queue_t *q, struct task_struct *p)
{
    q->flags = 0;
    q->private = p;
    q->func = default_wake_function;
}
```

添加到等待队列

```c
static inline void __add_wait_queue_exclusive(wait_queue_head_t *q,
                                wait_queue_t *wait)
{
    wait->flags |= WQ_FLAG_EXCLUSIVE;
    __add_wait_queue(q, wait);
}
```

通过 `set_current_state` 把当前进程设置为可打断。 调用 `schedule_hrtimeout_range` 让出 CPU，主动进入睡眠状态

```c
int __sched schedule_hrtimeout_range(ktime_t *expires,
    unsigned long delta, const enum hrtimer_mode mode)
{
    return schedule_hrtimeout_range_clock(
            expires, delta, mode, CLOCK_MONOTONIC);
}

int __sched schedule_hrtimeout_range_clock(...)
{
    schedule();
    ...
}
```

在 schedule 中选择下一个进程调度

```c
static void __sched __schedule(void)
{
    next = pick_next_task(rq);
    ...
    context_switch(rq, prev, next);
}
```

# 数据从网卡到进程

在前面 epoll_ctl 执行的时候，会给每一个 `socket `添加回调函数（等待队列对象）。 在 `epoll_wait` 运行完的时候，会给`event_poll`添加等待队列对象。数据开始接收之前，先把这些队列项的内容再稍微总结一下。
{{% figure class="center" src="/img/linux/epoll.png" title="epoll" alt="img" %}}

- `socket->sock->sk_data_ready `设置的就绪处理函数是 `sock_def_readable`
- 在`socket`等待队列中，回调函数是`ep_poll_callback`,`private`字段没有作用，置为`NULL`
- 在 `eventpoll `的等待队列项中，回调函数是 `default_wake_function`。其 private 指向的是等待该事件的用户进程。

## 接收数据到任务队列

软中断来的时候会一步一步往协议上层分解。网络帧->tcp数据包，这里重点分析` tcp `协议栈的处理入口函数 `tcp_v4_rcv `

```c
// file: net/ipv4/tcp_ipv4.c
int tcp_v4_rcv(struct sk_buff *skb)
{
    ......
    th = tcp_hdr(skb); //获取tcp header
    iph = ip_hdr(skb); //获取ip header

    //根据数据包 header 中的 ip、端口信息查找到对应的socket
    sk = __inet_lookup_skb(&tcp_hashinfo, skb, th->source, th->dest);
    ......

    //socket 未被用户锁定
    if (!sock_owned_by_user(sk)) {
        {
            if (!tcp_prequeue(sk, skb))
                ret = tcp_v4_do_rcv(sk, skb);
        }
    }
}
```

在 `tcp_v4_rcv` 中首先根据收到的网络包的 `header` 里的 `source` 和 `dest `信息来在本机上查询对应的 `socket`。找到以后，就进入接收的主体函数 `tcp_v4_do_rcv `。

```c
//file: net/ipv4/tcp_ipv4.c
int tcp_v4_do_rcv(struct sock *sk, struct sk_buff *skb)
{
    if (sk->sk_state == TCP_ESTABLISHED) {

        //执行连接状态下的数据处理
        if (tcp_rcv_established(sk, skb, tcp_hdr(skb), skb->len)) {
            rsk = sk;
            goto reset;
        }
        return 0;
    }

    //其它非 ESTABLISH 状态的数据包处理
    ......
}
```

如果处理的是 `ESTABLISH` 状态下的包，这样就又进入 `tcp_rcv_established `函数中

```c
//file: net/ipv4/tcp_input.c
int tcp_rcv_established(struct sock *sk, struct sk_buff *skb,
            const struct tcphdr *th, unsigned int len)
{
    ......

    //接收数据到队列中
    eaten = tcp_queue_rcv(sk, skb, tcp_header_len, &fragstolen);                         
    //数据 ready，唤醒 socket 上阻塞掉的进程
    sk->sk_data_ready(sk, 0);
```

在 `tcp_rcv_established` 中通过调用 `tcp_queue_rcv `函数中完成了将接收数据放到 socket 的接收队列上。

{{% figure class="center" src="/img/linux/tcp_queue_rcv.png" title="tcp_queue_rcv" alt="img" %}}

如下源码所示：

```c
//file: net/ipv4/tcp_input.c
static int __must_check tcp_queue_rcv(struct sock *sk, struct sk_buff *skb, int hdrlen,
            bool *fragstolen)
{
    //把接收到的数据放到 socket 的接收队列的尾部
    if (!eaten) {
        __skb_queue_tail(&sk->sk_receive_queue, skb);
        skb_set_owner_r(skb, sk);
    }
    return eaten;
}
```

## 查找就绪回调函数

调用 `tcp_queue_rcv `接收完成之后，接着再调用 `sk_data_ready` 。 在 `accept` 函数创建 socket 流程里提到的 `sock_init_data` 函数，在这个函数里已经把 sk_data_ready 设置成 sock_def_readable 函数了。它是默认的数据就绪处理函数。当 `socket` 上数据就绪时候，内核将以` sock_def_readable` 这个函数为入口，找到 `epoll_ctl `添加 `socket` 时在其上设置的回调函数`ep_poll_callback`。

{{% figure class="center" src="/img/linux/poll_callback.png" title="poll_callback" alt="img" %}}

具体代码如下所示：

```c
//file: net/core/sock.c
static void sock_def_readable(struct sock *sk, int len)
{
    struct socket_wq *wq;

    rcu_read_lock();
    wq = rcu_dereference(sk->sk_wq);

    //这个名字起的不好，并不是有阻塞的进程，
    //而是判断等待队列不为空
    if (wq_has_sleeper(wq))
        //执行等待队列项上的回调函数
        wake_up_interruptible_sync_poll(&wq->wait, POLLIN | POLLPRI |
                        POLLRDNORM | POLLRDBAND);
    sk_wake_async(sk, SOCK_WAKE_WAITD, POLL_IN);
    rcu_read_unlock();
}
```

重点看下`wake_up_interruptible_sync_poll`

```c
//file: include/linux/wait.h
#define wake_up_interruptible_sync_poll(x, m)       \
    __wake_up_sync_key((x), TASK_INTERRUPTIBLE, 1, (void *) (m))


//file: kernel/sched/core.c
void __wake_up_sync_key(wait_queue_head_t *q, unsigned int mode,
            int nr_exclusive, void *key)
{
    ...
    __wake_up_common(q, mode, nr_exclusive, wake_flags, key);
}
```

接着进入 `__wake_up_common`

```c
static void __wake_up_common(wait_queue_head_t *q, unsigned int mode,
            int nr_exclusive, int wake_flags, void *key)
{
    wait_queue_t *curr, *next;

    list_for_each_entry_safe(curr, next, &q->task_list, task_list) {
        unsigned flags = curr->flags;

        if (curr->func(curr, mode, wake_flags, key) &&
                (flags & WQ_FLAG_EXCLUSIVE) && !--nr_exclusive)
            break;
    }
}
```

在 `__wake_up_common `中，选出等待队列里注册某个元素 `curr`， 回调其` curr->func`。 在 `ep_insert `调用的时候，把这个 `func `设置成 `ep_poll_callback` 了。

## 执行ep_poll_callback

```c
//file: fs/eventpoll.c
static int ep_poll_callback(wait_queue_t *wait, unsigned mode, int sync, void *key)
{
    //获取 wait 对应的 epitem
    struct epitem *epi = ep_item_from_wait(wait);

    //获取 epitem 对应的 eventpoll 结构体
    struct eventpoll *ep = epi->ep;

    //1. 将当前epitem 添加到 eventpoll 的就绪队列中
    list_add_tail(&epi->rdllink, &ep->rdllist);

    //2. 查看 eventpoll 的等待队列上是否有在等待
    if (waitqueue_active(&ep->wq))
        wake_up_locked(&ep->wq);
```

在 `ep_poll_callback`根据等待任务队列项上的额外的 `base` 指针可以找到` epitem`， 进而也可以找到 `eventpoll` 对象。首先它做的第一件事就是**把自己的 epitem 添加到 epoll 的就绪队列中**。接着它又会查看 `eventpoll `对象上的等待队列里是否有等待项（`epoll_wait `执行的时候会设置）如果没执行软中断的事情就做完了。如果有等待项，那就查找到等待项里设置的回调函数`default_wake_function`。

## 执行epoll就绪通知

在 `default_wake_function` 中找到等待队列项里的进程描述符，然后唤醒之

代码如下所示：

```c
//file:kernel/sched/core.c
int default_wake_function(wait_queue_t *curr, unsigned mode, int wake_flags,
                void *key)
{
    return try_to_wake_up(curr->private, mode, wake_flags);
}
```

等待队列项` curr->private` 指针是在 epoll 对象上等待而被阻塞掉的进程。将 `epoll_wait `进程推入可运行队列，等待内核重新调度进程。然后 `epoll_wait `对应的这个进程重新运行后，就从` schedule` 恢复当进程醒来后，继续从 `epoll_wait` 时暂停的代码继续执行。把` rdlist` 中就绪的事件返回给用户进程

```c
//file: fs/eventpoll.c
static int ep_poll(struct eventpoll *ep, struct epoll_event __user *events,
             int maxevents, long timeout)
{

    ......
    __remove_wait_queue(&ep->wq, &wait);

    set_current_state(TASK_RUNNING);
    }
check_events:
    //返回就绪事件给用户进程
    ep_send_events(ep, events, maxevents))
}
```

# 总结

{{% figure class="center" src="/img/linux/event_poll_all.png" title="event_poll_all" alt="img" %}}

所有的回调：

- `sock_def_readable`： `sock `对象初始化时设置的
- `ep_poll_callback` : `epoll_ctl` 时添加到 `socket` 上
- `default_wake_function`: `epoll_wait` 是设置到 `epoll` 上的

为什么`epoll` 会如此高效呢，我想大概是因为，`socket`接收到软中断后不用马上唤醒进程，由`epoll`接管。当`epoll`唤醒进程后，只需要查看`rdllist` 就绪队列就知道哪些`socket`有数据，多路复用，复用的是进程。因为频繁的进程切换是效率低下的重要原因之一。

