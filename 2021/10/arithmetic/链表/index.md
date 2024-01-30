# 链表


<!--more-->

## 链表原地反转

```go
type Node struct {
	Ele  int
	Next *Node
}

func PrintNode(head *Node) {
	p := head
	for p != nil {
		if p.Next == nil {
			fmt.Printf("%d", p.Ele)
		} else {
			fmt.Printf("%d,", p.Ele)
		}
		p = p.Next
	}
}

func InitLinkList() *Node {
	head := &Node{
		Ele: 1,
		Next: &Node{
			Ele: 2,
			Next: &Node{
				Ele: 3,
				Next: &Node{
					Ele: 4,
					Next: &Node{
						Ele: 5,
						Next: &Node{
							Ele: 6,
							Next: &Node{
								Ele: 7,
								Next: &Node{
									Ele: 8,
									Next: &Node{
										Ele:  9,
										Next: nil,
									},
								},
							},
						},
					},
				},
			},
		},
	}
	return head
}
//反转链表
func Reverse(head *Node) *Node {
	if head == nil || head.Next == nil {
		return head
	}
	var newHead *Node
	p := head
	for p != nil {
		tmp := p.Next
		p.Next = newHead
		newHead = p
		p = tmp
	}
	return newHead
}

func TestB(t *testing.T) {
	head := InitLinkList()
	PrintNode(head)
	fmt.Println()
	PrintNode(Reverse(head))
}
1,2,3,4,5,6,7,8,9
9,8,7,6,5,4,3,2,1--- PASS: TestB (0.00s)
PASS
```


