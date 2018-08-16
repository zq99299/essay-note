# sharding-string-hash
功能： 根据分片键值 计算该串的 hash一致性hash值，即为最终dataNode
```xml
	<tableRule name="sharding-by-stringhash">
		<rule>
			<columns>user_id</columns>
			<algorithm>sharding-string-hash</algorithm>
		</rule>
	</tableRule>

	<function name="sharding-string-hash" class="io.mycat.route.function.PartitionByString">
		<property name="partitionLength">128,128,256</property>
		<property name="partitionCount">2,2,2</property>
		<property name="hashSlice">-1:0</property>
	</function>
```

## 配置说明：
* hashSlice : 配置 分片键值 范围截取规则；（hash预算位 格式为start:end）
* partitionLength 每个分区占用长度
* partitionCount 分区数量

hashSlice 示例：
假设 user_id = 11356789,以下规则截取出的结果如下所示

配置中的特殊值（0 means str.length(), -1 means str.length()-1）
```
“2” -> (0,2)  -> 1
“1:2” -> (1,2)  -> 11
“1:” -> (1,0) ->  1356789
“-1:” -> (-1,0) -> 9
“:-1” -> (0,-1) ->  1135678
“:” -> (0,0) -> 11356789
```

分区范围：需要partitionLength和partitionCount配合使用

```
partitionLength : 128,128,256
partitionCount  : 2,2,2  
```
所有分区范围必须是0~1024：如上所示 `128*2 + 128*2 +256 *2 = 1024` (`sum(partitionLength[i]*partitionCount[i])`)

如上配置表示：总共有6个分区,每个分区范围如下所示
```
1           2          3               4           5             6
0~128    128~256    256-384         384-512     512-768      768-1024
```
