## CASE表达式概述

-- 简单CASE表达式
CASE sex
    WHEN '1' THEN '男'
    WHEN '2' THEN '女'
ELSE '其他' END



-- 搜索CASE表达式
CASE WHEN sex = '1' THEN '男'
     WHEN sex = '2' THEN '女'
ELSE '其他' END



## 用check约束定义多个列的条件关系

### 问题:

用CHECK约束约束公司女性工资必须在20万日元以下

### 答案:

```sql
CONSTRAIN check_salary CHECK
    (CASE WHEN sex = '2'
        THEN CASE WHEN salary <= 200000
            THEN 1 ELSE 0 END
        ELSE 1 END = 1)
```



## 在UPDATE语句里进行条件分支

### 问题:

[<img src="https://user-images.githubusercontent.com/19871320/71479200-581b1d00-282e-11ea-87f3-a4591a37f340.png" alt="图片" style="zoom: 50%;" />](https://user-images.githubusercontent.com/19871320/71479200-581b1d00-282e-11ea-87f3-a4591a37f340.png)

### 答案:

> 注意这里一定要写ELSE, 否则会更新为NULL

```sql
UPDATE Salaries
    SET salary = CASE WHEN salary >= 30000
                        THEN salary * 0.9
                      WHEN salary >= 250000 AND salary < 280000
                        THEN salary * 1.2
                      ELSE salary END;
```



## 表之间的数据匹配

> CASE表达式里可以使用BETWEEN / LIKE / < / > 和IN / EXISTS谓词

```
-- 表的匹配: 使用谓词IN
SELECT course_name,
		CASE WHEN course_id IN
			(SELECT course_id FROM OpenCourses
			WHERE month = '200706') THEN '√'
		ELSE '×' END AS "6 月",
		CASE WHEN course_id IN
			(SELECT course FROM OpenCourses
			WHERE month = '200707') THEN '√'
		ELSE '×' END AS "7 月",
		CASE WHEN course_id IN
			(SELECT course_id FROM OpenCourses
			WHERE month = '200708') THEN '√'
		ELSE '×' END AS "8 月"
FROM CourseMaster; 

-- 表的匹配: 使用谓词EXIST
SELECT course_name,
		CASE WHEN EXIST
			(SELECT course_id FROM OpenCourses OC
			WHERE month = '200706'
				AND OC.course_id = CM.course_id) THEN '√'
		ELSE '×' END AS "6 月",
		CASE WHEN EXIST
			(SELECT course_id FROM OpenCourses OC
			WHERE month = '200707'
				AND OC.course_id = CM.course_id) THEN '√'
		ELSE '×' END AS "7 月",
		CASE WHEN EXIST
			(SELECT course_id FROM OpenCourses OC
			WHERE month = '200707'
				AND OC.course_id = CM.course_id) THEN '√'
		ELSE '×' END AS "8 月"
FROM CourseMaster AS CM; 
```



## 可重排列 排列 组合

### 排列与组合的区别

**定义:**

> 排列是有序的, 组合是无序的;

例如:

对于排序<1,2>和<2,1>, 两个是不同的排序;

对于组合{1,2}和{2,1}, 两个是相同的组合;

**排列组合的现实意义:**

> 以二人依次去超时买东西为例:

是A先去还是B先去这是有区别的, 因为A可能会将B想买的东西买完, 这时就可以说A先去, B后去是排列;

对于A来说, 超市是有物品a和物品b还是有物品b和物品a是等价的, 这时就可以说这是组合{a, b}.

### 背景:

[![图片](https://user-images.githubusercontent.com/19871320/71542215-69833700-299e-11ea-9cbd-efc6374c44fb.png)](https://user-images.githubusercontent.com/19871320/71542215-69833700-299e-11ea-9cbd-efc6374c44fb.png)

### 获取可重排列:

#### 答案:

[![图片](https://user-images.githubusercontent.com/19871320/71542262-31c8bf00-299f-11ea-9178-30266fc46013.png)](https://user-images.githubusercontent.com/19871320/71542262-31c8bf00-299f-11ea-9178-30266fc46013.png)

```
SELECT P1.name AS name_1, P2.name AS name_2
    FROM Products AS P1 INNER JOIN Products AS P2;
```

### 获取排列

[![图片](https://user-images.githubusercontent.com/19871320/71542268-4ad17000-299f-11ea-9b61-46914d9b031b.png)](https://user-images.githubusercontent.com/19871320/71542268-4ad17000-299f-11ea-9b61-46914d9b031b.png)

```
SELECT P1.name AS name_1, P2.name AS name_2
    FROM Products AS P1 INNER JOIN Products AS P2
WHERE P1.name <> P2.name;
```



### 获取组合

[![图片](https://user-images.githubusercontent.com/19871320/71542282-7fddc280-299f-11ea-8546-f29f2c3b3e7c.png)](https://user-images.githubusercontent.com/19871320/71542282-7fddc280-299f-11ea-8546-f29f2c3b3e7c.png)

```
SELECT P1.name AS name_1, P2.name AS name_2
    FROM Products AS P1 INNER JOIN Products AS P2
WHERE P1.name > P2.name;
```

### 如何让一个表自连接三次仍然能够得到一个无序的组合:

[![图片](https://user-images.githubusercontent.com/19871320/71542472-f8de1980-29a1-11ea-856a-5fc2f6351045.png)](https://user-images.githubusercontent.com/19871320/71542472-f8de1980-29a1-11ea-856a-5fc2f6351045.png)

```
SELECT P1.name, P2.name, P3.name
    FROM Products AS P1
        INNER JOIN Products AS P2
        INNER JOIN Products AS P3
WHERE P1.name > P2.name
    AND p2.name > p3.name;
```



## 删除重复行

### 问题:



[![图片](https://user-images.githubusercontent.com/19871320/71557172-68750700-2a7d-11ea-8901-edf2d60df56b.png)](https://user-images.githubusercontent.com/19871320/71557172-68750700-2a7d-11ea-8901-edf2d60df56b.png)

### 解法:

### 解法一:

```
DELETE FROM products as p1
  -- * 下面的括号里使用了关联子查询
  -- * rowid 是oracle数据库管理系统提供的
   WHERE  rowid < (SELECT MAX(p2.rowid)
                                   FROM procucts as p2
                                WHERE p1.name = p2.name
                                    AND p1.price = p2.price);
```

### 解法二:

```
delete from products as p1
  where exists (select * 
                           from products as p2
                         where p1.name = p2.name
                            and p1.price = p2.price
                            and p1.rowid < p2.rowid);
```

### 解释:

**何为关联子查询:**

- 关联子查询其实就是子查询的一种, 只是通过where语句限定了子查询与父查询之间的比较范围;
- 关于子查询是什么的详细解释可以查看<sql基础教程>的第168页;
- 关联子查询和连接虽然运算不同, 但是很多时候两者是等价的.

> 例如解法一的sql可用连接改写为:

```
delete from products
where rowid < max(select p1_id from
(select * from p1.rowid as p1_id, max(p2.rowid) as max_p2_id
from  products as p1
  inner join products as p2
    on p1.name = p2.name
       and p1.price = p2.price
group by p1.id)
where p1_id < max_p2_id );
-- 通过改写可以发现还是使用连接子查询方便
```



## 查找局部不一致的列 

### 问题:

**准备数据**

```
create table addresses (
    id serial,
    name text not null,
    family_id int not null,
    address text not null
);
insert into addresses (name, family_id, address) values
            ('前田义明', 100, '东京都港区虎之门3-2-29'),
            ('前田由美', 100, '东京都港区虎之门3-2-92'),
            ('加藤茶', 200, '东京都新宿区西新宿2-8-1'),
            ('加藤胜', 200, '东京都新宿区西新宿2-8-1'),
            ('福尔摩斯', 300, '贝克街221B'),
            ('华生', 300, '贝克街221B');
```

**问题:** 从addresses表中查出family_id相同但是address不相同的记录

### 解法:

- 使用非等值自连接

```
select distinct a1.name, a1.address
from addresses as a1, addresses as a2
where a1.family_id = a2.family_id;
```

- 使用关联子查询

```
select *
from addresses as a1
where exists (select * 
                        from addresses as a2
                      where a1.family_id = a2.family_id
                         and a1.id <> a2.id
                         and a1.address <> a2.address);
```

### 问题:

**数据准备**

```
create table products (
  id serial,
  name text not null,
  price numeric(10, 3) not null
);
insert into products (name, price)
          values('苹果', 50),
                     ('橘子',100),
                     ('葡萄', 50),
                     ('西瓜', 80),
                     ('柠檬', 30),
                     ('草莓', 100),
                     ('香蕉', 100);
```

### 答案:

- 使用非等值自连接

```sql
select distinct p1.name, p1.price
from products as p1, products as p2
where p1.price = p2.price
  and p1.name <> p2.name
order by price;
```

- 使用关联子查询

```sql
select name, price
  from products as p1
where exists (select * 
                        from products as p2
                      where p2.price = p1.price
                          and p1.name <> p2.name)
order by price;
```



# 排序 

## 问题:

将下表数据

```
id | name |  price
----+------+---------
  1 | 苹果 |  50.000
  2 | 橘子 | 100.000
  3 | 葡萄 |  50.000
  4 | 西瓜 |  80.000
  5 | 柠檬 |  30.000
  6 | 草莓 | 100.000
  7 | 香蕉 | 100.000
```

按如下方式排列:

```
 name |  price  | rank_1 | rank_2
------+---------+--------+--------
 草莓 | 100.000 |      1 |      1
 橘子 | 100.000 |      1 |      1
 香蕉 | 100.000 |      1 |      1
 西瓜 |  80.000 |      4 |      2
 苹果 |  50.000 |      5 |      3
 葡萄 |  50.000 |      5 |      3
 柠檬 |  30.000 |      7 |      4
```



## 解法:

**解法一:**

> 使用postgresql提供的窗口函数rank与dense_rank实现

```
SELECT name,
       price,
       RANK() OVER (ORDER BY price DESC) AS rank_1,
       DENSE_RANK() OVER (ORDER BY price DESC) AS rank_2,
FROM products;
```

**解法二:**

> 使用连接子查询

```
SELECT name,
        price,
        (select count(*) + 1 from products as p2 where p2.price > p1.price) as rank_1,
        (select count(*) + 1 from (select distinct price from products as p3 where p3.price > p1.price) as tmp) as rank_2,
        (select count(distinct price) +1 from products as p3 where p3.price > p1.price) as rank_3
FROM products AS p1
ORDER BY price DESC;
/*
name |  price  | rank_1 | rank_2 | rank_3
------+---------+--------+--------+--------
 草莓 | 100.000 |      1 |      1 |      1
 橘子 | 100.000 |      1 |      1 |      1
 香蕉 | 100.000 |      1 |      1 |      1
 西瓜 |  80.000 |      4 |      2 |      2
 苹果 |  50.000 |      5 |      3 |      3
 葡萄 |  50.000 |      5 |      3 |      3
 柠檬 |  30.000 |      7 |      4 |      4
*/
```

**解法三:**

> 使用自联结

```
select p1.name,
        max(p1.price) as price,
        count(p2.name) + 1 as rank_1,
        count(distinct p2.name) + 1 as rank_2
from products as p1
    -- 注意这里使用left join而不是使用内连接
    left join products as p2
        on p1.price < p2.price
group by p1.name
order by price;
```



# 用HAVING子句进行子查询: 求众数

## 背景问题:

**准备数据:**

```
create table graduates(
    name varchar(50),
    income float
);
insert into graduates(name, income)
        values('桑普森', 40000),
               ('迈克', 30000),
                ('怀特', 20000),
                ('阿诺德', 20000),
                ('史密斯', 20000),
                ('劳思伦', 15000),
                ('哈德逊', 15000),
                ('肯特', 10000),
                ('贝克', 10000),
                ('斯科特', 10000);
/*
  name  | income
--------+--------
 桑普森 |  40000
 迈克   |  30000
 怀特   |  20000
 阿诺德 |  20000
 史密斯 |  20000
 劳思伦 |  15000
 哈德逊 |  15000
 肯特   |  10000
 贝克   |  10000
 斯科特 |  10000
*/
```

**问题:** 查找出现最多的几个工资, 也就是工资的众数.

### 解法:

```
-- 使用限定谓词all

select income
    from graduates
group by income
having count(income) >= all(select count(income)
                                from graduates
                            group by income);

-- 使用极值函数max
select income
    from graduates
group by income
having count(income) >= (select max(cnt)
                            from (select count(income) as cnt
                                    from graduates
                                group by income) as tmp);
```



# 用HAVING子句进行自连接--求中位数

## 背景问题:

**准备数据:**

```
create table graduates(
    name varchar(50),
    income float
);
insert into graduates(name, income)
        values('桑普森', 40000),
               ('迈克', 30000),
                ('怀特', 20000),
                ('阿诺德', 20000),
                ('史密斯', 20000),
                ('劳思伦', 15000),
                ('哈德逊', 15000),
                ('肯特', 10000),
                ('贝克', 10000),
                ('斯科特', 10000);
/*
  name  | income
--------+--------
 桑普森 |  40000
 迈克   |  30000
 怀特   |  20000
 阿诺德 |  20000
 史密斯 |  20000
 劳思伦 |  15000
 哈德逊 |  15000
 肯特   |  10000
 贝克   |  10000
 斯科特 |  10000
*/
```

**问题:** 查找上述表格中工资的中位数, 也就是按降序排序工资出现在中间的一个.

## 解法:

**思路:**

> 将一个集合按大小顺序, 分成上半部分和下半部分两个子集, 然后取这两个子集的交集

[![图片](https://user-images.githubusercontent.com/19871320/71724386-1f50f880-2e6b-11ea-8dcc-7dd65490fed9.png)](https://user-images.githubusercontent.com/19871320/71724386-1f50f880-2e6b-11ea-8dcc-7dd65490fed9.png)

```
select avg(distinct income)
    from (select t1.income
            from graduates as t1, graduates as t2
        group by t1.income
        -- s1 的条件
        having sum(case when t2.income >= t1.income then 1 else 0 end)
            >= count(*) / 2
        -- s1 的条件
            and sum(case when t2.income <= t1.income then 1 else 0 end)
                >= count(*) / 2) as TMP;
/*
  avg
-------
 17500
(1 row)
*/
```

**解释:**

> 假设有2n个数, 从小到大依次排列
>
> - 那么第1个到第n+1个均可以从2n个数中找出>=2n/2个数比它大;
> - 那么从第n个数到第2n个数, 均可以找出<= 2n/2个数比它小; 用sql表示就是:

```
select t1.income
             from graduates as t1, graduates as t2
         group by t1.income
         -- s1 的条件
         having sum(case when t2.income >= t1.income then 1 else 0 end)
             >= count(*) / 2;
```

> 假设有2n+1个数, 从小到大依次排列
>
> - 那么从第1个数到第n+1个均可以从2n+1个数中找出>=2n+1/2个数比它大;
> - 那么从第n+1个数到第2n个数, 均可以找出<= 2n+1/2个数比它小; 用sql表示就是:

```
select t1.income
              from graduates as t1, graduates as t2
          group by t1.income
          -- s1 的条件
          having sum(case when t2.income <= t1.income then 1 else 0 end)
              >= count(*) / 2;
```

> 取两个集合的交集可以用关联子查询解决,
>
> 但是这种情况直接用having过滤出交集就行了
>
> 所以这两个子集的交集就是:

```
select avg(distinct income)
    from (select t1.income
            from graduates as t1, graduates as t2
        group by t1.income
        -- s1 的条件
        having sum(case when t2.income >= t1.income then 1 else 0 end)
            >= count(*) / 2
        -- s1 的条件
            and sum(case when t2.income <= t1.income then 1 else 0 end)
                >= count(*) / 2) as TMP;
/*
  avg
-------
 17500
(1 row)
*/
```



# 查询不包含NULL的集合

## 问题:

**准备数据:**

```
create table students1(
    student_id int not null , -- 学生id
    dpt varchar(50) not null, -- 学院
    sbmt_at timestamp with time zone -- 提交时间
);
insert into students1(student_id, dpt, sbmt_at)
                    values(100, '理学院', 'now()'),
                            (101, '理学院', 'now()'),
                            (102, '文学院', null),
                            (103, '文学院', 'now()'),
                            (104, '文学院', 'now()'),
                            (105, '文学院', null),
                            (106, '工学院', null),
                            (107, '经济学院', 'now()');               
```

**问题:** 找出所有sbmt_at(提交时间)不为空的dpt(学院)

## 解答

**利用count的特性查找:**

```
select dpt
    from students1
group by dpt
having count(*) = count(sbmt_at);

```

**使用count与case写的特征表达式查找:**

```
select dpt
    from students1
group by dpt
having count(*) = sum(case when sbmt_at is null then 0 else 1 end);
```



# 关系除法运算进行购物篮分析

## 背景问题

**准备数据:**

```
create table items(
    item varchar(100) not null -- 商品
);
insert into items(item)
            values('啤酒'),
                   ('纸尿裤'),
                    ('自行车');
/*
  item
--------
 啤酒
 纸尿裤
 自行车
(3 rows)
*/
create table shop_items(
    shop varchar(100) not null, -- 店铺
    item varchar(100) not null -- 商品
);
insert into shop_items(shop, item)
                values('仙台', '啤酒'),
                        ('仙台', '纸尿裤'),
                        ('仙台', '自行车'),
                        ('仙台', '窗帘'),
                        ('东京', '啤酒'),
                        ('东京', '纸尿裤'),
                        ('东京', '自行车'),
                        ('大阪', '电视'),
                        ('大阪', '纸尿裤'),
                        ('大阪', '自行车');
/*
shop |  item
------+--------
 仙台 | 啤酒
 仙台 | 纸尿裤
 仙台 | 自行车
 仙台 | 窗帘
 东京 | 啤酒
 东京 | 纸尿裤
 东京 | 自行车
 大阪 | 电视
 大阪 | 纸尿裤
 大阪 | 自行车
*/
```

**问题:**

1. 找出包含所有类型商品的店铺;
2. 找出所有仅包含items表中商品的店铺;

## 问题一的解法

```
-- 法一: 使用内连接
select si.shop
    from shop_items si, items i
where si.item = i.item
group by si.shop
having count(si.item) = (select count(*) from items);

-- 法二: 
select shop
    from shop_items
where item = any (select item from items)
group by shop
having count(distinct item) = (select count(*) from items);


```

## 问题二解法:

```
select si.shop
    from shop_items si
left outer join items i
    on si.item = i.item
group by si.shop
having count(si.item) = count(i.item);

```



# 用外连接进行行转列--制作交叉表

## 问题:

### 准备数据:

```
create table courses(
    name varchar(100) not null, -- 员工名称
    course varchar(100) not null -- 课程
);
insert into courses(name, course)
            values('赤井', 'SQL入门'),
                   ('赤井', 'Unix基础'),
                    ('铃木', 'SQL入门'),
                    ('工藤', 'SQL入门'),
                    ('工藤', 'Java入门'),
                    ('吉田', 'Unix基础'),
                    ('渡边', 'SQL入门');
/*
 name |  course
------+----------
 赤井 | SQL入门
 赤井 | UNIX基础
 铃木 | SQL入门
 工藤 | SQL入门
 工藤 | Java入门
 吉田 | Unix基础
 渡边 | SQL入门
(7 rows)
*/
```

### 生成如下格式的交叉表:

```
 name | SQL入门 | Unix基础 | Java入门
------+---------+----------+----------
 吉田 |         | √        |
 工藤 | √       |          | √
 渡边 | √       |          |
 赤井 | √       | √        |
 铃木 | √       |          |
```

## 解法:

### 解法一(使用关联子查询):

```
select c1.name,
      case when  exists (select *
                            from courses c2
                        where c2.name = c1.name
                            and c2.course = 'SQL入门') then '√'
        end as "SQL入门",
      case when exists (select *
                            from courses c2
                        where c2.name = c1.name
                            and c2.course = 'Unix基础') then '√'
            end as "Unix基础",
      case when exists (select *
                            from courses c2
                        where c2.name = c1.name
                            and c2.course = 'Java入门') then '√'
                end as "Java入门"
from courses c1 group by c1.name;
```

### 解法二(使用标量子查询):

```
select c1.name,
    (select '√'
        from courses c2
    where c1.name = c2.name and c2.course = 'SQL入门') as "SQL入门",
    (select '√'
        from courses c2
    where c1.name = c2.name and c2.course = 'Unix基础') as "Unix基础",
    (select '√'
        from courses c2
    where c1.name = c2.name and c2.course = 'Java入门') as "Java入门"
from courses c1
group by c1.name;
```

### 解法三(嵌套使用case表达式):

上面使用子查询的方式虽然很方便,但是在select子句中使用标量子查询和关联西查询的性能开销还是很大的, 使用嵌套case表达式就可以减少开销

```
select name,
        case when (sum(case when course = 'SQL入门' then 1 else 0 end)) > 0 then '√' end as "SQL入门",
        case when (sum(case when course = 'Unix基础' then 1 else 0 end)) > 0 then '√' end as "Unix基础",
        case when (sum(case when course = 'Java入门' then 1 else 0 end)) > 0 then '√' end as "Java入门"
    from courses
group by name;
```



# 用外连接进行列转行--汇总重复列于一行



## 问题:



### 准备数据:



```
create table personnel(
    employee varchar(100), -- 员工
    child_1 varchar(100), -- 孩子一
    child_2 varchar(100), -- 孩子二
    child_3 varchar(100) -- 孩子3
);
insert into personnel (employee, child_1, child_2, child_3)
        values('赤井', '一郎', '二郎', '三郎'),
            ('工藤', '春子', '夏子', null),
            ('铃木', '夏子', null, null),
            ('吉田', null, null, null);
/*
 employee | child_1 | child_2 | child_3
----------+---------+---------+---------
 赤井     | 一郎    | 二郎    | 三郎
 工藤     | 春子    | 夏子    |
 铃木     | 夏子    |         |
 吉田     |         |         |
(4 rows)
*/
```



### 问题:



将上述行数据改为列数据

```
 employee | child
----------+-------
 赤井     | 二郎
 赤井     | 三郎
 赤井     | 一郎
 工藤     | 春子
 工藤     | 夏子
 铃木     | 夏子
 吉田     |
(7 rows)
```



## 解法:



### 解法一(解法有问题):



> 使用union all虽然可以进行列转行, 但是最终会转出多个为null的行; 即使是使用union也会有问题, 因为可能会导致有的员工有孩子,但是却会转出一个有孩子为null的行

```
select employee, child_1 as child from personnel
union all
select employee, child_2 as child from personnel
union all
select employee, child_3 as child from personnel
order by employee;

/*
employee | child
----------+-------
 吉田     |
 吉田     |
 吉田     |
 工藤     |
 工藤     | 春子
 工藤     | 夏子
 赤井     | 一郎
 赤井     | 二郎
 赤井     | 三郎
 铃木     |
 铃木     |
 铃木     | 夏子
*/
select employee, child_1 as child from personnel
union
select employee, child_2 as child from personnel
union
select employee, child_3 as child from personnel
order by employee;
/*
 employee | child
----------+-------
 吉田     |
 工藤     |
 工藤     | 春子
 工藤     | 夏子
 赤井     | 二郎
 赤井     | 三郎
 赤井     | 一郎
 铃木     | 夏子
 铃木     |
(9 rows)
*/
```



### 解法二:



```
create view childs(child)
as select child_1 from personnel
    union
   select child_2 from personnel
    union
   select child_3 from personnel;

select personnel.employee, childs.child
    from personnel
        left outer join childs
            on childs.child in (personnel.child_1, personnel.child_2, personnel.child_3);
/*
 employee | child
----------+-------
 赤井     | 二郎
 赤井     | 三郎
 赤井     | 一郎
 工藤     | 春子
 工藤     | 夏子
 铃木     | 夏子
 吉田     |
(7 rows)
*/
drop view childs
```



# 在交叉表里制作嵌套式表侧栏

## 问题:

### 准备数据:

```
create table tbl_age(
    age_class int not null, -- 年龄层级
    age_range varchar(100) not null -- 年龄
);
insert into tbl_age(age_class, age_range)
                    values(1, '21岁~30岁'),
                            (2, '31岁~40岁'),
                            (3, '41岁~50岁');
/*
 age_class | age_range
-----------+-----------
         1 | 21岁~30岁
         2 | 31岁~40岁
         3 | 41岁~50岁
(3 rows)
*/

create table tbl_sex(
    sex_cd varchar(10) not null, -- 性别编号
    sex varchar(10) -- 性别
);
insert into tbl_sex(sex_cd, sex)
            values('m', '男'),
                    ('f', '女');
/*
 sex_cd | sex
--------+-----
 m      | 男
 f      | 女
(2 rows)
*/

create table tbl_pop(
    pref_name varchar(100) not null,
    age_class int not null,
    sex_cd  varchar(100) not null,
    population int not null
);
insert into tbl_pop(pref_name, age_class, sex_cd, population)
            values('秋田', 1, 'm', 400),
                    ('秋田', 3, 'm', 1000),
                    ('秋田', 1, 'f', 800),
                    ('秋田', 3, 'f', 1000),
                    ('青森', 1, 'm', 700),
                    ('青森', 1, 'f', 500),
                    ('青森', 3, 'f', 800),
                    ('东京', 1, 'm', 900),
                    ('东京', 1, 'f', 1500),
                    ('东京', 3, 'f', 1200),
                    ('千叶', 1, 'm', 900),
                    ('千叶', 1, 'f', 1000),
                    ('千叶', 3, 'f', 900);
/*
 pref_name | age_class | sex_cd | population
-----------+-----------+--------+------------
 秋田      |         1 | m      |        400
 秋田      |         3 | m      |       1000
 秋田      |         1 | f      |        800
 秋田      |         3 | f      |       1000
 青森      |         1 | m      |        700
 青森      |         1 | f      |        500
 青森      |         3 | f      |        800
 东京      |         1 | m      |        900
 东京      |         1 | f      |       1500
 东京      |         3 | f      |       1200
 千叶      |         1 | m      |        900
 千叶      |         1 | f      |       1000
 千叶      |         3 | f      |        900
(13 rows)
*/
```

### 问题:

生成嵌套式表侧栏的统计表:

[![p79包含嵌套式表侧栏的统计表](https://user-images.githubusercontent.com/19871320/71862137-d4b5d180-3133-11ea-8de7-92debedb44fc.png)](https://user-images.githubusercontent.com/19871320/71862137-d4b5d180-3133-11ea-8de7-92debedb44fc.png)

```
age_range | sex | pop_tohoko | pop_kanto
-----------+-----+------------+-----------
 21岁~30岁 | 男  |       1100 |      1800
 21岁~30岁 | 女  |       1300 |      2500
 31岁~40岁 | 男  |            |
 31岁~40岁 | 女  |            |
 41岁~50岁 | 男  |       1000 |
 41岁~50岁 | 女  |       1800 |      2100
```

## 解法:

```
select age_sex.age_range as age_range,
        age_sex.sex as  sex,
        case when sum(case when tbl_pop.pref_name  in ('青森', '秋田') then population else 0 end) > 0
                then sum(case when tbl_pop.pref_name  in ('青森', '秋田') then population else 0 end)
            else null end  as pop_tohoko,
        case when sum(case when tbl_pop.pref_name in ('东京', '千叶') then population else 0 end) > 0
                then sum(case when tbl_pop.pref_name in ('东京', '千叶') then population else 0 end)
            else null end  as pop_kanto
    from (select * from tbl_sex, tbl_age) as age_sex
        left outer join tbl_pop
            on tbl_pop.age_class = age_sex.age_class
            and age_sex.sex_cd = tbl_pop.sex_cd
group by age_sex.age_range, age_sex.sex
order by age_range, sex desc;
/*
 age_range | sex | pop_tohoko | pop_kanto
-----------+-----+------------+-----------
 21岁~30岁 | 男  |       1100 |      1800
 21岁~30岁 | 女  |       1300 |      2500
 31岁~40岁 | 男  |            |
 31岁~40岁 | 女  |            |
 41岁~50岁 | 男  |       1000 |
 41岁~50岁 | 女  |       1800 |      2100
*/
```



# 作为乘法运算的连接

### 准备数据:

```
create table items2(
    item_no int not null unique,
    item varchar(100) not null unique 
);
insert into items2(item_no, item)
            values(10, 'FD'),
                    (20, 'CD-R'),
                    (30, 'MO'),
                    (40, 'DVD');
/*
 item_no | item
---------+------
      10 | FD
      20 | CD-R
      30 | MO
      40 | DVD
(4 rows)
*/

create table sales_history(
    sale_date date not null,
    item_no int not null,
    quantity int not null 
);
insert into sales_history(sale_date, item_no, quantity)
            values('2017-10-01', 10, 4),
                    ('2017-10-01', 20, 10),
                    ('2017-10-01', 30, 3),
                    ('2017-10-03', 10, 32),
                    ('2017-10-03', 30, 12),
                    ('2017-10-04', 20, 22),
                    ('2017-10-04', 30, 7);
/*
 sale_date  | item_no | quantity
------------+---------+----------
 2017-10-01 |      10 |        4
 2017-10-01 |      20 |       10
 2017-10-01 |      30 |        3
 2017-10-03 |      10 |       32
 2017-10-03 |      30 |       12
 2017-10-04 |      20 |       22
 2017-10-04 |      30 |        7
(7 rows)
*/
```

### 问题:

统计每个商品销售的总量, 生成如下表格

```
 item_no | sum
---------+-----
      10 |  36
      20 |  32
      30 |  22
      40 |
(4 rows)
```

## 解法:

### 解法一(先生成一个视图在进行外连接):

```
select i.item_no, sh.quantity
    from (select sales_history.item_no, sum(quantity) quantity
            from sales_history
          group by  sales_history.item_no) sh
        right outer join items2 i
            on i.item_no = sh.item_no
order by i.item_no;
/*
 item_no | quantity
---------+----------
      10 |       36
      20 |       32
      30 |       22
      40 |
(4 rows)
*/
```

这样写虽然可以完成统计, 但是因为中间生成了一个视图, 而生成视图就要占用内存, 这样不够好, 需要优化;

### 解法二(先外连接再分组计算数量, 避免视图的生成):

```
select i.item_no, sum(sh.quantity) quantity
    from items2 i
        left outer join sales_history sh
            on i.item_no = sh.item_no
group by i.item_no
order by i.item_no;
/*
 item_no | quantity
---------+----------
      10 |       36
      20 |       32
      30 |       22
      40 |
(4 rows)
*/
```

注意: 这里因为items2表中的item_no是唯一的, 所以不会因为笛卡尔积导致总数算错, 因为1*n=n; 若是items2中的item_no不是惟一的就要做额外的考虑了;



# 全外连接

## 三种外连接:

外连接总共有三种:

- 左外连接(left outer join);
- 右外连接(right outer join);
- 全外连接(full outer join); 其中左外连接和右外连接在功能上没有区别, 都是只能用一个表作为主表. 全外连接是将两种表都作为主表.

## 从集合角度理解外连接:

- 内连接(inner join)相当于求两个集合的交集;
- 全外连接(full outer join)相当于求集合的并集;
- union也相当于求集合的并集;



# 用外连接求差集

## 准备数据:

```
create table class_a2(
    id int not null,
    name varchar(100) not null 
);
insert into class_a2(id, name)
            values(1, '田中'),
                (2, '铃木'),
                (3, '尹集院');
/*
 id |  name
----+--------
  1 | 田中
  2 | 铃木
  3 | 尹集院
(3 rows)
*/

create table class_b2(
    id int not null,
    name varchar(100) not null
);
insert into class_b2(id, name)
            values(1, '田中'),
                (2, '铃木'),
                (4, '西园寺');
/*
 id |  name
----+--------
  1 | 田中
  2 | 铃木
  4 | 西园寺
(3 rows)
*/
```

## 思路:

### 思路一(使用了关联子查询, 性能会不太好):

```
select * from class_a2
    where not exists (select *
                        from class_b2
                    where class_b2.id = class_a2.id);
/*
 id |  name
----+--------
  3 | 尹集院
(1 row)
*/
```

### 思路二(使用左或右外连接取补集):

```
select a.id, a.name
    from class_a2 a
        left outer join class_b2 b
            on a.id = b.id
where b.name is null;
/*
 id |  name
----+--------
  3 | 尹集院
(1 row)
*/
```



# 用全外连接求异或集

## 准备数据:

```
create table class_a2(
    id int not null,
    name varchar(100) not null 
);
insert into class_a2(id, name)
            values(1, '田中'),
                (2, '铃木'),
                (3, '尹集院');
/*
 id |  name
----+--------
  1 | 田中
  2 | 铃木
  3 | 尹集院
(3 rows)
*/

create table class_b2(
    id int not null,
    name varchar(100) not null
);
insert into class_b2(id, name)
            values(1, '田中'),
                (2, '铃木'),
                (4, '西园寺');
/*
 id |  name
----+--------
  1 | 田中
  2 | 铃木
  4 | 西园寺
(3 rows)
*/
```

## 思路:

```

select coalesce(a.id, b.id) as id,
        coalesce(a.name, b.name) as name
    from class_a2 a 
        full outer join class_b2 b 
            on a.id = b.id
where a.id is null
    or b.id is null;
/*
 id |  name
----+--------
  3 | 尹集院
  4 | 西园寺
(2 rows)
*/
```



# 增长/减少/维持现状

## 问题:

**准备数据:**

```
create table sales (
    year int not null,
    sale int not null
);
insert into sales(year, sale)
            values(1990, 50),
                   (1991, 51),
                    (1992, 52),
                    (1993, 52),
                    (1994, 50),
                    (1995, 50),
                    (1996, 49),
                    (1997, 55);
/*
 year | sale
------+------
 1990 |   50
 1991 |   51
 1992 |   52
 1993 |   52
 1994 |   50
 1995 |   50
 1996 |   49
 1997 |   55
(8 rows)
*/
```

**问题:** 找出销售额与上一年相等的年份

## 解法:

### 解法一(使用关联子查询):

```
select s1.year, s1.sale
    from sales s1
where s1.sale = (select sale
                    from sales s2
                where s2.year = s1.year-1
                    and s1.sale = s2.sale);
/*
year | sale
------+------
 1993 |   52
 1995 |   50
(2 rows)
*/
```

### 解法二(使用自连接):

```
select s1.year, s1.sale
    from sales s1
        inner join sales s2
            on s1.year = s2.year+1
                and s1.sale = s2.sale;
-- 或者
select s1.year, s1.sale
    from sales s1, sales s2
where s1.year = s2.year+1
    and s1.sale = s2.sale;
```



# 用列表展示与上一年的比较结果

## 问题:

**准备数据:**

```
create table sales (
    year int not null,
    sale int not null
);
insert into sales(year, sale)
            values(1990, 50),
                   (1991, 51),
                    (1992, 52),
                    (1993, 52),
                    (1994, 50),
                    (1995, 50),
                    (1996, 49),
                    (1997, 55);
/*
 year | sale
------+------
 1990 |   50
 1991 |   51
 1992 |   52
 1993 |   52
 1994 |   50
 1995 |   50
 1996 |   49
 1997 |   55
(8 rows)
```

**问题:** 比较每一年与上一年的销量, 并输出如下结果:

```
 year | sale | var
------+------+-----
 1990 |   50 | ▬
 1991 |   51 | ↑
 1992 |   52 | ↑
 1993 |   52 | →
 1994 |   50 | ↓
 1995 |   50 | →
 1996 |   49 | ↓
 1997 |   55 | ↑
```

## 解法:

### 解法一(使用关联子查询):

```
select s1.year,
    s1.sale,
    case when s1.sale = (select sale from sales s2 where s2.year+1 = s1.year) then '→'
        when s1.sale > (select sale from sales s2 where s2.year+1 = s1.year) then '↑'
        when s1.sale < (select sale from sales s2 where s2.year+1 = s1.year) then '↓'
    else '▬' end var
    from sales s1;
/*
 year | sale | var
------+------+-----
 1990 |   50 | ▬
 1991 |   51 | ↑
 1992 |   52 | ↑
 1993 |   52 | →
 1994 |   50 | ↓
 1995 |   50 | →
 1996 |   49 | ↓
 1997 |   55 | ↑
*/
```

### 解法二(使用外连接):

```
select s1.year, s1.sale,
    case when s1.sale = s2.sale then '→'
         when s1.sale > s2.sale then '↑'
        when s1.sale < s2.sale then '↓'
    else '▬' end var
    from sales s1 left outer join sales s2
        on s1.year = s2.year + 1;
/*
 year | sale | var
------+------+-----
 1990 |   50 | ▬
 1991 |   51 | ↑
 1992 |   52 | ↑
 1993 |   52 | →
 1994 |   50 | ↓
 1995 |   50 | →
 1996 |   49 | ↓
 1997 |   55 | ↑
```





# 时间轴有间断时, 和过去最邻近的时间比较

## 准备数据:

```
create table sales2 (
    year int not null,
    sale int not null
);
insert into sales2(year, sale)
            values(1990, 50),
                    (1992, 52),
                    (1993, 52),
                    (1994, 55),
                    (1997, 55);
/*
 year | sale
------+------
 1990 |   50
 1992 |   52
 1993 |   52
 1994 |   55
 1997 |   55
(5 rows)
*/
```

## 问题:

上面的数据缺少了1991/1995和1996的, 我们再想像前面一样比较上一年的就不可能了, 所以现在改成比较过去最近一年的数据. 

## 解法:

### 解法一(使用外(内)连接+极值函数MAX):

```
select s1.year, s1.sale
    from sales2 s1
        left outer join sales2 s2
            on s2.year < s1.year
                and s2.year = (select max(year)
                                    from sales2 s3
                                where s3.year < s1.year)
where s1.sale = s2.sale
order by s1.year;

-- 或者
select s1.year, s1.sale
    from sales2 s1, sales2 s2
where s1.sale = s2.sale
    and s2.year = (select max(year)
                      from sales2 s3
                    where s3.year < s1.year);

/*
 year | sale
------+------
 1993 |   52
 1997 |   55
(2 rows)
*/
```

### 解法二(使用外(内)连接+限定谓词EXISTS):

```
select s1.year, s1.sale
    from sales2 s1
        left outer join sales2 s2
            on s2.year < s1.year
                and not exists (select *
                                    from sales2 s3
                                where s3.year < s1.year
                                    and s3.year > s2.year)
where s1.sale = s2.sale
order by s1.year;

-- 或者
select s1.year, s1.sale
    from sales2 s1, sales2 s2
where s1.sale = s2.sale
    and s1.year > s2.year
    and not exists (select *
                        from sales2 s3
                    where s3.year < s1.year
                        and s3.year > s2.year);
/*
 year | sale
------+------
 1993 |   52
 1997 |   55
(2 rows)
*/
```

### 使用关联子查询:

```
select s1.year, s1.sale
    from sales2 s1
where s1.sale = (select s2.sale
                    from sales2 s2
                where s2.year = (select max(s3.year)
                                    from sales2 s3
                                where s3.year < s1.year));
/*
 year | sale
------+------
 1993 |   52
 1997 |   55
(2 rows)
*/
-- 或者
select s1.year, s1.sale
    from sales2 s1
where exists (select *
                from sales s2
             where s2.year = (select max(s3.year)
                                from sales2 s3
                            where s3.year < s1.year)
                and s2.sale = s1.sale);
```



# 移动累计值与移动平均值



## 准备数据:



```
create table accounts(
    prc_date date not null, -- 处理时间
    prc_amount int not null  -- 处理金额
);
insert into accounts(prc_date, prc_amount)
            values('2006-10-26', 12000),
                    ('2006-10-28', 2500),
                    ('2006-10-31', -15000),
                    ('2006-11-03', 34000),
                    ('2006-11-04', -5000),
                    ('2006-11-06', 7200),
                    ('2006-11-11', 11000);
/*
  prc_date  | prc_amount
------------+------------
 2006-10-26 |      12000
 2006-10-28 |       2500
 2006-10-31 |     -15000
 2006-11-03 |      34000
 2006-11-04 |      -5000
 2006-11-06 |       7200
 2006-11-11 |      11000
(7 rows)
*/
```



## 问题与解法:



### 问题一:



求截止到某个处理时间, 累计的处理金额

```
  prc_date  | prc_amount | onhand_amount
------------+------------+---------------
 2006-10-26 |      12000 |         12000
 2006-10-28 |       2500 |         14500
 2006-10-31 |     -15000 |          -500
 2006-11-03 |      34000 |         33500
 2006-11-04 |      -5000 |         28500
 2006-11-06 |       7200 |         35700
 2006-11-11 |      11000 |         46700
(7 rows)
```



**解法一(使用窗口函数):**

> [窗口函数](https://www.postgresql.org/docs/9.1/tutorial-window.html)

```
select prc_date, prc_amount,
       sum(prc_amount) over(order by prc_date) onhand_amount
    from accounts
order by prc_date;
/*
  prc_date  | prc_amount | onhand_amount
------------+------------+---------------
 2006-10-26 |      12000 |         12000
 2006-10-28 |       2500 |         14500
 2006-10-31 |     -15000 |          -500
 2006-11-03 |      34000 |         33500
 2006-11-04 |      -5000 |         28500
 2006-11-06 |       7200 |         35700
 2006-11-11 |      11000 |         46700
(7 rows)
*/
```



**解法二(使用关联子查询):**

```
select prc_date, prc_amount,
       (select sum(a2.prc_amount) from accounts a2 where a2.prc_date <= a1.prc_date) onhand_amount
from accounts a1
order by prc_date;
/*
  prc_date  | prc_amount | onhand_amount
------------+------------+---------------
 2006-10-26 |      12000 |         12000
 2006-10-28 |       2500 |         14500
 2006-10-31 |     -15000 |          -500
 2006-11-03 |      34000 |         33500
 2006-11-04 |      -5000 |         28500
 2006-11-06 |       7200 |         35700
 2006-11-11 |      11000 |         46700
(7 rows)
*/
```



### 问题二:



以连续三个时间为单位, 求累计值

```
  prc_date  | prc_amount | mvg_sum
------------+------------+---------
 2006-10-26 |      12000 |   12000
 2006-10-28 |       2500 |   14500
 2006-10-31 |     -15000 |    -500
 2006-11-03 |      34000 |   21500
 2006-11-04 |      -5000 |   14000
 2006-11-06 |       7200 |   36200
 2006-11-11 |      11000 |   13200
(7 rows)
```



**解法一(使用窗口函数):**

```
select prc_date, prc_amount,
       sum(prc_amount) over (order by prc_date
                            rows 2 preceding) mvg_sum
from accounts;
/*
  prc_date  | prc_amount | mvg_sum
------------+------------+---------
 2006-10-26 |      12000 |   12000
 2006-10-28 |       2500 |   14500
 2006-10-31 |     -15000 |    -500
 2006-11-03 |      34000 |   21500
 2006-11-04 |      -5000 |   14000
 2006-11-06 |       7200 |   36200
 2006-11-11 |      11000 |   13200
(7 rows)
*/
```



**解法二(使用关联子查询):**

```
select prc_date, prc_amount,
        (select sum(a2.prc_amount)
            from accounts a2
        where a2.prc_date <= a1.prc_date
            and (select count(*)
                    from accounts a3
                 where a3.prc_date between a2.prc_date and a1.prc_date) <= 3) mvg_sum
    from accounts a1
order by prc_date;
/*
  prc_date  | prc_amount | mvg_sum
------------+------------+---------
 2006-10-26 |      12000 |   12000
 2006-10-28 |       2500 |   14500
 2006-10-31 |     -15000 |    -500
 2006-11-03 |      34000 |   21500
 2006-11-04 |      -5000 |   14000
 2006-11-06 |       7200 |   36200
 2006-11-11 |      11000 |   13200
(7 rows)
*/
```



### 问题三:



仅当凑够连续三个日期的时候才计算累计和

```
  prc_date  | prc_amount | mvg_sum
------------+------------+---------
 2006-10-26 |      12000 |
 2006-10-28 |       2500 |
 2006-10-31 |     -15000 |    -500
 2006-11-03 |      34000 |   21500
 2006-11-04 |      -5000 |   14000
 2006-11-06 |       7200 |   36200
 2006-11-11 |      11000 |   13200
(7 rows)
```



**解法(使用关联子查询):**

```
select prc_date, prc_amount,
        (select sum(a2.prc_amount)
            from accounts a2
        where a2.prc_date <= a1.prc_date
            and (select count(*)
                    from accounts a3
                 where a3.prc_date between a2.prc_date and a1.prc_date) <= 3
        having count(*) = 3) mvg_sum -- 利用集合本身数量一定为3, 进行过滤
    from accounts a1
order by prc_date;
/*
  prc_date  | prc_amount | mvg_sum
------------+------------+---------
 2006-10-26 |      12000 |
 2006-10-28 |       2500 |
 2006-10-31 |     -15000 |    -500
 2006-11-03 |      34000 |   21500
 2006-11-04 |      -5000 |   14000
 2006-11-06 |       7200 |   36200
 2006-11-11 |      11000 |   13200
(7 rows)
*/
```



# 查询重叠的时间区间



## 准备数据:



```
create table reservations (
    reserver varchar(100) not null, -- 入住客人
    start_date date not null, -- 入住日期
    end_date date not null  -- 离店日期
);
insert into reservations(reserver, start_date, end_date)
            values('木村', '2006-10-26', '2006-10-27'),
                ('荒木', '2006-10-28', '2006-10-31'),
                ('堀', '2006-10-31', '2006-11-01'),
                ('山本', '2006-11-03', '2006-11-04'),
                ('内田', '2006-11-03', '2006-11-05'),
                ('水谷', '2006-11-06', '2006-11-06');
/*
 reserver | start_date |  end_date
----------+------------+------------
 木村     | 2006-10-26 | 2006-10-27
 荒木     | 2006-10-28 | 2006-10-31
 堀       | 2006-10-31 | 2006-11-01
 山本     | 2006-11-03 | 2006-11-04
 内田     | 2006-11-03 | 2006-11-05
 水谷     | 2006-11-06 | 2006-11-06
(6 rows)
*/
```



## 问题:

假设表中所有的客人都是入住在同一个房间的, 试找出所有入住时间与他人重合的客人.

## 解法:

```
select reserver, start_date, end_date
    from reservations s1
where exists (select *
                from reservations s2
            where s1.reserver <> s2.reserver 
                and (s1.start_date between s2.start_date and s2.end_date
                    or s1.end_date between  s2.start_date and s2.end_date
                    or (s2.start_date between s1.start_date and s1.end_date
                        and s2.end_date between  s1.start_date and s1.end_date)));
```



## 集合运算的几个注意事项:

### 1. union与union all的区别:

在集合论中不允许存在重复元素, 因此集合{1,1,2,3,3,3} 等于{1, 2, 3}.但是在关系型数据库里的表允许存在重复行.

- **对于运算的结果**: 在关系型数据库中union不允许存在重复行, 但union all允许存在重复行;
- **对于运算的性能**: 因为union为了排除重复行, 需要进行排序, 而union all就不必排序, 所以union all的性能要好于union

### 2. 集合运算符的优先级:

尽管在sql标准中intersect运算符比union和except优先级高, 但是并不是所有dbms供应商都是按照这种方式实现的.

### 3. 并不是所有的dbms供应商都:

因为早期的sql对集合运算的支持程度不高, 所以各个数据库供应商对集合运算符的实现程度也参差不齐.

### 4. 除法运算没有标准定义:

因为种种原因, 虽然和(union)差(except)积(corss join)都有标准定义, 但是商(divide by)迟迟没有标准化.



# 比较表和表 检查集合相等性之基础篇

## 问题:

**准备数据:**

```
create table tbl_a(
    key varchar(100) not null ,
    col_1 int not null,
    col_2 int not null,
    col_3 int not null
);
insert into tbl_a(key, col_1, col_2, col_3)
            values('A', 2, 3, 4),
                    ('B', 0, 7, 9),
                    ('C', 5, 1, 6);
/*
 key | col_1 | col_2 | col_3
-----+-------+-------+-------
 A   |     2 |     3 |     4
 B   |     0 |     7 |     9
 C   |     5 |     1 |     6
(3 rows)
*/
create table tbl_b(
    key varchar(100) not null ,
    col_1 int not null,
    col_2 int not null,
    col_3 int not null
);
insert into tbl_b(key, col_1, col_2, col_3)
            values('A', 2, 3, 4),
                    ('B', 0, 7, 9),
                    ('C', 5, 1, 6);
/*
 key | col_1 | col_2 | col_3
-----+-------+-------+-------
 A   |     2 |     3 |     4
 B   |     0 |     7 |     9
 C   |     5 |     1 |     6
(3 rows)
*/
```

**问题:** 在我们迁移数据库的时候, 我们会判断两个表是否相等. 那么如何比较两张表中的数据是否相等? 虽然这两张表都只有三行数据, 很容易用眼睛看出来, 但是若两张表都有上千,上万, 上亿的数据, 如何比较呢?

## 解法:

```
select (select count(*)
    from (select *
            from tbl_a
        union 
        select *
            from tbl_b) tmp) = (select count(*)
                                    from tbl_a) is_equal;
/*
 is_equal
----------
 t
(1 row)
*/
```

**解析:** 若集合S不存在重复元素, 则 S UNION S = S, 继而 S UNION S UNION S UNION ... = S. 利用这个幂等性就可以判断两个非重复集合是否相等. 这道题就是根据tbl_a, tbl_b中集合的非重复性与幂等性来判断两个集合是否相等.



# 检查集合相等性之进阶篇

## 问题:

**准备数据:**

```
create table tbl_a(
    key varchar(100) not null ,
    col_1 int not null,
    col_2 int not null,
    col_3 int not null
);
insert into tbl_a(key, col_1, col_2, col_3)
            values('A', 2, 3, 4),
                    ('B', 0, 7, 9),
                    ('C', 5, 1, 6);
/*
 key | col_1 | col_2 | col_3
-----+-------+-------+-------
 A   |     2 |     3 |     4
 B   |     0 |     7 |     9
 C   |     5 |     1 |     6
(3 rows)
*/
create table tbl_b(
    key varchar(100) not null ,
    col_1 int not null,
    col_2 int not null,
    col_3 int not null
);
insert into tbl_b(key, col_1, col_2, col_3)
            values('A', 2, 3, 4),
                    ('B', 0, 7, 9),
                    ('C', 5, 1, 6);
/*
 key | col_1 | col_2 | col_3
-----+-------+-------+-------
 A   |     2 |     3 |     4
 B   |     0 |     7 |     9
 C   |     5 |     1 |     6
(3 rows)
*/
```

## 解法:

```
select case when count(*) = 0 then '相等' else '不相等' end as result
    from ((select * from tbl_a
           union
            select * from tbl_b)
        except
         (select * from tbl_a
            intersect
        select * from tbl_b)) tmp;
```

**解析:** 若有两个集合s1和s2.则有如下推论

```
s1 = s2 <=> (s1 交 s2) = (s1 并 s2) <=> ((s1 并 s2) 差 (s1 交 s2)) = 空集
```