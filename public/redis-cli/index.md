# redis cli


<!--more-->

# KEY

## DEL

删除`key`

DEL key [key ...]

示例

```
127.0.0.1:6379> SET name joker
OK
127.0.0.1:6379> GET name
joker
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> DEL name
1
127.0.0.1:6379> GET name

127.0.0.1:6379>
```

## DUMP



## EXISTS

判断一个key是否存在

时间复杂度：O(1)

返回值：

若 `key` 存在，返回 `1` ，否则返回 `0` 。

```
127.0.0.1:6379> set name haha
OK
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> exists name
1
127.0.0.1:6379>
127.0.0.1:6379> del name
1
127.0.0.1:6379> exists name
0
127.0.0.1:6379>
```

## EXPIRE

**EXPIRE key seconds**

设置`key`的过期时间

以下几个操作不会改变`key`的过期时间

- 对一个`key`进行`INCR`,`LPUSH`,`HSET`等
- `RENAME`，重命名某个`key`

{{< admonition >}}
`SET ` 同一个 `key`时会删除过期时间

```
127.0.0.1:6379> set name joker
OK
127.0.0.1:6379> expire name 10000
1
127.0.0.1:6379> ttl name
9997
127.0.0.1:6379> set name top
OK
127.0.0.1:6379> ttl name
-1
127.0.0.1:6379>
```



{{< /admonition >}}

**另外如果想移除某个`key`的过期时间，可以用 `PERSIST`命令**

时间复杂度

O(1)

```
127.0.0.1:6379> get name
haha
127.0.0.1:6379> expire name 1000
1
127.0.0.1:6379> ttl name
993
127.0.0.1:6379> expire name 1000
1
127.0.0.1:6379> ttl name
998
127.0.0.1:6379> ttl name
976
127.0.0.1:6379> PERSIST name
1
127.0.0.1:6379> ttl name
-1
127.0.0.1:6379>
```

## EXPIREAT

**EXPIREAT key timestamp**

从字面意思可知，设置过期日期

时间复杂度：

O(1)

```
// 1658997850 -->2022-07-28 16:44:10
127.0.0.1:6379> expireat name 1658997850
1
127.0.0.1:6379> ttl name
7183
127.0.0.1:6379>
```

## KEYS

**KEYS pattern**

`KEYS *` 匹配数据库中所有 `key` 

`KEYS h?llo` 匹配 `hello` ， `hallo` 和 `hxllo` 等。

`KEYS h*llo` 匹配 `hllo` 和 `heeeeello` 等。

`KEYS h[ae]llo` 匹配 `hello` 和 `hallo` ，但不匹配 `hillo` 。

{{< admonition warning >}}
虽然`keys` 的速度非常快，但是在存在大量`key`的数据库中尽量少用。原因之一是`redis`是单线程来处理逻辑，如果`key`的数量非常多就会很耗时，必定会影响其他的请求，此操作是O(n)的复杂度
{{< /admonition >}}

```
127.0.0.1:6379> MSET name1 joker name2 bob name3 haha
OK
127.0.0.1:6379> keys *am*
name2
name1
name3
```

## MIGRATE

**MIGRATE host port key  destination-db timeout [COPY] [REPLACE] [AUTH password]**

这个命令是将指定的 `key` 原子性地从当前实例传送到目标实例的指定数据库上，一旦传送成功， `key` 保证会出现在目标实例上，而当前实例上的 `key` 会被删除。

{{< admonition  >}}
下面的例子中`ip`和密码已抹去真实性
{{< /admonition >}}

```
127.0.0.1:6379> get joker
hahaha
127.0.0.1:6379> MIGRATE 111.36.72.254 6375 AUTH 1234 joker 0 1000 REPLACE
ERR syntax error

127.0.0.1:6379> MIGRATE 111.36.72.254 6375  joker 0 1000 REPLACE
ERR Target instance replied with error: NOAUTH Authentication required

127.0.0.1:6379> MIGRATE 111.36.72.254 6375  joker 0 1000 REPLACE AUTH 1234
OK

111.36.72.254:6375> get joker
"hahaha"

127.0.0.1:6379> get joker
//本地已删除
```

该命令是原子操作，执行时会阻塞两个实例，直到以下任意结果发生：迁移成功，迁移失败，网络超时等。

命令的内部实现是这样的：在传输目标实例之前，会执行`DUMP`操作将本地指定的`key`序列化，然后在传送给目标实例，目标实例接收后执行`RESTORE`将数据反序列化，并添加到数据库中。如果可选参数是`REPLACE`会删除本地实例的`key`,如果是`COPY`则不会删除，但是如果目标实例有相同的`key`会发生错误`ERR Target instance replied with error: BUSYKEY Target key name already exists.`

```
127.0.0.1:6379> MIGRATE 111.36.72.254 6375  joker 0 1000 COPY AUTH 1234
OK
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> get joker
hahahhah

111.36.72.254:6375> get joker
"hahahhah"


127.0.0.1:6379> MIGRATE 111.36.72.254 6375  joker 0 1000 COPY AUTH 1234
ERR Target instance replied with error: BUSYKEY Target key name already exists.
```

{{< admonition  >}}
MIGRATE 命令是原子操作，即使失败也不会丢`key`

{{< /admonition >}}

## MOVE

**MOVE key db**

将指定`key`的移动到指定的数据库中

```
127.0.0.1:6379> select 0
OK
127.0.0.1:6379> get joker
hahahhah
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> move joker 1
1
127.0.0.1:6379> exists joker
0
127.0.0.1:6379>
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> get joker
hahahhah
127.0.0.1:6379[1]>
```

但是试图移动的`key`和目标数据库存在`key`相同，目标数据库的`key`对应的`value`并不会被覆盖

```
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379>
127.0.0.1:6379> get joker

127.0.0.1:6379> set joker jjjjj
OK
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> move joker 1
0
127.0.0.1:6379>
127.0.0.1:6379> get joker
jjjjj
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> get joker
hahahhah
127.0.0.1:6379[1]>
```

## OBJECT

**OBJECT subcommand [arguments [arguments]]**

- `OBJECT REFCOUNT key`  返回给定 `key` 引用所储存的值的次数
- `OBJECT ENCODING key`  储存的值所使用的内部表示
- `OBJECT IDLETIME key`  自储存以来的空转时间

```
127.0.0.1:6379> set name haha
OK
127.0.0.1:6379> OBJECT REFCOUNT name
1
127.0.0.1:6379>
127.0.0.1:6379> OBJECT ENCODING name
embstr
127.0.0.1:6379>
127.0.0.1:6379> OBJECT IDLETIME name
36
127.0.0.1:6379> GET name
haha
127.0.0.1:6379> OBJECT IDLETIME name
2
127.0.0.1:6379>
```

## PERSIST

**PERSIST key**

移除给定 `key` 的生存时间

```
127.0.0.1:6379> set name haha
OK
127.0.0.1:6379>
127.0.0.1:6379> expire name 10000
1
127.0.0.1:6379> ttl name
9997
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> PERSIST name
1
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> ttl name
-1
127.0.0.1:6379>
```

## PEXPIREAT

**PEXPIREAT key milliseconds-timestamp**

以毫秒为单位设置过期时间

```
127.0.0.1:6379> set name 123
OK
127.0.0.1:6379>
//1659166810000 -->2022-07-30 15:40:10
127.0.0.1:6379> PEXPIREAT name 1659166810000
1
127.0.0.1:6379> ttl name
3547
127.0.0.1:6379>
```

## PTTL

**PTTL key**

以毫秒为单位返回 `key` 的剩余生存时间

```
127.0.0.1:6379> set name haha
OK
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> expire name 10
1
127.0.0.1:6379> pttl name
4075
127.0.0.1:6379>
```

## RANDOMKEY

从当前数据库中随机返回一个`key`

```
127.0.0.1:6379> MSET name haha name1 joker name2 jhsj name3 sasdad
OK
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> RANDOMKEY
name3
127.0.0.1:6379> RANDOMKEY
name2
127.0.0.1:6379> RANDOMKEY
name1
127.0.0.1:6379> RANDOMKEY
name3
```

## RENAME

**RENAME key newkey**

将 `key` 改名为 `newkey` 

可以看见`rename`不会删除原有过期时间

```
127.0.0.1:6379> set name 123
OK
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> expire name 100
1
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> ttl name
92
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> rename name rename
OK
127.0.0.1:6379> ttl rename
77
```

如果已经存在新的`key`，则会覆盖

```
127.0.0.1:6379> set name haha
OK
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> set name1 joker
OK
127.0.0.1:6379>
127.0.0.1:6379> rename name name1
OK
127.0.0.1:6379>
127.0.0.1:6379> get name1
haha
127.0.0.1:6379> get name

127.0.0.1:6379>
```



## RENAMENX

**RENAMENX key newkey**

当且仅当 `newkey` 不存在时，将 `key` 改名为 `newkey` 

```
127.0.0.1:6379> set name haha
OK
127.0.0.1:6379> set name1 joker
OK
127.0.0.1:6379> RENAMENX name name1
0
127.0.0.1:6379> MGET name name1
haha
joker
127.0.0.1:6379>
```

## RESTORE

**RESTORE key ttl serialized-value**

## SORT

**SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination]**





## TTL

以秒为单位，返回给定 `key` 的剩余生存时间(TTL, time to live)。

返回值

- 当 `key` 不存在时，返回 `-2` 。
- 当 `key` 存在但没有设置剩余生存时间时，返回 `-1` 。
- 否则，以秒为单位，返回 `key` 的剩余生存时间。

```
127.0.0.1:6379> set name haha
OK
127.0.0.1:6379> ttl name
-1
127.0.0.1:6379> ttl nam
-2
127.0.0.1:6379> expire name 100
1
127.0.0.1:6379> ttl name
96
127.0.0.1:6379>
```

## TYPE

**TYPE key**

返回`key`的类型

- `none `    (key不存在)
- `string` (字符串)
- `list`     (列表)
- `set`       (集合)
- `zset`     (有序集)
- `hash`     (哈希表)

```
```



## SCAN

**SCAN cursor [MATCH pattern] [COUNT count]**

`SCAN`是基于游标的迭代器。执行命令后都会返回一个新的游标，第二次`SCAN`根据新的游标进行迭代

`SCAN`从0开始，如果迭代已结束会返回0，如下所示

```
127.0.0.1:6379> SCAN 0
1) "0"
2) 1) "name1"
```

### MATCH 选项

过滤某种`key`

```
127.0.0.1:6375[5]> SCAN 294913  MATCH first_pay*
1) "16385"
2) 1) "first_pay_0455024283648"
   2) "first_pay_4210839597056"
   3) "first_pay_3615148937216"
```

### COUNT 选项

指定返回的个数

```
127.0.0.1:6375[5]> SCAN 720896  COUNT 12
1) "360448"
2)  1) "mobile_5347552301056"
    2) "first_pay_7677150433280"
    3) "firstPay:1455201652736"
    4) "first_pay_7533806170112"
    5) "firstPay:2564312084480"
    6) "first_pay_8645317152768"
    7) "first_pay_9162111070208"
    8) "first_pay_8233627316224"
    9) "first_pay_1507511316480"
   10) "first_pay_9747146321920"
```

# string

## APPEND

**APPEND key value**

- 如果`key`存在并且是字符串，`APPEND`将`value`追加到原来的`value`中
- 如果`key`不存在，就直接设置`value`,和 `SET key value` 一样

```
127.0.0.1:6379> EXISTS name
(integer) 0
127.0.0.1:6379> APPEND name haha
(integer) 4
127.0.0.1:6379> get name
"haha"
127.0.0.1:6379> APPEND name 'love you'
(integer) 12
127.0.0.1:6379> get nmae
(nil)
127.0.0.1:6379> get name
"hahalove you"
```

返回的是追加后的字符个数

## BITCOUNT

计算字符串中，被设置为1的比特位数量。可以指定`start` 或 `end` 参数

例如

```
127.0.0.1:6379> SETBIT haha 0 1
(integer) 0
127.0.0.1:6379> SETBIT haha 1 1
(integer) 0
127.0.0.1:6379> SETBIT haha 2 1
(integer) 0
127.0.0.1:6379> SETBIT haha 3 0
(integer) 0
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> BITCOUNT haha 0 -1
(integer) 3
127.0.0.1:6379>
```

`BITCOUNT haha 0 -1` -1表示最后一个，也可以不指定`start` `end`，表示全部

如图

{{% figure  src="/img/redis/1.png" alt="img" alt="img" %}}

# HASH

## HSET

**HSET key field value**

**返回值：**

如果 `field` 是哈希表中的一个新建域，并且值设置成功，返回 `1` 。

如果哈希表中域 `field` 已经存在且旧值已被新值覆盖，返回 `0` 。

````
127.0.0.1:6379> HSET website google "www.g.cn"
(integer) 1
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 0
127.0.0.1:6379>
````






