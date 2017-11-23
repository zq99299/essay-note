# mycat 实战测试

在mycat中只支持一个字段进行分片。而且不支持单库内分表。是把一张大表按照规则分到不同的库中相同的表中去。比如类似下面这样

```bash

在mycat中看到有一个db1，下面有3张表
------------------------------------
 | mycat：192.168.1.5:8066
    |- db1
       |- table1
       |- table2
       |- table3
------------------------------------

但是这三张表有可能在物理机器上是这样分布的
------------------------------------
| mysql：192.168.1.6:3306
   |- db1
      |- table1
      |- table2
      |- table3
   |- db2
      |- table1
      |- table2
      |- table3
      
| mysql：192.168.1.7:3306
   |- db1
      |- table1
      |- table2
      |- table3
   |- db2
      |- table1
      |- table2
      |- table3

------------------------------------

也就是说mycat中的一个逻辑库其实就是由n个mysql的实际库支持起来的。
```

## schema.xml 配置分片表和节点

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">
	
	<!-- 配置逻辑库 -->
	<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
		<!-- auto sharding by id (long) -->
		<!--<table name="sam_test" dataNode="dn1,dn2,dn3" rule="auto-sharding-long" autoIncrement="true" primaryKey="id_"/> -->
		<table name="SLICE_USER" dataNode="dn1,dn2,dn3,dn4,dn5,dn6,dn7,dn8,dn9,dn10,dn11,dn12" rule="auto-sharding-long" autoIncrement="true" primaryKey="id"/>
		<table name="PRODUCT" dataNode="dn1,dn2,dn3,dn4,dn5,dn6,dn7,dn8,dn9,dn10,dn11,dn12" rule="auto-sharding-long" autoIncrement="true" primaryKey="id"/>
		<table name="SLICE_PRODUCT_BROWSE" dataNode="dn1,dn2,dn3,dn4,dn5,dn6,dn7,dn8,dn9,dn10,dn11,dn12" rule="auto-sharding-long" autoIncrement="true" primaryKey="id"/>
		<table name="SLICE_ORDER" dataNode="dn1,dn2,dn3,dn4,dn5,dn6,dn7,dn8,dn9,dn10,dn11,dn12" rule="auto-sharding-long" autoIncrement="true" primaryKey="id"/>
		<table name="SLICE_SUB_ORDER" dataNode="dn1,dn2,dn3,dn4,dn5,dn6,dn7,dn8,dn9,dn10,dn11,dn12" rule="auto-sharding-long" autoIncrement="true" primaryKey="id"/>
		<table name="SLICE_SUBSCRIBE" dataNode="dn1,dn2,dn3,dn4,dn5,dn6,dn7,dn8,dn9,dn10,dn11,dn12" rule="auto-sharding-long" autoIncrement="true" primaryKey="id"/>
		<table name="SLICE_ORDER_SUMMARY_DAY" dataNode="dn1,dn2,dn3,dn4,dn5,dn6,dn7,dn8,dn9,dn10,dn11,dn12" rule="auto-sharding-long" autoIncrement="true" primaryKey="id"/>
	</schema>
	<dataNode name="dn1" dataHost="host112" database="mycat_test_db1" />
	<dataNode name="dn2" dataHost="host112" database="mycat_test_db2" />
	<dataNode name="dn3" dataHost="host112" database="mycat_test_db3" />
	<dataNode name="dn4" dataHost="host112" database="mycat_test_db4" />
	<dataNode name="dn5" dataHost="host112" database="mycat_test_db5" />
	<dataNode name="dn6" dataHost="host112" database="mycat_test_db6" />
	
	<dataNode name="dn7" dataHost="host166" database="mycat_test_db7" />
	<dataNode name="dn8" dataHost="host166" database="mycat_test_db8" />
	<dataNode name="dn9" dataHost="host166" database="mycat_test_db9" />
	<dataNode name="dn10" dataHost="host166" database="mycat_test_db10" />
	<dataNode name="dn11" dataHost="host166" database="mycat_test_db11" />
	<dataNode name="dn12" dataHost="host166" database="mycat_test_db12" />
	
	
	<dataHost name="host112" maxCon="1000" minCon="10" balance="0"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<writeHost host="hostM1" url="192.168.7.112:3306" user="root" password="123456">
		</writeHost>
	</dataHost>
	
	<dataHost name="host166" maxCon="1000" minCon="10" balance="0"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<writeHost host="hostM1" url="192.168.7.166:3306" user="root" password="123456">
	</writeHost>
	</dataHost>
</mycat:schema>
```

* schema 配置逻辑库中的表分片在哪些节点上，和关联分片规则等
* dataNode 分片节点，对应一个库
* dataHost 配置具体物理机中的数据库链接等信息

下面配置了2台物理机，和12个节点，一个节点对应一个库。
对应的逻辑表中需要分片的表有7张，全部分片到12个节点中去。

table 标签中的`rule="auto-sharding-long"`是引用了 rule.xml中定义的分片规则。


## rule.xml 定制分片规则

```xml
<mycat:rule xmlns:mycat="http://io.mycat/">
	<tableRule name="auto-sharding-long">
		<rule>
			<columns>SITE_ID</columns>
			<algorithm>rang-long</algorithm>
		</rule>
	</tableRule>
	<function name="rang-long"
		class="io.mycat.route.function.AutoPartitionByLong">
		<property name="mapFile">autopartition-long.txt</property>
	</function>
</mycat:rule>
```

* tableRule  规则名称
* `<columns>SITE_ID</columns>` 按表中的某一个字段进行分片
* `<algorithm>rang-long</algorithm>` 分片规则的映射，对应 function标签

这里用的范围分片规则

* function标签 定义调用哪一个类来处理这个规则
* mapFile 这里指定autopartition-long.txt文件来定义范围的规则

autopartition-long.txt

```
# range start-end ,data node index
# K=1000,M=10000.
0-1=0
1-2=1
2-3=2
4-5=3
5-6=4
6-7=5
7-8=6
8-9=7
9-10=8
11-12=9
13-14=10
18-19=11
```
这里我是想用site_id 的值来分片路由。从0开始，对应schema中的dataNode顺序。0-1分到第一个节点（顺序0），包含头不包含尾。

我其实就是想按站点分片。把不同站点的数据分到一个库中去。

## server.xml 服务配置

这里只列出修改过的项目

### system
* sequnceHandlerType = 0 ，使用 `config/sequence_conf.properties` 作为全局id生成器配置文件

配置逻辑库的授权信息。mycat是一个中间件，要链接mycat，在前面说过了假设有这么一个链接 `192.168.1.5:8066` 那么需要有用户名和密码。 就是这里配置的。
`
```xml
<user name="root" defaultAccount="true">
	<property name="password">123456</property>
	<property name="schemas">TESTDB</property>
</user>
```
* schemas ： 在schema.xml中schema中配置过的名称就是引用这里的

到此为止。吗，mycat
