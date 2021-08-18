# Oracle常用sql和函数

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

- 日期转字符串

```sql
to_char(time,'yyyy-mm-dd hh24:mi:ss')
```

- 字符串转日期

```sql
to_date('2021-01-01','yyyy-mm-dd hh24:mi:ss')
```

- 获取数据库系统当前时间

```sql
sysdate
```

- 截取当前时间的日期，时间使用00:00:00补充

```sql
trunc(time)
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
select * from tableName   OFFSET ((current-1)*size) ROWS FETCH NEXT (current*size) ROWS ONLY
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