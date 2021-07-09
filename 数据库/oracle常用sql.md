# Oracle常用sql和函数

### 1、分组，取每组第一条数据

```sql
SELECT * FROM （
	SELECT t.*,row_number() over(partition by REPORT_ID ORDER BY INSERT_TIME) mm FROM 	tableName)
WHERE mm =1
```

### 2、时间DATE类型处理

- 日期转字符串

```java
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

```java
trunc(time)
```

- 获取当月最后一天

```sql
last_day(sysdate)
```

- 获取下个星期的某周日期,1表示周日

```java
next_day(sysdate,1)
```

### 3、null处理

```java
nvl(str,'null')
nvl(date,sysdate)
```

**注意：varchar2类型，会将空字符串当做null储存处理**

### 4、分组，查看组数>1的数据

 ```java
select * from （
SELECT t.*,COUNT(t.type) OVER(partition by t.type) mm FROM tableName  t 
） where mm>1
 ```

