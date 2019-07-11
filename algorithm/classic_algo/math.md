## 常用数学算法

### 1 求最大公约数

辗转相除法: 不断求模直到模数为0，取被除数最为最大公约数

```
func findGcd(a, b int) int{
    divisor, divisored := -1, -1
    if a >= b {
        divisor = a
        divisored = b
    }else {
        divisor = b
        divisored = a
    }

    if divisored == 0 {
        return divisor
    }

    return findGcd(divisor % divisored, divisored)
}

```

参考题型leetcode[914]


