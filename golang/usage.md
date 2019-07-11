## 常用功能总结

### int/uint的最值

```
| 无符号
	| const UINT_MIN uint = 0
	| const UINT_MAX = ^uint(0)

| 有符号
	| 最大值: 首位0，其余1
		|  const INT_MAX int(^uint(0) >> 1)
	| 最小值: 首位1，其余0
		| ^INT_MAX

```

### string

```
//字符串和数组的相互转化
  
//string -> rune array
S := "abc"
s := []rune{S}

//rune array -> string
S2 = string(s)

```
