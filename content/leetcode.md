# 高频Leetcode算法题
> 此开源图书由[thaiq](https://github.com/ithaiq)原创，创作不易转载请注明出处

* [两数之和](https://leetcode-cn.com/problems/two-sum/)
> 分析：空间换时间借助一个map。key存具体的数，val存索引

``` go
func twoSum(nums []int, target int) []int {
    var result []int
    tmp:=make(map[int]int)
    for i:=0;i<len(nums);i++{
        if v,ok:=tmp[target-nums[i]];ok{
            result = append(result,v,i)
            return result
        }else{
            tmp[nums[i]]=i
        }
    }
    return result
}
```

* [两数相加](https://leetcode-cn.com/problems/add-two-numbers/)
> 分析:创建哨兵节点和一个进位值，每次数值相加再加上进位值再计算，最后再判断进位进行处理

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    dummy:=&ListNode{}
    current:=dummy
    carry:=0
    for l1!=nil || l2!=nil{
        var sum int
        if l1!=nil{
            sum+=l1.Val
            l1=l1.Next
        }
        if l2!=nil{
            sum+=l2.Val
            l2=l2.Next
        }
        sum = sum+carry
        current.Next = &ListNode{Val:sum%10}
        carry = sum/10
        current = current.Next
    }
    if carry!=0{
        current.Next = &ListNode{Val:carry}
    }
    return dummy.Next 
}
```

