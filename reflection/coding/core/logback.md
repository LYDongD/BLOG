## logback用法

1 关于属性additity的理解

```
| logger按照最长原则优先匹配
	| 根据logger的level过滤
	| 找appender进行输出
		| 如果additivity = true, 在输出时会向上查找所有满足条件的appender，其他logger不需要判断level
		| 如果additivity = false, 则不会查看其他appender

```
