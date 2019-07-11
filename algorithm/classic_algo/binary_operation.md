## 位运算

* 2的次方
    * 只有高位为1，例如16->10000

* 四则运算
    * n % k = n & (k - 1)，满足k = 2^t 
        * 例如， 20 % 8 = 4 <==> (10100) & (00111) = 00100 = 4

    * n / k = n >>> t, 满足 k = 2^t 
        * 例如, 20 / 8 = 2 <==> (10100) >>> 3 = 00010 = 2 
        * 例如，n / 2 = n >> 1

* 异或性质
    * 抵消相同，保留不同
        * a ^ a = 0
        * a ^ b = b ^ a
        * a ^ b ^ c = a ^ c ^ b

#### 1 进制转换

* 求模取位
* 除法更新

```
//10进制转16进制
func toHex(num int) string {
    
    if num == 0 {
		return "0"
	}
    
   hexmap := [16]string{"0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "a", "b", "c", "d", "e", "f"}

	var result string
	for num != 0 {
		result = hexmap[num & 15] + result
		num = int(uint32(num) >> 4)
	}
    
	return result 
}


```

#### 2 交换数

* 异或性质

```

//交换变量a和b
a = a^b
b = b^a
a = a^b


```
