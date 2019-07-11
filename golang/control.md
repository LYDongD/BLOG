## 流程控制

1 for

```

//标准形式，省略括号
func main(){
    sum := 0
    for i := 0; i < 10; i++ {
        sum += i
    }
    fmt.Println(sum)
}


//初始值和后置语句可以省略
func main(){

    sum := 1
    for ; sum < 10 ; {
        sum += sum
    }
    fmt.Println(sum)
}

//如果仅保留条件，相当于while
func main(){

    sum := 1
    for sum < 100 {
        sum += sum
    }
    fmt.Println(sum)
}


//如果连条件都省略，就是无限循环
func main() {
    for {
    }
}


```

2 if

```

//if 后无需括号
func sqrt(x float64) string {
    if x < 0 {
        return sqrt(-x) + "i"
    }
    return fmt.Sprint(math.Sqrt(x))
}

//if 后可以承接简短语句
func pow(x, n, limit float64) float64 {
    if v := math.Pow(x, n); v < limit {
        return v
    }else{
        fmt.Printf("%g is bigger than %g\n", v, limit)
    }

    return limit
}

```

3 综合if和for

```

//牛顿法求平方根
func Sqrt(num float64) float64 {

    z := num / 2
    for i := 0; i < 10; i++ {

        if z * z - num < 0.0000001{
            fmt.Printf("has executed %d 次, 结果:%g\n", i, z)
            return z
        }

        z -= (z * z - num) / (2 * z)
        fmt.Println(z)
    }

    return z
}

```

4 switch

```
func main(){
    //case 后的常量不必是整数，不需要break
    switch os := runtime.GOOS; os {
    case "darwin":
        fmt.Println("OS X.")
    case "linux":
        fmt.Println("Linux.")
    default:
        fmt.Printf("%s\n", os)
    }
}


func main(){
    fmt.Println("when is Wednesday?")
    today := time.Now().Weekday()
    
    //case可以是表达式
    switch time.Wednesday {
    case today + 0:
        fmt.Println("Today")
    case today + 1:
        fmt.Println("Tomorrow")
    default:
        fmt.Println("too far away")
    }

}

func main(){

    now := time.Now()
    //switch可以不接条件
    switch {
    case now.Hour() < 12:
        fmt.Println("Goood morining")
    case now.Hour() < 17:
        fmt.Println("Good afternoon")
    default:
        fmt.Println("Good evening")
    }

}

```

5 defer

```

//defer将函数调用，推迟到外层函数返回后调用
func main(){
    defer fmt.Println("world")
    fmt.Println("hello")
}


//defer的本质是将函数push入另外一个栈中，当前栈函数返回后，defer函数后进先出调用
func main(){
    fmt.Println("Counting")
    for i := 0; i < 10; i++ {
        defer fmt.Println(i)
    }

    fmt.Println("Done")
}

```
