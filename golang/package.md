## 包，变量和函数


1 包和导入

```
//main包是程序的入口包
package main


//分组导入
import (
    "fmt"
    "math/rand"
)


//导出名必须是大写的
func main(){
    fmt.Println(math.Pi)
}

```


2 函数

```

//参数类型和返回类型写在后面
//从左向右读的方式对比于C来说可读性更强
func add(a int, b int) int{
    return a + b
}


//形参类型相同时，前面可以省略
func add2(a, b int) int{
    return a + b
}

//函数可以有多个返回值
func swap(x, y string)(string, string){
    return y, x
}


//函数返回值可以命名，这样return可以省略参数
func split(sum int) (x, y int){
    x = sum * 4 / 9
    y = sum - x
    return 
}

```


3 变量

```

//var声明变量列表，前面类型可以省略
var c, python, java bool

func main(){
    var i int
    fmt.Println(i, c, python, java)
}



//变量可以赋初始值，这样不必声明类型，自动推断
var c, python, java  = true, false, "no!"

func main(){
    var i int = 2
    k := 3 // 函数内赋值语句:= 可以替代关键字var
    fmt.Println(i, c, python, java)
}



//分组声明变量，变量类型中没有long，增加了复数complex
var(
    ToBe bool = false
    MaxInt uint64 = 1<<64 - 1
    z complex128 = cmplx.Sqrt(-5 + 12i)
)

func main(){
    fmt.Printf("type : %T value: %v\n", ToBe, ToBe)
    fmt.Printf("type : %T value: %v\n", MaxInt, MaxInt)
    fmt.Printf("type : %T value: %v\n", z, z)
}


//默认值0，false或空串
func main(){

    var i int 
    var j float64
    var b bool
    var s string
    fmt.Printf("%v %v %v %v\n", i, j, b, s)
}


//类型强制转换
func main(){
    var x, y int = 3, 4
    var f float64 = math.Sqrt(float64(x*x + y*y))
    var z uint = uint(f)

    fmt.Println(x, y, f, z)
}


```


4 常量

```

//const声明常量
const Pi = 3.14


func main(){
    const world = "世界"
    fmt.Println("hello", world)
    fmt.Println("happy", Pi, "day")

    const Truth = true;
    fmt.Println("Go rules?", Truth)

}

```

