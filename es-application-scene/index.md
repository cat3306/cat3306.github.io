# es 的应用场景


## 条件查询

 `"must": []`

`wildcard`:模糊查询

```json
POST user_action_record/_search
{
  "size": 1000,
  "from": 0, 
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "create_date": {
              "gte": "2022-07-30 17:23:55",
              "lte": "2022-07-31 17:23:55"
            }
          }
        },
        {
          "match": {
            "public.appver": "1.5.1"
          }
        },
        {
          "wildcard": {
            "context.ex1": "*没有获取*"
          }
        }
      ]
    }
  }
}
```

对应的`sql`

```sql
SELECT
	* 
FROM
	user_action_record 
WHERE
	create_date >= '2022-07-30 17:23:55' 
	AND create_date <= '2022-07-31 17:23:55' 
	AND appver = '1.5.1' 
	AND ex1 LIKE '%没有获取%' 
	LIMIT 0,
	1000
```

## IN 查询

对多个uid查询

```json
{
  "size": 20000,
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "create_date": {
              "gte": "2022-08-01 00:00:00",
              "lte": "2022-08-01 23:59:59"
            }
          }
        },
        {
          "match": {
            "context.action": "boost_click"
          }
        },
        {
          "terms": {
            "public.uid": [
              "8686520496128",
              "3151070081024",
              "7021632212992",
              "4221968166912",
              "9486033596416",
              "5187451686912",
              "7987553398784"
            ]
          }
        }
      ]
    }
  },
  "_source": "context.ex1"
}
```

## 根据时间分组查询

并且根据分组的时间倒序排列

```sql
{
  "size": 0,
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "create_date": {
              "gte": "2022-08-03 00:00:00",
              "lte": "2022-08-04 23:59:59"
            }
          }
        },
        {
          "match": {
            "context.action": "app_new_install"
          }
        },
        {
          "match": {
            "type": "app_info"
          }
        }
      ]
    }
  },
  "aggs": {
    "group": {
      "date_histogram": {
        "field": "create_date",
        "calendar_interval": "day",
        "format": "yyyy-MM-dd",
        "order": {
          "_key": "desc"
        }
      }
    }
  }
}
```

## `mapping`

删除

```
DELETE user_action_record_2022-10-18
```

获取`mapping`信息

```
GET user_action_record_2022-10-18/_mapping
```






