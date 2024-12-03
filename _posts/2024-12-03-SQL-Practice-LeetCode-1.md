---
layout: post
title: 【复健日记】 重走 SQL 之旅 · 第一弹
categories: [SQL, LeetCode]
description: LeetCode SQL Practice
keywords: SQL, LeetCode
---

## 0. 一些废话感想

其实我的 SQL 水平从一开始就挺一般的，可能是因为我一直都没有太做过后端的工作，农行那两年有些接触，但也不过是些皮毛，这么多年下来也忘的差不多了。

为什么会又想捡起来呢？

大概是因为我始终还是想把 HR 的活做的不要那么手工账吧，就跟我六年前立下的目标一样。

虽然我已经不是程序员了，但我在骨子里，依然对于研发保有很高的热情。这一年，基本已经被学习薪酬方面的事和各种杂活占满了，年初嚎着要搞自动化要系统要这样那样的，也没有实现。

后来冷静下来想想，在只有我一个人的情况下，搞系统既要还要，性价比到底高不高？以及我到底只是想最后的年终汇报好看，还是真的弄懂了工作痛点？

一年半的薪酬工作积累，初步积累到了一些经验，虽然仍不是核心内容，只是些事务性的，比如怎么设计薪酬表使得台账统计更快、怎么做台账使得报税工作更快之类。只能说，这条路走了这么多年，才终于有机会接触到薪酬工作，必须快点克服事务性工作的负担，才有力气来思考更核心的内容。

其实我也不知道把 SQL 捡起来到底能帮我改善多少，但是以我目前对薪酬的浅显理解，一切还是以数据处理为主，直觉告诉我 SQL 一定会有用。

所以，刷题复健先吧。

## 1. 刷题笔记（LeetCode 难度: EASY ）  

### 175. Combine Two Tables

无 address 信息的也要输出空值，所以不能默认使用内联结。

本题使用左联结，语法： `SELECT ... FROM ... LEFT JOIN ... ON ...`

#### Solution

```
SELECT p.firstName, p.lastName, a.city, a.state
FROM Person p LEFT JOIN Address a
ON p.personId = a.personId;

```

### 181. Employees Earning More Than Their Managers

相当简单，因为一张表没法解决，顺理成章弄成俩表实现自联结。

#### Solution

```
SELECT e.name as "Employee"
FROM Employee e, Employee m
WHERE e.managerId = m.id and e.salary > m.salary;
```

### 182. Duplicate Emails

这题有意思的地方在于要查找的有重复项的，没重复项的不要输出，所以不能直接用 DISTINCT

- 注意 GROUP  BY 和 HAVING 的搭配使用，语法： `SELECT ... FROM ... GROUP BY ... HAVING ...`
- 直接用 WHERE + 函数会报错！

#### Solution

```
SELECT email as "Email"
FROM Person
GROUP BY email
HAVING count(email) > 1;
```

### 183. Customers Who Never Order

一开始是考虑 Customers 左联结 Orders ，然后靠 GROUP BY 和 HAVING 筛选 <1 的，但是这样不对，因为左联结会保留 Customer 所有行，哪怕无匹配的空值；所以想到改用子查询的方式来处理。

#### My Solution

```
SELECT name as "Customers"
FROM Customers
WHERE Customers.id NOT IN (SELECT c.id
FROM Customers c, Orders o
WHERE c.id = o.customerId);
```

#### A BETTER Solution

但是还是搞复杂了，子查询不需要再内联结一遍，完全可以`select customerid from orders`就可以了。

- 但是还是很猪，评论区有更好的处理：

```
SELECT c.name AS Customers
FROM Customers c
LEFT JOIN Orders o
ON c.id = o.customerId
WHERE o.customerId is NULL;
```

**注意这个 LEFT JOIN ... ON ... 之后还可以用 WHERE 的做法。**而且最重要的是，没有想清楚一个关键，就是左联结之后保留无匹配空值，“无匹配”这个就是答案了！！ （ CALLBACK 175 )

### 196. Delete Duplicate Emails

这题有 3 个要注意的点：

- DELETE 一删就是一行，所以没必要加什么 * 之类的
- WHERE ... NOT IN **必须用 SELECT 多包一层（相当于弄张新表）**，不然就会报 `You can't specify target table 'Person' for update in FROM clause` ，即**不能在同一个语句中先 SELECT 马上又发生 UPDATE** 

- 一开始我写的是 SELECT MAX(id) ，即删 id 最大的那条，这种在重复数 >2 的时候就会报错，∵只删了最大的一条，其他重复的小 id 没有删掉

#### Solution

```
DELETE
FROM Person
WHERE id NOT IN (
    SELECT dup_id FROM (
        SELECT min(id) as dup_id 
        FROM Person 
        GROUP BY email) as p);
```

### 197. Rising Temperature

#### 一些 Tips

本题有几个陷阱：

1. 表格日期不一定是连续的
2. 输出结果必须是**连续日期**中**后一天温度高于前一天温度**

本题有几个新知识：

1. 我的思路是排序后错行匹配，再筛选。
2. 获取排序后的行号用 ROW_NUMBER() OVER (ORDER BY ...) ：`ROW_NUMBER() OVER (ORDER BY recordDate) - 1 AS rowId`
3. **date 类型 +1 并不是真的日期往后一天，而是纯粹数字化后加一**，比如 2025-01-31 + 1 结果会是 2025-01-32！
	- 这就是为什么有个 testcase 用 date + 1 来比较始终无法输出 id 为 2 的行
	
```
	| id | recordDate | temperature |
	| -- | ---------- | ----------- |
	| 1  | 2015-01-31 | 10          |
	| 2  | 2015-02-01 | 25          |
	| 3  | 2015-02-03 | 20          |
	| 4  | 2015-02-04 | 30          |
```
  
正确的日期加法要用 `DATE_ADD(..., INTERVAL 1 DAY)`

- 如果不是 SELECT 里直接输出用还要再套个 DATE() 做转换先： `DATE(DATE_ADD(w1.recordDate, INTERVAL 1 DAY))`

#### My Solution

```
SELECT w2.id as "Id"
FROM (SELECT id, recordDate, temperature, ROW_NUMBER() OVER (ORDER BY recordDate) AS rowId
    FROM Weather
    ORDER BY recordDate) AS w1 INNER JOIN
    (SELECT id, recordDate, temperature, ROW_NUMBER() OVER (ORDER BY recordDate) - 1 AS rowId
    FROM Weather
    ORDER BY recordDate) AS w2
ON w1.rowId = w2.rowId
WHERE DATE(DATE_ADD(w1.recordDate, INTERVAL 1 DAY)) = w2.recordDate AND
    w1.temperature < w2.temperature;
``` 

#### A BETTER Solution

整完之后我就感觉这道 easy 的题目铁皮不应该写这么多，且获取行号这个重复代码真的很丑。
评论区也是不出意外有更简洁的写法：

```
SELECT w1.id
FROM Weather w1, Weather w2
WHERE DATEDIFF(w1.recordDate, w2.recordDate) = 1 AND w1.temperature > w2.temperature;
```

所以新知识 +1 ，**可以用 DATEDIFF() 来轧差算天数**。