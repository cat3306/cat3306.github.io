# 算法题


<!--more-->

# 删除链表的倒数第`n`个节点

示例1：

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

示例2:

```
输入：head = [1], n = 1
输出：[]
```

示例3:

```
输入：head = [1,2], n = 1
输出：[1]
```

提示:

- 链表中结点的数目为 `sz`
- `1 <= sz <= 30`
- `0 <= Node.val <= 100`
- `1 <= n <= sz`

````go

func removeNthFromEnd(head *ListNode, n int) *ListNode {
   	p := head
	q := head
	i := 0
	for q != nil {
		if i > n {
			p = p.Next
		}
		i++
		q = q.Next
	}

	if n-i >= 0 {
		head = head.Next
	} else {
		tmp := p.Next
		if tmp != nil {
			p.Next = tmp.Next
		}
	}
	return head
}
````

# 将下面的字符串转成map

````go

//下面若干个空格
//frame=  425 fps= 71 q=-1.0 Lsize=    5158kB time=00:00:17.11 bitrate=2468.7kbits/s dup=1 drop=0 speed=2.84x
func Task00(s string) map[string]string {
	m := make(map[string]string)

	var start, end int
	for i := 0; i < len(s); i++ {
		if s[i] == ' ' {
			continue
		}

		var key, value string
		for end = i; end < len(s); end++ {
			if s[end] == '=' {
				key = s[start:end]
				break
			}
		}
		start = end + 1
		for ; start < len(s); start++ {
			if s[start] != ' ' {
				end = start
				break
			}
		}

		for ; end < len(s); end++ {
			if s[end] == ' ' {
				value = s[start:end]
				break
			}
			if end == len(s)-1 {
				value = s[start:]
			}
		}
		i = end
		start = end + 1
		m[key] = value
	}
	return m
}


func Task10(s string) map[string]string {
	list := strings.Fields(s)
	m := make(map[string]string)
	for i := 0; i < len(list); i++ {
		if list[i] == "" {
			continue
		}
		v := list[i]
		if v[len(list[i])-1] == '=' {
			if i+1 < len(list) {
				m[v[:len(v)-1]] = list[i+1]
				i++
			}
		} else {
			strs := strings.Split(v, "=")
			if len(strs) > 1 {
				m[strs[0]] = strs[1]
			}
		}
	}
	return m
}
````


