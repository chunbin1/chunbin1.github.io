---
title: LeetCode 刷题总结
date: 2026-05-25 10:00:00
updated: 2026-05-25 10:00:00
tags: [算法, JavaScript]
code_block_shrink: false
excerpt: 用 JavaScript 刷 LeetCode 的思路总结，涵盖哈希表、滑动窗口、双指针、栈、链表等常见解题模式。
---

刷了一些 LeetCode，把思路整理一下。不按题号，而是按**解题模式**来归类，同一类型的题往往思路相通。

## 哈希表：用空间换时间

### 1. 两数之和（Easy）

> 给定数组和目标值，找出两数之和等于目标值的下标。

暴力是两层循环 O(N²)。用哈希表只需遍历一次：**遍历时把「需要的补数」存进 map，下一个数来了先查 map 有没有**。

```js
var twoSum = function(nums, target) {
  const map = {}
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i]
    if (map[complement] !== undefined) {
      return [map[complement], i]
    }
    map[nums[i]] = i
  }
  return []
}
```

时间复杂度 O(N)，遍历一次搞定。

---

## 双指针：从两端往中间夹

### 11. 盛最多水的容器（Medium）

> 给定数组，每个元素表示一根柱子的高度，找出两根柱子能装最多水的组合。

面积 = `min(左高, 右高) × 宽度`。从最宽（两端）开始，**每次移动较矮的一侧**，因为移动较高一侧只会让面积更小。

```js
var maxArea = function(height) {
  let max = 0
  let left = 0
  let right = height.length - 1
  while (left < right) {
    max = Math.max(max, Math.min(height[left], height[right]) * (right - left))
    if (height[left] > height[right]) {
      right--
    } else {
      left++
    }
  }
  return max
}
```

---

## 滑动窗口：维护一个合法区间

### 3. 无重复字符的最长子串（Medium）

> 找出字符串中不含重复字符的最长子串长度。

用左右两个指针维护一个无重复的窗口。右指针右移扩大窗口，遇到重复字符就左移收缩，直到窗口内无重复。

```js
var lengthOfLongestSubstring = function(s) {
  let left = 0, right = 0
  let max = 0
  const map = {}
  while (right < s.length) {
    if (map[s[right]] === undefined) {
      map[s[right]] = 1
      right++
      max = Math.max(max, right - left)
    } else {
      delete map[s[left]]
      left++
    }
  }
  return max
}
```

---

## 链表：模拟进位

### 2. 两数相加（Medium）

> 两个链表分别表示两个逆序整数，返回它们相加结果的链表。

按位相加，维护一个进位 `carry`。只要三者（l1、l2、carry）有一个不为 0，循环就继续。

```js
var addTwoNumbers = function(l1, l2) {
  const dummy = new ListNode(0)
  let cur = dummy
  let carry = 0
  while (l1 || l2 || carry) {
    let sum = carry
    if (l1) { sum += l1.val; l1 = l1.next }
    if (l2) { sum += l2.val; l2 = l2.next }
    carry = Math.floor(sum / 10)
    cur.next = new ListNode(sum % 10)
    cur = cur.next
  }
  return dummy.next
}
```

---

## 字符串：中心扩散 / 贪心 / 规律

### 5. 最长回文子串（Medium）

> 找出字符串中最长的回文子串。

**中心扩散法**：枚举每个字符作为回文中心，分奇偶两种情况向两边扩散，记录最长区间。

```js
var longestPalindrome = function(s) {
  let start = 0, end = 0
  for (let i = 0; i < s.length; i++) {
    const len = Math.max(expand(s, i, i), expand(s, i, i + 1))
    if (len > end - start) {
      start = i - Math.floor((len - 1) / 2)
      end = i + Math.floor(len / 2)
    }
  }
  return s.substring(start, end + 1)
}

function expand(s, left, right) {
  while (left >= 0 && right < s.length && s[left] === s[right]) {
    left--; right++
  }
  return right - left - 1
}
```

### 12 / 13. 罗马数字互转（Medium）

**整数转罗马**：贪心，维护一个从大到小的值表，每次尽量用最大的去减。

```js
var intToRoman = function(num) {
  const values = [1000,900,500,400,100,90,50,40,10,9,5,4,1]
  const symbols = ['M','CM','D','CD','C','XC','L','XL','X','IX','V','IV','I']
  let res = ''
  values.forEach((val, i) => {
    while (num >= val) {
      num -= val
      res += symbols[i]
    }
  })
  return res
}
```

**罗马转整数**：从左往右扫，当前字符比下一个小（如 IV 中的 I）就减，否则加。

```js
var romanToInt = function(s) {
  const map = { M:1000, D:500, C:100, L:50, X:10, V:5, I:1 }
  let res = 0
  for (let i = 0; i < s.length; i++) {
    map[s[i]] < map[s[i + 1]] ? res -= map[s[i]] : res += map[s[i]]
  }
  return res
}
```

### 14. 最长公共前缀（Easy）

用 `reduce` 两两比较，每次求出当前公共前缀再和下一个对比：

```js
var longestCommonPrefix = function(strs) {
  if (!strs.length) return ''
  return strs.reduce((pre, next) => {
    let i = 0
    while (pre[i] && next[i] && pre[i] === next[i]) i++
    return pre.slice(0, i)
  })
}
```

---

## 栈：括号匹配

### 20. 有效的括号（Easy）

> 判断括号字符串是否合法。

遇到左括号压栈，遇到右括号检查栈顶是否匹配。最后栈为空则合法。

```js
var isValid = function(s) {
  if (s.length % 2 !== 0) return false
  const map = { '(': ')', '[': ']', '{': '}' }
  const stack = []
  for (const c of s) {
    if (map[c]) {
      stack.push(map[c])
    } else {
      if (stack.pop() !== c) return false
    }
  }
  return stack.length === 0
}
```

---

## 总结

| 模式 | 适用场景 | 代表题 |
|------|---------|--------|
| 哈希表 | 查找补数、去重 | 两数之和 |
| 双指针 | 有序数组、两端夹逼 | 盛最多水 |
| 滑动窗口 | 连续子数组/子串 | 最长无重复子串 |
| 中心扩散 | 回文类问题 | 最长回文子串 |
| 贪心 | 有规律的最优选择 | 罗马数字转换 |
| 栈 | 括号匹配、逆序处理 | 有效括号 |
| 链表模拟 | 按位操作、进位 | 两数相加 |

算法题的核心不是记答案，而是认清「这道题属于哪种模式」，然后套对思路。
