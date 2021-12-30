# Oracle开发常用

## Oracle常用sql和函数

### 1、分组，取每组第一条数据

```sql
SELECT * FROM （
	SELECT t.*,row_number() over(partition by REPORT_ID ORDER BY INSERT_TIME) mm FROM 	tableName)
WHERE mm =1
```

### 2、时间DATE类型处理

- DATE类型用于表示 日期+时间，当只有日期时，则默认使用00:00:00补充，因此当进行日期区间比较时，应该手动补全时间，否则最后一天将会被忽略：

```
time between to_date('2021-01-01 00:00:00','yyyy-mm-dd hh24:mi:ss') and to_date('2021-01-31 23:59:59','yyyy-mm-dd hh24:mi:ss')
```

- 日期转字符串,获取指定参数值

```sql
to_char(time,'yyyy-mm-dd hh24:mi:ss') 2019-08-03 10:58:44

to_char(time,"mm")  08
```

- 字符串转日期

```sql
to_date('2021-01-01','yyyy-mm-dd hh24:mi:ss')
```

- 获取数据库系统当前时间

```sql
sysdate
```

- 截取当前时间某个值,默认为yyyy-mm-dd

```sql
trunc(time,"yyyy-mm-dd") 2019-08-03 00:00:00

trunc(time,"yyyy")    2019-01-01 00:00:00
```

- 获取当月最后一天

```sql
last_day(sysdate)
```

- 获取下个星期的某周日期,1表示周日

```sql
next_day(sysdate,1)
```

### 3、null处理

```sql
nvl(str,'null')
nvl(date,sysdate)
```

**注意：varchar2类型，会将空字符串当做null储存处理**

### 4、分组，查看组数>1的数据

 ```sql
select * from （
SELECT t.*,COUNT(t.type) OVER(partition by t.type) mm FROM tableName  t 
） where mm>1
 ```

### 5、结果集添加常量，并指定字段名

```sql
SELECT t.*,1 AS NUM FORM tableName t
```

### 6、分页：

oracle12c：

```sql
select * from tableName   OFFSET ((current-1)*size) ROWS FETCH NEXT (size) ROWS ONLY
```

oracle11g:

```sql
select * from （
	select t.*,rownum rn from tableName t where ROWNUM <= (current*size)
) WHERE rn >((current-1)*size)
```

**不直接使用rn between  1 and 10的原因:**

rownum是一个伪列，会根据当前临时表的条数，进行动态排序；从而表记录1开始，判断如果rownum>1,则第一条数据不满足，则被提出临时表，而此时动态的第二条记录的rownum会变为1，这样往复，最后得到一个空的临时表

因此需要先在临时表中，判断最大范围，然后再子查询中，rownum不在是伪列，在判断最小范围

**当然也可以直接在子查询中完成（推荐）：**

```sql
select * from (
	select t.*,rownum rn from tableName 
) where rn between 1 and 10
```

**rownum分页排序注意：**

在使用在使用rownum进行分页时，不能把order by语句和rownum的条件判断写在一个查询中，必须通过子查询分开，因为判断条件优先于排序，因此后排序会打乱rownum值

### 7、Date类型比较：

- 获取两个Date类型的相差天数，如果两个Date的时分秒不同时，则为小数

  ```java
  to_number(date1-date2) days
  ```

  基于天数，就可以推出两个Date类型相差的小时数（* 24）、分钟数（ * 24 *60）、秒数（ * 24 * 60 *60）

- 获取两个Date类型的相差月数，默认会忽略时分秒的影响；当日期不同，且不会月份的最后一天时，则为小数

   ```java
  months_between(date1,date2) months
  ```

  基于月份，可以推出两个Date类型相差的年数（/12）

### 8、将数据insert到相同结构的另外表中：

```sql
insert into tableName1 select * from tableName2
```

### 9、交集、并集、差集

**必须保证两组数据的字段数、类型相同，一 一对应**

- intersect

  获取两组数据的交集

  ```sql
  SELECT id FROM tab1   
  INTERSECT    
  SELECT id FROM tab2; 
  ```

- minus

  获取第一个查询结果与第二个查询结果的不同部分，即差集

- union/union all

  获取两组数据的并集，union会将重复内容取一个，union all则允许有重复内容

### 10、字符串长度、截取、替换、拼接和判断是否包含

- length（str）字符串长度
- substr(str, m, n)  截取字符串的子串
- replace(str, search_string, replace_string)   字符串替换
- str1||str2   用于将多个字符串拼接
- instr（str,'subStr'）判读是否包含某个字符串，包含则返回匹配开始位置N，因此如果instr返回值>0，则包含，反之不包含

注意：

**可以使用instr函数来代替like通配符，并且在like语句无法走索引的前提下，当数据量比较大时，instr要比like效率高**

### 11、批量新增

```sql
INSERT ALL
   INTO MY_TABLE (ID, CREATE_TIME, MODIFY_TIME, NAME) VALUES ('1', SYSDATE, SYSDATE, '我的表 1')
   INTO MY_TABLE (ID, CREATE_TIME, MODIFY_TIME, NAME) VALUES ('2', SYSDATE, SYSDATE, '我的表 2')
   INTO YOUR_TABLE (ID, CREATE_TIME, MODIFY_TIME, NAME) VALUES ('1', SYSDATE, SYSDATE, '你的表 1')
 SELECT * FROM dual
```

**dual为一个空表**，oracle中经常使用 SELECT * FROM dual该语句，作为一些特殊sql的补充；防止oracle报缺失关键字select的错误

**1、INSERT ALL必须搭配一个子查询使用，无论是否需要该子查询的表数据**

**2、既可以插入同一个表，又可以插入不同表**

## Oracle使用注意

### 1、查询数据总行数：

​		oracle提供四种count：count（1）、count（*）、count（rowid）、count（列名），区别如下：

1、前三个不会忽略为null，count（列名）则会忽略

2、没有主键时，count（1）效率最高

3、有主键时，count（主键列），count（ * ）效率最高，两种没有太大区别，因为count（ * ）会自动优化使用主键列进行计算

### 2、别名使用：		

- oracle对表名添加别名时，一定要省略as保留字，原因：

  as在oracle存储过程有其他含义，在表名后添加as时，会产生歧义，因此Oracle禁止使用

- oracle在select子句中使用*后，就无法使用字段名，必须为表名添加别名，然后引入其他字段

  ```sql
  select t.*,t.username from tableName t
  ```

### 3、select子句对聚合函数字段处理：

​		oracle在select子句中使用聚合函数后，就无法添加其他非group by表字段；但oracle可以通过分析函数来实现：查询所有表字段的同时，加入聚合函数字段



