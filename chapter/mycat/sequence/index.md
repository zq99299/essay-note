# 全局序列

## 数据库方式

配置步骤：

1. 在一个节点上创建一张表 `MYCAT_SEQUENCE`
2. 在该表所在数据库中创建三个function（获取当前值，设置值，获取下一个值）
3. server.xml中设置 `sequnceHandlerType = 1` 使用数据库方式
4. 表中的内容一行对应一个序列，比如可以分为用户序列，订单序列等。和文件方式一样
5. `sequence_db_conf.properties` 中配置表内容name相同的名称在那台dataNode上


### 创建表
```sql
CREATE TABLE MYCAT_SEQUENCE (
   name VARCHAR (50) NOT NULL comment  "名称",
   current_value INT NOT NULL  comment  "当前值",
   increment INT NOT NULL DEFAULT 100  comment  "步长",
   PRIMARY KEY (name)
) ENGINE = INNODB;
```

### 创建function
```sql
#取当前squence的值
DROP FUNCTION IF EXISTS mycat_seq_currval;
DELIMITER $$
CREATE FUNCTION mycat_seq_currval(seq_name VARCHAR(50))RETURNS VARCHAR(64) CHARSET 'utf8'
BEGIN
DECLARE retval VARCHAR(64);
SET retval='-999999999,NULL';
SELECT CONCAT(CAST(current_value AS CHAR),',',CAST(increment AS CHAR)) INTO retval FROM
MYCAT_SEQUENCE WHERE NAME = seq_name;
RETURN retval;
END$$
DELIMITER ;

#设置 sequence 值
DROP FUNCTION IF EXISTS mycat_seq_setval;
DELIMITER $$
CREATE FUNCTION mycat_seq_setval(seq_name VARCHAR(50),VALUE INTEGER) RETURNS VARCHAR(64) CHARSET 'utf8'
BEGIN
   UPDATE MYCAT_SEQUENCE SET current_value = VALUE    WHERE NAME = seq_name;
RETURN mycat_seq_currval(seq_name);
END$$
DELIMITER ;

#取下一个sequence的值
DROP FUNCTION IF EXISTS mycat_seq_nextval;
DELIMITER $$
CREATE FUNCTION mycat_seq_nextval(seq_name VARCHAR(50)) RETURNS VARCHAR(64) CHARSET 'utf8'
BEGIN
UPDATE MYCAT_SEQUENCE SET current_value = current_value + increment 
WHERE NAME = seq_name;
RETURN mycat_seq_currval(seq_name);
END$$
DELIMITER ;
```

### server.xml

```xml
<system>
   <property name="sequnceHandlerType">1</property>
</system>
```

### 插入一条内容

这里插入了一条名为 GLOBAL 的序列
```sql
INSERT INTO MYCAT_SEQUENCE(name,current_value,increment) VALUES (‘GLOBAL’, 100000, 100);
```

### sequence_db_conf.properties 配置

这里指明 GLOBAL 这个序列在 dnSEQUENCE 这个节点上

```
GLOBAL=dnSEQUENCE
```

### 使用示例

这里id使用了  GLOBAL  这个序列。
```sql
insert into table1(id,name) values(next value for GLOBAL,‘test’);
```

这个要起效果的话，数据库字段也要设置成 自增模式，mybatis中和之前一样也需要写上
```xml
    <selectKey resultType="java.lang.Integer" keyProperty="id" order="AFTER" >
      SELECT LAST_INSERT_ID()
    </selectKey>
```



