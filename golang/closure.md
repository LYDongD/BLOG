## 闭包

1 闭包

```

//函数返回闭包，并绑定一个独立的sum变量
func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}


func main() {
    pos, neg := adder(), adder()

    for i := 0; i < 10; i++ {
        fmt.Println(pos(i), neg(-2 * i))
    }
}


//Fibonacci数列

func fibonacci() func() int {
    a, b := 0, 1
    return func() int {
        c := a
        a = b
        b = a + c 
        return c
    }
}


func main() {
    f := fibonacci()
    for i := 0; i < 10; i++ {
        fmt.Println(f())
    }
}

```
