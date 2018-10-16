#### 解决老库主键自增问题

我们有时需要自己控制字段自增，而不能直接使用mysql的自增字段。比如一个表中有多个字段需要自增，或者其他特定需求。mysql没有oracle的sequence，需要自己实现sequence功能。mysql在SQL中可以使用变量，我们可以利用这一特性实现sequence。实现方法如下：
1）创建一个保存自增ID的数据表：
```sql
CREATE TABLE sequence (
        name VARCHAR(64) NOT NULL,
        value BIGINT(20) NOT NULL,
        PRIMARY KEY (name)
);
```

2）插入一条初始记录（如有需要，可以插入多条记录）：
```sql
INSERT INTO sequence (name, value) VALUES ('your_key_name', 0);
```
3）生成自增ID的SQL（在SQL中使用变量保存字段值的做法，可以保证原子性）：
```sql
UPDATE sequence SET value=(@nextval:=value+1) WHERE name = 'your_key_name'; SELECT @nextval
```