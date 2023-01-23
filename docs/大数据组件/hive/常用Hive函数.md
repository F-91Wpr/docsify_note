# 常用Hive函数

1. NVL(v1, v2)

   如果v1为空，返回v2, 否则返回v1

   ```sql
   select job, nvl(job, "无职") nvl_job from emp;
   ```

2. COALESCE(v1,v2,v3.....)

   返回从左到右顺序下第一个不为null的值

   ```
   select COALESCE(null, null, 1);
   ```

3. case when

   ```sql
   -- 新建表格
   create table emp_sex(
       name string,     --姓名
       dept_id string, --部门id
       sex string       --性别
   ) 
   row format delimited fields terminated by "\t";
   -- 要求统计各个部门分男女有多少人
   ```

   数据如下

   ```bash
   +---------------+------------------+--------------+
   | emp_sex.name  | emp_sex.dept_id  | emp_sex.sex  |
   +---------------+------------------+--------------+
   | 悟空           | A                | 男           |
   | 大海           | A                | 男           |
   | 宋宋           | B                | 男           |
   | 凤姐           | A                | 女           |
   | 婷姐           | B                | 女           |
   | 婷婷           | B                | 女           |
   +---------------+------------------+--------------+
   ```

   首先尝试统计各个部门有多少人

   ```sql
   -- 统计各个部门各有多少人
   select dept_id, sum(1) from emp_sex group by dept_id;
   ```

   在此基础上，按照男女区分，即可分别统计男女

   ```sql
   select dept_id, 
     sum(1) total, 
     sum(case sex when '男' then 1 when '人妖' then 0.5 else 0 end) male,      -- 男人累加1，人妖累加0.5，其他是0
     sum(case when sex='女' then 1 when sex='人妖' then 0.5 else 0 end) female -- 女人累加1，人妖累加0.5，其他是0
   from emp_sex group by dept_id;
   ```

4. if

   范例数据和需求沿用case when，用if改写

   ```sql
   -- 统计各个部门男女各有多少人
   select dept_id,
     count(name) total,                                          -- 返回3
     count(if(sex='男', name, null)) male,                       -- 如果sex为男，count(name) 否则count(null)
     count(if(sex='女', name, null)) female                      -- 如果sex为女，count(name) 否则count(null)
   from emp_sex group by dept_id;
   ```

5. 日期相关函数

   - current_date

     ```sql
     -- 返回当前日志
     select current();
     ```

   - date_add/date_sub

     ```sql
     date_add(date, days): 返回自date开始days天后的日期
     date_sub(date, days): 返回字date开始days天前的日期
     ```

     ```sql
     -- 返回6天后的日期
     select date_add(current_date(), 6);
     -- 返回6天前的日期
     select date_sub(current_date(), 6);
     select date_add(current_date(), -6);
     ```

   - datediff

     ```
     datediff(date1, date2): 返回的是date1-date2的差值
     ```

     ```sql
     -- 返回今天和2022年2月10日的天数差
     select datediff(current_date(), '2022-02-10');
     ```

   - date_format

     ```sql
     date_format(date, format_string): 按照format_string标志的格式返回日期
     ```

     ```sql
     -- 例子
     select date_format(current_date(), 'yyyy/MM/dd');
     ```

6. 取整函数

   ```
   floor(double): 向下取整
   round(double): 四舍五入
   ceil(double): 向上取整
   ```

   ```sql
   -- 例子
   select floor(5.8), round(5.5), ceil(5.1);
   ```

7. 取年月日相关函数

   ```
   year(date): 返回年份
   month(date): 返回月份
   day(date): 返回日期
   dayofmonth(date): 同day
   dayofweek(date): 返回星期几（周日为1）
   weekofyear(date): 返回本周是今年第几周
   ```

   ```sql
   -- 例子
   select year(current_date()) `year`,
          month(current_date()) `month`,
          day(current_date()) `day`,
          dayofmonth(current_date()) `dayofmonth`,
          dayofweek(current_date()) `dayofweek`,
          weekofyear(current_date()) `weekofyear`;
   ```

# 行转列和列转行

1. 行转列

   数据如下

   | **name** | **constellation** | **blood_type** |
   | -------- | ----------------- | -------------- |
   | 孙悟空   | 白羊座            | A              |
   | 大海     | 射手座            | A              |
   | 宋宋     | 白羊座            | B              |
   | 猪八戒   | 白羊座            | A              |
   | 凤姐     | 射手座            | A              |
   | 苍老师   | 白羊座            | B              |

   期望输出：把星座和血型一样的人归类到一起。结果如下：

   ```
   射手座,A            大海|凤姐
   白羊座,A            孙悟空|猪八戒
   白羊座,B            宋宋|苍老师
   ```

   - 在hive里面建表并导入数据

     ```sql
     -- 建表
     create table person_info(
         name string,            --姓名
         constellation string, --星座
         blood_type string      --血缘
     ) 
     row format delimited fields terminated by "\t";
     ```

   - 如果需求是：求相同星座和血型的人数各是多少

     ```sql
     -- 求相同星座血型的人数
     select constellation, blood_type, count(name)
     from person_info
     group by constellation, blood_type;
     ```

     在上面的需求中，count我们称之为聚合函数(User Defined Aggregation Function)，类似函数有count, sum, avg, min, max

   - 行转列要求的是另外一个聚合函数：collect_set或collect_list

     ```
     collect_list(column): 将一列数据捏合成一个数组
     collect_set(column): 将一列数据捏合成一个数组，并去重
     ```

     ```sql
     -- 求相同星座血型的人
     select constellation, blood_type, collect_list(name)
     from person_info
     group by constellation, blood_type;
     ```

   - 拼接字符串 concat 和 concate_ws

     ```
     concat(a, b, ...) 将a, b拼接
     concat_ws(sep, array) 将数组array的所有元素拼接，用sep分隔
     ```

     ```sql
     -- 拼接
     select concat(constellation, ",", blood_type) xzxx, concat_ws("|", collect_list(name)) rentou
     from person_info
     group by constellation, blood_type;
     ```

2. 列转行

   数据如下：

   | **movie**      | **category**                 |
   | -------------- | ---------------------------- |
   | 《疑犯追踪》   | 悬疑，动作，科幻，剧情       |
   | 《Lie to  me》 | 悬疑，警匪，动作，心理，剧情 |
   | 《战狼2》      | 战争，动作，灾难             |

   希望处理成以下形式：

   ```
   《疑犯追踪》      悬疑
   《疑犯追踪》      动作
   《疑犯追踪》      科幻
   《疑犯追踪》      剧情
   《Lie to me》   悬疑
   《Lie to me》   警匪
   《Lie to me》   动作
   《Lie to me》   心理
   《Lie to me》   剧情
   《战狼2》        战争
   《战狼2》        动作
   《战狼2》        灾难
   ```

   - 建表并导入数据

     ```sql
     create table movie_info(
         movie string,     --电影名称
         category string   --电影分类
     ) 
     row format delimited fields terminated by "\t";
     ```

   - 列转行核心函数(User Defined Table-generating Function)：explode 

     ```
     explode(array|map): 可以将输入的array或map，炸开成一张表。array炸开成一列，map炸开成两列
     ```

     ```sql
     -- 有了explode，就可以尝试炸开类型字段
     -- split(str, sep), 将str用sep分成数组
     select explode(split(category,",")) from movie_info;
     ```

   - 如果想完成列转行，需要另外一种固定写法: lateral view

     ```sql
     -- 用Lateral View完成列转行
     select m.movie,
            t.category_id
     from movie_info m
     lateral view explode(split(category,",")) t as category_id;
     ```

# 窗口函数

1. 窗口函数作用是给**聚合函数**开一个数据操作的窗口

   ```sql
   -- 建表
   create table business(
       name string,        -- 顾客
       orderdate string,  -- 下单日期
       cost int             -- 购买金额
   ) 
   row format delimited fields terminated by ',';
   ```

   ```sql
   -- 查询在2017年4月购买过的顾客以及总人数
   -- 如果只求顾客
   select distinct name from business where substring(orderdate,1,7) = '2017-04';
   -- 如果只求人数
   select count(distinct name) cnt from business where substring(orderdate,1,7) = '2017-04';
   -- 用开窗函数将两者结合
   select distinct name, count(distinct name) over() from business where substring(orderdate,1,7) = '2017-04';
   ```

   ```sql
   -- 查询顾客的购买明细及月购买总额
   select name, orderdate, cost, 
          sum(cost) over(partition by substring(orderdate,1,7)) yuezonge
   from business;
   
   -- 查询明细以及每位顾客的购买总额
   select name, orderdate, cost,
          sum(cost) over(partition by name) meirenzonge
   from business;
   
   -- 查询明细以及每位顾客每月消费每单的平均值
   select name, orderdate, cost,
          avg(cost) over(partition by name, substring(orderdate,1,7)) avg_order
   from business;
   
   -- 求每个顾客的累加消费额
   select name, orderdate, cost,
          sum(cost) over(partition by name order by orderdate
                         rows between unbounded preceding and current row)
   from business;
   ```

2. 给一些专门结合窗口函数使用的函数开窗

   - lag/lead

     ```
     lag(col, n, default): 显示col这一列的前n行，如果不存在值，则显示default
     lead(col, n, default): 显示col这一列的后n行，如果不存在值，则显示default
     两个函数都需要结合有序窗口使用
     ```

     ```sql
     -- lag例子
     -- 查询顾客的购买明细和顾客的上一次购买时间，如果是第一次，则上一次显示为1970-01-01
     select name, 
            orderdate,
            lag(orderdate, 1, "1970-01-01") over(partition by name order by orderdate) last_order, 
            cost
     from business;
     ```

     ```sql
     -- lead例子
     select name, 
            lead(orderdate, 1, "1970-01-01") over(partition by name order by orderdate) next_order,
            orderdate,
            lag(orderdate, 1, "1970-01-01") over(partition by name order by orderdate) last_order, 
            cost
     from business;
     ```

   - first_value/last_value

     ```
     first_value(col, bool): 显示当前窗口的第一个值，如果第二个参数为true，显示的是该窗口第一个不为null的值
     last_value(col, bool): 显示当前窗口最后一个值，如果第二个参数为true，显示的是该窗口最后一个不为null的值
     ```

     ```sql
     -- first_value和last_value展示
     select name, orderdate, cost,
            first_value(orderdate, true) over(partition by name order by orderdate rows between 1 preceding and 1 following) fv,
            last_value(orderdate, true) over(partition by name order by orderdate rows between 1 preceding and 1 following) lv,
            if(month(orderdate)=1, null, orderdate),
            first_value(if(month(orderdate)=1, null, orderdate), true) over(partition by name order by orderdate rows between 1 preceding and 1 following) ftrue,
            first_value(if(month(orderdate)=1, null, orderdate), false) over(partition by name order by orderdate rows between 1 preceding and 1 following) ffalse
     from business;
     ```

   - ntile

     ```
     ntile(n): 表示将数据分n组，并返回本行的组号
     结合有序窗口使用
     ```

     ```sql
     -- ntile例子
     select name, orderdate, cost,
            ntile(5) over(order by orderdate) zu
     from business;
     ```

   - rank系列

     ```
     rank(): 输出该行在本窗口中的排名
     dense_rank(): 输出该行在本窗口中的排名，如果有并列不会跳排名
     row_number(): 输出该行在本窗口中的行号
     三个函数都需要结合有序窗口使用
     ```

     ```sql
     -- 建表
     create table score(
         name string,     -- 姓名
         subject string, -- 学科
         score int        -- 分数
     ) 
     row format delimited fields terminated by "\t";
     ```

     ```sql
     -- 求每门学科成绩排名
     select name, subject, score,
            rank() over(partition by subject order by score desc) `rank`,
            dense_rank() over(partition by subject order by score desc) `dense_rank`,
            row_number() over(partition by subject order by score desc) `row_number`
     from score;
     ```
