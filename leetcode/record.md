# leetcode record

### cache（待定问题）
时间复杂度 空间复杂度


## 1.两数之和
Q：给到一个数组和一个目标和，找出和为目标值的两下标

sol1


## 3.无重复字符的最长字符串
```js
/**
 * 可以把res想象成一个滑块，没有重复的就增加滑块长度，
 * 遇到重复就开始右移行走，一次走一格，直到走出包含重复字符的区域
 */
let findMax = function(str) {
    let res = [];
    let max = 0;
    for (let s of str) {
        while (res.includes(s)) {
            res.shift();
        }
        res.push(s);
        max = Math.max(max, res.length);
    }
    return max;
}
```