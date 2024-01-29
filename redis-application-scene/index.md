# redis 应用场景


# 分布式锁

```bash
set joker 1 nx ex 10
```

```lua
var delScript = `
if redis.call("get", KEYS[1]) == ARGV[1] then
	return redis.call("del", KEYS[1])
else
	return 0
end`
```


