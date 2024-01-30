# mysql application scene


## 统计重复字段的重复次数

```sql
	SELECT game_id,count(1)as c FROM android_game_detail  GROUP BY game_id HAVING c>1
```


