---
layout: default
title: "LeetCode 56. 合并区间（Merge Intervals）学习笔记"
date: 2026-11-09
categories: [Java, 编程学习]
---

# LeetCode 56. 合并区间（Merge Intervals）学习笔记

> **日期：2026 年 11 月 9 日**  
> **难度：Medium**  
> **标签：排序、二维数组、贪心思想、线性扫描、Java 集合**  
> **题目链接：** https://leetcode.com/problems/merge-intervals/

## 1. 题目与目标

给定若干区间 `intervals[i] = [start, end]`，把所有重叠的区间合并，返回互不重叠、且覆盖原有全部区间的结果。

例如：

```text
输入：[[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
```

注意：`[1,4]` 和 `[4,5]` **也算重叠**，应合并成 `[1,5]`，因为两个区间有共同端点 `4`。

## 2. 核心算法：先排序，再合并

1. **按左端点升序排序**：让后面出现的区间的左端点不小于前面的左端点。
2. **维护正在合并的区间 `merged`**：初始为排序后第一个区间。
3. **从第二个区间开始遍历**，记当前区间为 `current`：
   - 若 `current[0] <= merged[1]`，两个区间重叠，更新右端点为 `max(merged[1], current[1])`。
   - 否则，说明当前的 `merged` 已经不能与之后的区间合并，把它加入结果，并令 `merged = current`。
4. **循环结束后，别忘了保存最后一个 `merged`**。

### 为什么先排序有效？

左端点升序排列后，如果下一个区间的左端点已经大于 `merged` 的右端点，那么更后面的区间左端点只会更大，不可能再与 `merged` 重叠。所以可以放心将 `merged` 写入结果。这也是本题贪心/扫描策略正确的关键。

**不要只比较两个相邻的原始区间。** 例如 `[[1,5],[2,3],[4,6]]`，虽然 `[2,3]` 与 `[4,6]` 不重叠，但 `[1,5]` 先合并 `[2,3]` 后仍然能与 `[4,6]` 合并，最终应得到 `[1,6]`。因此要比较的是 `current` 与**持续更新的** `merged`。

## 3. 本次提交的 Java 代码

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        List<int[]> result = new ArrayList<>();
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

        int[] merged = intervals[0];
        for (int i = 1; i < intervals.length; i++) {
            int[] current = intervals[i];
            if (merged[1] >= current[0]) {
                if (merged[1] < current[1]) {
                    merged[1] = current[1];
                }
            } else {
                result.add(merged);
                merged = current;
            }
        }

        result.add(merged);
        return result.toArray(new int[result.size()][]);
    }
}
```

> 上面保留了自己的原始写法。重叠时的内部 `if` 也可写成：
>
> ```java
> merged[1] = Math.max(merged[1], current[1]);
> ```
>
> 两种写法等价。

## 4. Java 知识点

### 4.1 `int[][]`、`int[]` 与 `int` 的关系

```java
int[][] intervals = {% raw %}{{1, 3}, {2, 6}, {8, 10}}{% endraw %};

int[] first = intervals[0]; // 第一个区间，即 {1, 3}
int start = intervals[0][0]; // 1：左端点
int end   = intervals[0][1]; // 3：右端点
```

| 表达式 | 类型 | 含义 |
| --- | --- | --- |
| `intervals` | `int[][]` | 所有区间组成的二维数组 |
| `intervals[i]` | `int[]` | 第 `i` 个区间 |
| `intervals[i][0]` | `int` | 第 `i` 个区间的左端点 |
| `intervals[i][1]` | `int` | 第 `i` 个区间的右端点 |
| `intervals.length` | `int` | 区间总数 |
| `intervals[i].length` | `int` | 单个区间的元素数，本题恒为 2 |

把二维数组理解成**“元素是 `int[]` 的一维数组”**，就不容易混淆。

### 4.2 对二维数组按第一列排序

```java
Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
```

- `Arrays.sort`：Java 数组排序工具。
- `(a, b) -> ...`：Lambda 表达式，用于定义两个区间如何比较。
- `a`、`b`：排序时拿来比较的两个 `int[]` 区间。
- `Integer.compare(a[0], b[0])`：比较两个区间的左端点，从小到大排序。

排序后可从左到右依次处理区间，而不需要对所有区间两两比较。

### 4.3 `List<int[]>` 保存不定数量的区间

```java
List<int[]> result = new ArrayList<>();
result.add(new int[]{1, 6});
```

`result` 是一个列表，每个元素是一个一维数组 `int[]`。因为最后合并出的区间数量不固定，用 `ArrayList` 比提前估算结果数组长度更方便。

### 4.4 将列表转换成 `int[][]`

```java
return result.toArray(new int[result.size()][]);
```

- `result` 的类型是 `List<int[]>`，**不能直接** `return result` 给返回类型为 `int[][]` 的函数。
- `toArray(...)` 返回新的二维数组，不会把原本的 `result` 变量改成数组。
- `new int[result.size()][]` 创建了一个有指定行数的二维数组类型模板；各行长度未指定，适合作为转换参数。

### 4.5 数组是引用类型：当前代码会修改输入内容

```java
int[] merged = intervals[0];
merged[1] = 6;
```

`merged` 与 `intervals[0]` 引用同一个 `int[]`，所以上面的赋值也会改变排序后 `intervals[0][1]`。而 `Arrays.sort(intervals, ...)` 本身也会重排输入数组的区间顺序。

本题可以这样写。若今后题目要求**不能修改输入数组**，则需要考虑复制数据，而不是直接修改 `merged`。

## 5. 运行过程示例

输入已按左端点排好序：

```text
[[1,3], [2,6], [8,10], [15,18]]
```

| 时刻 | 操作 | `merged` | `result` |
| --- | --- | --- | --- |
| 初始化 | 取第一个区间 | `[1,3]` | `[]` |
| 遇到 `[2,6]` | `2 <= 3`，重叠，扩展右端点 | `[1,6]` | `[]` |
| 遇到 `[8,10]` | `8 > 6`，保存旧区间 | `[8,10]` | `[[1,6]]` |
| 遇到 `[15,18]` | `15 > 10`，保存旧区间 | `[15,18]` | `[[1,6],[8,10]]` |
| 循环结束 | 保存最后一个区间 | `[15,18]` | `[[1,6],[8,10],[15,18]]` |

## 6. 复杂度分析

设区间数为 `n`：

- **排序：** `O(n log n)`。
- **单次遍历合并：** `O(n)`。
- **转换返回数组：** `O(n)`。
- **总时间复杂度：** `O(n log n)`，主要由排序决定。
- **额外空间复杂度：** 结果列表与返回数组可占 `O(n)`；排序内部也可能使用额外辅助空间。因此总体额外空间为 `O(n)`。

## 7. 本次实际遇到的错误与收获

1. **容易混淆二维数组下标。** `intervals[i]` 是一个区间，`intervals[i][0]` / `[1]` 才是具体端点。
2. **遗漏最后一个区间。** 只在“不重叠”分支调用 `result.add(merged)`，会使循环结束时仍在维护的最后一个区间丢失。需要在循环后补上一句 `result.add(merged)`。
3. **错误地返回 `List<int[]>`。** 方法要求返回 `int[][]`，应使用 `result.toArray(new int[result.size()][])`。
4. **理解了排序与贪心/扫描的配合。** 先将区间按起点排序，再维护一个已合并区间，就能将合并阶段变成一次线性扫描。
5. **注意引用语义。** `int[] merged = intervals[0]` 不是复制数组，对 `merged` 的修改会影响原数组。

## 8. 一句话总结

**本题的核心不是复杂的二维数组操作，而是：先按左端点排序，用一个不断更新的 `merged` 与后面的区间比较；重叠则扩展，不重叠则保存，最后补存收尾区间。**
