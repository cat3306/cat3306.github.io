# redis 常见操作


<!--more-->

### 批量删除KEY
``` lua 
EVAL "local cursor='0'; repeat local result=redis.call('SCAN',cursor,'MATCH',ARGV[1],'COUNT',1000); cursor=result[1]; for _,key in ipairs(result[2]) do redis.call('DEL',key); end until cursor=='0'; return true;" 0 "user:*"
```

### COUNT(前缀KEY)

``` lua
 EVAL "local cursor = '0'; local count = 0; repeat local result = redis.call('SCAN', cursor, 'MATCH', ARGV[1], 'COUNT', 1000); cursor = result[1]; count = count + #result[2]; until cursor == '0'; return count" 0 "ws:*"
```

### 统计zset 个数
``` lua
eval 'local total = 0;for i = 0, 9 do local key = "xyz:onlineuser:queue:" .. i total = total + redis.call("ZCARD", key) end return total' 0
```
