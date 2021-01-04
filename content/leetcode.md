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

* [无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)
>分析：维护一个滑动窗口，由go map+slice相当于定义有序set，如果窗口值重复则从窗口起始开始删除直至不重复为止。

```go
func lengthOfLongestSubstring(s string) int {
	if len(s) == 0 {
		return 0
	}
	var maxLen int
	var windowArr []byte
	windowMap := make(map[byte]bool)
	for i := 0; i < len(s); i++ {
		if _, ok := windowMap[s[i]]; !ok {
			windowMap[s[i]] = true
			windowArr = append(windowArr, s[i])
			maxLen = max(maxLen, len(windowMap))
		} else {
			for exist := true; exist; _, exist = windowMap[s[i]] {
				val := windowArr[0]
				delete(windowMap, val)
				windowArr = windowArr[1:]
			}
			windowMap[s[i]] = true
			windowArr = append(windowArr, s[i])
		}
	}
	return maxLen
}

func max(x, y int) int {
	if x < y {
		return y
	}
	return x
}
```