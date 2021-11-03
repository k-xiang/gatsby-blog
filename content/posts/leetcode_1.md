---
title: LeetCode上的「简单」题（一）1-20
date: 2019-08-07
category: cs
tags: ['leetcode']
---

离春招还有一个学期了，自己这半吊子技术哪里拿得出手。俗话说得好，躲的了洛谷躲不过力扣，所以开始刷题吧～ 🙇🏻‍♂️

抄一段知乎上的回答：

> LeetCode 的题目都是精选过的（尤其是前 300 题），实现出来的代码一般都在 50 行以内，很适合现场面试的时候作为笔试题。就算最简单的题目，也有一些边界问题，能看出应聘者思维的严谨程度。
>
> 举个例子吧，就拿最简单的第一题来说吧：“从一个数组中找出 2 个数 a 和 b，使得 a+b 为给定的值，返回 a 和 b 的下标。”。
>
> 首先可以肯定，这么简单的题目都做不出来的人，肯定是不符合要求的。其次这个题目有几种解法，比如说先排序，然后用二分查找，如果用这种解法做的，就可以问问他知道哪些排序方法、stable_sort 和 sort 的区别。对于二分查找，STL 的算法里面至少就有 3 个 lower_bound/upper_bound/binary_search，可以问问他这 3 个有啥区别，还可以问问他是否知道其他的 search 算法，如果他都答得很好，说明对 STL 的熟悉程度相当之高，那肯定是属于基础很牢很勤奋的人了；再如果他这题用了哈希表，可以问问他，哈希表的用法、实现等等，关于 hash 表的问题其实也有不少，比如说 rehash 等等。

## 1 两数之和

给定一个整数数组 `nums`  和一个目标值 `target`，请你在该数组中找出和为目标值的那  **两个**  整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例：

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

解：

> 下意识的思路就是两次循环暴力解了。遍历`x`同时，查找是否存在`y=target-x`。显然，时间复杂度为$O(n^2)$.

```js
var twoSum = function (nums, target) {
  for (var i = 0; i < nums.length; i++)
    for (var j = i + 1; j < nums.length; j++)
      if (nums[i] == target - nums[j]) return [i, j];
};
```

遇到的坑：

1. 最基本的 for 循环语法都忘了，把`condition`和`final-expression`写反了导致死循环

   ```js
   for ([initialization]; [condition]; [final - expression]) statement;
   ```

2. 第二次循环的时候给`j`初始化的时候忘了`+1`
3. `for(var i in nums)`中的`i`是字符串格式

高阶解法：

> 以空间换时间，将每次遍历的数打上「存在」的标记，并检查当前遍历的数是否与「已存在」的数之和为`target`。如何打上「存在」标记呢？其实就是以数为下标，存在则对应键值为下标，否则为 0.

```js
var twoSum = function (nums, target) {
  const map = {};
  for (let i = 0; i < nums.length; i++) {
    if (map[target - nums[i]] >= 0) {
      return [map[target - nums[i]], i];
    }
    map[nums[i]] = i;
  }
};
```

![](https://pic.rhinoc.top/mweb/15650986359887.jpg)

更高阶解法：

> 使用尾递归进行优化，不过性能不升反降？

```js
var twoSum = function (nums, target, i = 0, map = {}) {
  if (map[target - nums[i]] >= 0) {
    return [map[target - nums[i]], i];
  } else {
    map[nums[i]] = i;
    i++;
    return twoSum(nums, target, i, map);
  }
};
```

![](https://pic.rhinoc.top/mweb/15650990864454.jpg)

## 7 整数反转

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。
示例 1:

```
输入: 123
输出: 321
```

示例 2:

```
输入: -123
输出: -321
```

示例 3:

```
输入: 120
输出: 21
```

[[danger]]
|假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为$[-2^{31},2^{31}-1]$。请根据这个假设，如果反转后整数溢出那么就返回 0。

解（伪）：

> 使用`while`循环对`x`进行求余将每一位分离。

```js
var reverse = function (x) {
  var i = 0;
  var re = 0;
  while (parseInt(x / 10)) {
    re = 10 * re + x - 10 * parseInt(x / 10);
    x = parseInt(x / 10);
    i++;
  }
  re = 10 * re + x;
  if (re > 2147483647 || re < -2147483648) return 0;
  return re;
};
```

解（真）：

> 上面的解忽略了题目中`环境只能存储得下 32 位的有符号整数`这一限制，虽然最后对`re`进行了范围判断，但是显然在`re`已溢出的情况下范围判断是失效的。
>
> 因此，定位到产生溢出的语句`re = 10 * re + x;`，发生溢出有两种情况：

- `10 * re`时溢出
- `+ x`时溢出，当`10 * re = INT_MIN/10 或 INT_MAX/10`且`x > 7 或 x < -8`发生

```js
var reverse = function (x) {
  var re = 0;
  while (parseInt(x / 10)) {
    re = 10 * re + x - 10 * parseInt(x / 10);
    x = parseInt(x / 10);
  }
  if (re > 214748364 || re < -214748364) return 0;
  if ((re == 214748364 && x > 7) || (re == 214748364 && x < -8)) return 0;
  re = 10 * re + x;
  return re;
};
```

![](https://pic.rhinoc.top/mweb/15651034499622.jpg)

## 9 回文数

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

示例 1:

```
输入: 121
输出: true
```

示例  2:

```
输入: -121
输出: false
解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
```

示例 3:

```
输入: 10
输出: false
解释: 从右向左读, 为 01 。因此它不是一个回文数。
```

进阶：你能不将整数转为字符串来解决这个问题吗？

解：

> 和「整数反转」几乎一样的思路，看了官方题解为了避免溢出**反转一半数字**，发现我的解法**把每一位都保存下来**刚好避免了这种问题。
>
> 不过有得必有失，数组占用的空间较大。虽然按照要求没有用到字符串，但用数组逐位保存实际上还是字符串的思想，算是钻了空子。

```js
var isPalindrome = function (x) {
  if (x < 0) return false;
  var arr = [];
  var i = 0;
  while (parseInt(x / 10)) {
    //123->12
    arr[i] = x - 10 * parseInt(x / 10);
    x = parseInt(x / 10);
    i++;
  }
  var l = i;
  arr[i] = x;
  for (; i >= l / 2; i--) {
    if (arr[i] != arr[l - i]) return false;
  }
  return true;
};
```

## 13 罗马数字转整数

罗马数字包含以下七种字符: `I`，`V`，`X`，`L`，`C`，`D`和`M`。

```
字符          数值
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```

例如， 罗马数字 2 写做`II`，即为两个并列的`I`。12 写做`XII` ，即为`X + II`。 27 写做`XXVII`，即为`XX + V + II`。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做`IIII`，而是`IV`。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为`IX`。这个特殊的规则只适用于以下六种情况：

- `I`  可以放在  `V` (5) 和  `X` (10) 的左边，来表示 4 和 9。
- `X`  可以放在  `L` (50) 和  `C` (100) 的左边，来表示 40 和  90。
- `C`  可以放在  `D` (500) 和  `M` (1000) 的左边，来表示  400 和  900。

给定一个罗马数字，将其转换成整数。输入确保在 1  到 3999 的范围内。

示例  1:

```
输入: "III"
输出: 3
```

示例  2:

```
输入: "IV"
输出: 4
```

示例  3:

```
输入: "IX"
输出: 9
```

示例  4:

```
输入: "LVIII"
输出: 58
解释: L = 50, V= 5, III = 3.
```

示例  5:

```
输入: "MCMXCIV"
输出: 1994
解释: M = 1000, CM = 900, XC = 90, IV = 4.
```

解一：

> `switch`判断每一位的字母，当字母为`I`, `X`, `C`中之一时判断下一位字母与之结合是否符合特殊情况。

```js
var romanToInt = function (s) {
  var re = 0;
  for (var i = 0; i < s.length; i++) {
    switch (s[i]) {
      case 'M':
        re += 1000;
        break;
      case 'D':
        re += 500;
        break;
      case 'C':
        if (s[i + 1] == 'D') {
          re += 400;
          i++;
        } else if (s[i + 1] == 'M') {
          re += 900;
          i++;
        } else {
          re += 100;
        }
        break;
      case 'L':
        re += 50;
        break;
      case 'X':
        if (s[i + 1] == 'L') {
          re += 40;
          i++;
        } else if (s[i + 1] == 'C') {
          re += 90;
          i++;
        } else {
          re += 10;
        }
        break;
      case 'V':
        re += 5;
        break;
      case 'I':
        if (s[i + 1] == 'V') {
          re += 4;
          i++;
        } else if (s[i + 1] == 'X') {
          re += 9;
          i++;
        } else {
          re += 1;
        }
        break;
    }
  }
  return re;
};
```

解二：

> 将特殊情况替换为其他字母表示，相当于拓展字母表。并将`switch`条件判断改为映射。缺点是内存占用较大。

```js
var romanToInt = function (s) {
  s = s.replace('IV', 'Q'); //4
  s = s.replace('IX', 'W'); //9
  s = s.replace('XL', 'E'); //40
  s = s.replace('XC', 'R'); //90
  s = s.replace('CD', 'T'); //400
  s = s.replace('CM', 'Y'); //900
  var map = {
    I: 1,
    V: 5,
    X: 10,
    L: 50,
    C: 100,
    D: 500,
    M: 1000,
    Q: 4,
    W: 9,
    E: 40,
    R: 90,
    T: 400,
    Y: 900,
  };
  var re = 0;
  for (var i in s) {
    re += map[s[i]];
  }
  return re;
};
```

解三：

> 相当于在解一的基础上进行改进，当上一位字母对应数值小于当前字母对应数值时则出现了特殊情况。

```js
var romanToInt = function (s) {
  var map = {
    I: 1,
    V: 5,
    X: 10,
    L: 50,
    C: 100,
    D: 500,
    M: 1000,
  };
  var re = map[s[0]];
  for (var i = 1; i < s.length; i++) {
    if (map[s[i - 1]] >= map[s[i]]) re += map[s[i]];
    else re += map[s[i]] - 2 * map[s[i - 1]];
  }
  return re;
};
```

## 14 最长公共前缀

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串  `""`。

示例  1:

```
输入: ["flower","flow","flight"]
输出: "fl"
```

示例  2:

```
输入: ["dog","racecar","car"]
输出: ""
解释: 输入不存在公共前缀。
```

说明：所有输入只包含小写字母  a-z 。

解一：

> 逐位比较，比较全部通过时`re`增加当前字符，不通过时直接返回`re`。

```js
var longestCommonPrefix = function (strs) {
  var re = '';
  if (!strs.length) return re;
  for (var j = 0; j < strs[0].length; j++) {
    //第j位
    for (var i = 1; i < strs.length; i++) {
      //第i个
      if (strs[i][j] != strs[0][j]) return re;
    }
    re += strs[0][j];
  }
  return re;
};
```

解二：

> 解一的递归版本，需要增加一些判断语句。

```js
var longestCommonPrefix = function (strs, re = '') {
  if (!strs.length) return re;
  if (strs.length == 1) return strs[0];
  for (var i = 1; i < strs.length; i++) {
    if (!strs[i][0]) return re;
    if (strs[i][0] != strs[0][0]) return re;
    strs[i] = strs[i].slice(1, strs[i].length);
  }
  re += strs[0][0];
  strs[0] = strs[0].slice(1, strs[0].length);
  return longestCommonPrefix(strs, re);
};
```

解三：

> 和解一恰好相反，`re`初始化为数组中第一个元素，逐个比较，当比较通过时返回`re`，否则削去末位直至比较通过。这里的比较使用了正则表达式。

```js
var longestCommonPrefix = function (strs) {
  var re = strs[0] ? strs[0] : '';
  for (var i = 1; i < strs.length; i++) {
    var regex = new RegExp('^' + re);
    while (!regex.test(strs[i]) && re.length) {
      re = re.slice(0, re.length - 1);
      regex = new RegExp('^' + re);
    }
  }
  return re;
};
```

![](https://pic.rhinoc.top/mweb/15651872232259.jpg)

## 20 有效的括号

给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'`  的字符串，判断字符串是否有效。

有效字符串需满足：

- 左括号必须用相同类型的右括号闭合。
- 左括号必须以正确的顺序闭合。

[[danger]]
|注意：空字符串可被认为是有效字符串。

示例 1:

```
输入: "()"
输出: true
```

示例  2:

```
输入: "()[]{}"
输出: true
```

示例  3:

```
输入: "(]"
输出: false
```

示例  4:

```
输入: "([)]"
输出: false
```

示例  5:

```
输入: "{[]}"
输出: true
```

解一：

> 找到最内层的括号对，消去，重复此过程，若存在无法消去的字符则说明字符串无效。

```js
var isValid = function (s) {
  while (s.length) {
    var temp = s;
    s = s.replace('()', '');
    s = s.replace('[]', '');
    s = s.replace('{}', '');
    if (s == temp) return false;
  }
  return true;
};
```

解二：

> 解一跑出来的速度和内存都不尽人意，我怀疑是`replace`的问题，就用同样的思想重写了一遍，没想到性能更差了。

```js
var isValid = function (s) {
  var map = {
    '(': ')',
    '[': ']',
    '{': '}',
  };
  while (s.length) {
    var left = s[0];
    if (!(left in map)) return false;
    var i = 1;
    while (s[i] != map[left] && i < s.length) left = s[i++];
    if (s[i] != map[left]) return false;
    s = s.slice(0, i - 1) + s.slice(i + 1, s.length);
  }
  return true;
};
```

![](https://pic.rhinoc.top/mweb/15651933859184.jpg)

解三：

> 换一种思路，可以边遍历边匹配。也就是遍历的时候遇到左括号存入数组，下次遇到的第一个右括号必须和数组中最后一个元素匹配，否则为无效字符串，匹配完成后从数组中删除此元素。若最终数组为空，表示括号已全部匹配完，字符串有效。

```js
var isValid = function (s) {
  var map = {
    '(': ')',
    '[': ']',
    '{': '}',
  };
  var leftArr = [];
  for (var ch of s) {
    if (ch in map) leftArr.push(ch);
    //为左括号时，顺序保存
    else {
      //为右括号时，与数组末位匹配
      if (ch != map[leftArr.pop()]) return false;
    }
  }
  return !leftArr.length; //防止全部为左括号
};
```

![](https://pic.rhinoc.top/mweb/15651936967308.jpg)
