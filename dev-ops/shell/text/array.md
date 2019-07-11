## 数组

### 关联数组

shell中关联数组的索引可以是字符串类型

### 常用操作

```

#声明一个数组 declare -a
declare -a fruit_price_array
fruit_price_array=([apple]='10' [orange]='20' [banana]='30')

#获取数组值${}
#打印索引 !array[*]
echo ${!fruit_price_array[*]}

#打印索引值 array[*]
echo ${fruit_price_array[*]}

#打印苹果的价格
echo "${fruit_price_array[apple]}"

```
