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
* [最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)
>分析：回文子串有两种情况，aba和abba，所以字符串一个字符应该遍历两遍

```go
func longestPalindrome(s string) string {
	if len(s) < 2 {
		return s
	}
	start, maxLen := 0, 1
	getLongestPalindrome := func(left, right int) {
		for left >= 0 && right < len(s) && s[left] == s[right] {
			val := right - left + 1
			if val > maxLen {
				maxLen = val
				start = left
			}
			left--
			right++
		}
	}
	for i := 0; i < len(s); i++ {
		getLongestPalindrome(i-1, i+1)
		getLongestPalindrome(i, i+1)
	}
	return s[start : start+maxLen]
}
```

* 15. [三数之和](https://leetcode-cn.com/problems/3sum/)
>分析：先将数字排序，然后遍历，每次遍历定义start和end节点 如果大了end--,小了start++，注意重复情况

```go
func threeSum(nums []int) [][]int {
	var result [][]int
	sortNum := func(nums []int) {
		for i := 0; i < len(nums)-1; i++ {
			for j := i + 1; j < len(nums); j++ {
				if nums[i] > nums[j] {
					nums[i], nums[j] = nums[j], nums[i]
				}
			}
		}
	}
	sortNum(nums)
	for i := 0; i < len(nums)-2; i++ {
		if i == 0 || nums[i] != nums[i-1] {
			start, end := i+1, len(nums)-1
			for start < end {
				val := nums[i] + nums[start] + nums[end]
				if val == 0 {
					result = append(result, []int{nums[i], nums[start], nums[end]})
					start++
					end--
					for start < end && nums[start] == nums[start-1] {
						start++
					}
					for start < end && nums[end] == nums[end+1] {
						end--
					}
				} else if val < 0 {
					start++
				} else {
					end--
				}
			}
		}
	}
	return result
}
```
* 19. [删除链表的倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)
>分析：创建dummy节点处理边界情况

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func removeNthFromEnd(head *ListNode, n int) *ListNode {
	dummy := &ListNode{}
	dummy.Next = head
	start, end := dummy, dummy
	for i := 0; i < n; i++ {
		end = end.Next
	}
	for end.Next != nil {
		start = start.Next
		end = end.Next
	}
	start.Next = start.Next.Next
	return dummy.Next
}
```

* 20. [有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)
>分析：go使用slice切片模拟stack栈，这里要注意的是string字符串遍历得到的是byte

```go
func isValid(s string) bool {
	tmp := map[byte]byte{
		'{': '}',
		'[': ']',
		'(': ')',
	}
	var stack []byte
	for i := 0; i < len(s); i++ {
		if value, ok := tmp[s[i]]; ok {
			stack = append(stack, value)
		} else {
            if len(stack) == 0 {
				return false
			}
			endValue := stack[len(stack)-1]
			if s[i] != endValue {
				return false
			}
			stack = stack[:len(stack)-1]
		}
	}
	if len(stack) != 0 {
		return false
	}
	return true
}
```

* 21. [合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)
>分析：使用dummy节点返回根节点

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
	node := &ListNode{}
	dummy := node
	for l1 != nil && l2 != nil {
		if l1.Val > l2.Val {
			node.Next = l2
			l2 = l2.Next
		} else {
			node.Next = l1
			l1 = l1.Next
		}
		node = node.Next
	}
	if l1 != nil {
		node.Next = l1
	}
	if l2 != nil {
		node.Next = l2
	}
	return dummy.Next
}
```

* 24. [两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)
>分析：利用dummy节点，详细分析见下

```go
//a->b->c->d
//a->c
//b->d
//c->b
//a=b
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func swapPairs(head *ListNode) *ListNode {
	dummy := &ListNode{}
	dummy.Next = head
	start := dummy
	for start.Next != nil && start.Next.Next != nil {
		p1 := start.Next
		p2 := start.Next.Next
		start.Next = p2
		p1.Next = p2.Next
		p2.Next = p1
		start = p1
	}
	return dummy.Next
}
```

* 49. [字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/)
>分析：可以先排序后再遍历，这里是开一个26大小的数组然后遍历比较

```go
func groupAnagrams(strs []string) [][]string {
	var result [][]string
	tmp := make(map[string][]string)
	for _, str := range strs {
		arr := [26]int{}
		for i := 0; i < len(str); i++ {
			arr[str[i]-97]++
		}
		var strArr []string
		for j := 0; j < 26; j++ {
			strArr = append(strArr, fmt.Sprintf("%d", arr[j]))
		}
		fiStr := strings.Join(strArr, " ")
		if v, ok := tmp[fiStr]; ok {
			v = append(v, str)
			tmp[fiStr] = v
		} else {
			tmp[fiStr] = []string{str}
		}
	}
	for _, v := range tmp {
		result = append(result, v)
	}
	return result
}
```
* 53. [最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)
>分析：遍历下一个数要比较和上一个最大值是否比较大 选择加入或者直接从这个数新开始计算最大值

```go
func maxSubArray(nums []int) int {
	maxFun := func(a, b int) int {
		if a < b {
			return b
		} else {
			return a
		}
	}
	if len(nums) == 0 {
		return 0
	}
	memo := make([]int, len(nums))
	memo[0] = nums[0]
	max := memo[0]
	for i := 1; i < len(nums); i++ {
		memo[i] = maxFun(memo[i-1]+nums[i], nums[i])
		max = maxFun(max, memo[i])
	}
	return max
}
```
* 54. [螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix/)
>分析：右下左上的遍历方向 注意边界情况

```go
/*
输入:
[
[ 1, 2, 3 ],
[ 4, 5, 6 ],
[ 7, 8, 9 ]
]
输出: [1,2,3,6,9,8,7,4,5]
*/
func spiralOrder(matrix [][]int) []int {
	var (
		left      = 0
		right     = len(matrix[0]) - 1
		top       = 0
		bottom    = len(matrix) - 1
		direction = "right"
		result    []int
	)
	for left <= right && top <= bottom {
		if direction == "right" {
			for i := left; i <= right; i++ {
				result = append(result, matrix[top][i])
			}
			top++
			direction = "bottom"
		} else if direction == "bottom" {
			for i := top; i <= bottom; i++ {
				result = append(result, matrix[i][right])
			}
			right--
			direction = "left"
		} else if direction == "left" {
			for i := right; i >= left; i-- {
				result = append(result, matrix[bottom][i])
			}
			bottom--
			direction = "top"
		} else if direction == "top" {
			for i := bottom; i >= top; i-- {
				result = append(result, matrix[i][left])
			}
			left++
			direction = "right"
		}
	}
	return result
}
```

* 55. [跳跃游戏](https://leetcode-cn.com/problems/jump-game/)
>分析：可以使用动态规划也可以用贪心算法解题，定义dp，1代表可以到达，-1代表不能到达，0是初始化未知值

``` go
func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

func jump(index int, nums []int, dp []int) bool {
	if dp[index] == 1 {
		return true
	} else if dp[index] == -1 {
		return false
	}
	maxJump := min(nums[index]+index, len(nums)-1)
	for i := index + 1; i <= maxJump; i++ {
		if jump(i, nums, dp) {
			dp[index] = 1
			return true
		}
	}
	dp[index] = -1
	return false
}

func canJump(nums []int) bool {
	dp := make([]int, len(nums))
	dp[len(nums)-1] = 1
	return jump(0, nums, dp)
}

//贪心算法
//0 1 2 3 4
//3 1 0 2 4
func canJump(nums []int) bool {
	maxJump := len(nums) - 1
	for i := len(nums) - 2; i >= 0; i-- {
		if i+nums[i] >= maxJump {
			maxJump = i
		}
	}
	return maxJump == 0
}
```

* 56. [合并区间](https://leetcode-cn.com/problems/merge-intervals/)
>分析：首先排序，然后合并区间，注意之前合并的值是会被后面的值合并的。

``` go
func merge(intervals [][]int) [][]int {
	var result [][]int

	if len(intervals) < 2 {
		return intervals
	}
	sort.SliceStable(intervals, func(i, j int) bool {
		return intervals[i][0] < intervals[j][0]
	})
	//{0, 2}, {1, 4}, {3, 5}
	for index := 0; index < len(intervals); index++ {
		if index == 0 || intervals[index][0] > result[len(result)-1][1] {
			result = append(result, intervals[index])
		} else if intervals[index][1] > result[len(result)-1][1] {
			result[len(result)-1][1] = intervals[index][1]
		}
	}
	return result
}
```

* 62. [不同路径](https://leetcode-cn.com/problems/unique-paths/)
>分析：动态规划经典题，dp方程是dp[i][j] = dp[i-1][j] + dp[i][j-1]

``` go
func uniquePaths(m int, n int) int {
	dp := make([][]int, m)
	for i := range dp {
		dp[i] = make([]int, n)
	}
	for i := 0; i < m; i++ {
		for j := 0; j < n; j++ {
			if i == 0 || j == 0 {
				dp[i][j] = 1
			} else {
				dp[i][j] = dp[i-1][j] + dp[i][j-1]
			}
		}
	}
	return dp[m-1][n-1]
}
```

* 66. [加一]](https://leetcode-cn.com/problems/plus-one/)
>分析：注意999的数组计算后结果是1000

``` go
func plusOne(digits []int) []int {
	var result []int
	for i := len(digits) - 1; i >= 0; i-- {
		if digits[i] != 9 {
			digits[i]++
			return digits
		} else {
			digits[i] = 0
		}
	}
	result = append(result, 1)
	result = append(result, digits...)
	return result
}
```

* 70. [爬楼梯]](https://leetcode-cn.com/problems/climbing-stairs/)
>分析：经典动态规划题，dp方程是dp[i] = dp[i-1] + dp[i-2]

``` go
func climbStairs(n int) int {
	if n <= 2 {
		return n
	}
	dp := make([]int, n+1)
	dp[1] = 1
	dp[2] = 2
	for i := 3; i <= n; i++ {
		dp[i] = dp[i-1] + dp[i-2]
	}
	return dp[n]
}
```