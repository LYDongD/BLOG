## 迭代法

### 应用

求一元高次方程的近似解，例如求平方根

#### 步骤

* 确定迭代变量和初始值
* 求递推公式
* 定义循环退出的条件，通常是某个逼近的精度


### 举例

使用牛顿迭代法求平方根

```
func mySqrt(x int) int {
       precision := 0.00001
    lastN := float64(x)
    n := lastN - f(lastN, x) / (2 * lastN - 1)
    for math.Abs(lastN - n) >= precision {
        lastN = n
        n = n - f(n, x) / (2 * n - 1)
    }
    if math.Abs(math.Floor(n + 0.5)) -  n < precision {
        return int(math.Floor(n + 0.5))
    }
    return int(n)
}

func f(n float64, x int) float64 {
    return n * n - float64(x)
}


```
