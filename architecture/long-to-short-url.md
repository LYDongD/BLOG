## url transformation system design

### REQURIMENTS

设计一个海量的长短网址转换系统


### IMPLEMENTATION

> 算法

1. 为每个长网址生成一个key，用字典保存映射关系
2. 短网址的规则为：domin + key
3. key的生成规则为：从62个字符中随机选择n个字符
4. 短网址请求时，解析key，从字典查找对应的长网址

> 系统设计

[参考](https://www.educative.io/collection/page/5668639101419520/5649050225344512/5668600916475904)

![](blogimage.ponymew.com/liam/long-2-short-url.png)




