## 数组和切片

1 数组

```

//标准用法，数组需要指定长度，并且是不可变的
func main(){
    var a [2]string
    a[0] = "hello"
    a[1] = "world"
    fmt.Println(a[0], a[1])
    fmt.Println(a)

    primes := [6]int{2, 3, 5, 7, 11, 13}
    fmt.Println(primes)

    //切片，左闭右开[begin:end)
    var s []int = primes[1:4]
    fmt.Println(s)
}


```

2 切片

* 一个切片是一个数组片段的描述。它包含了指向数组的指针，片段的长度， 和容量（片段的最大长度）
* 切片类型没有给定固定的长度: []T, 有点类似于动态数组



```

//切片本质上是数组的引用，共享数组中的元素
func main(){
    names := [4]string {
        "John",
        "Paul",
        "Pony",
        "Liam"}
    fmt.Println(names)

    a := names[0:2]
    b := names[1:3]
    fmt.Println(a, b)

    b[0] = "XXX"
    fmt.Println(a, b)
    fmt.Println(names)
}

//切片文法，构建一个引用数组的切片
func main(){
    q := []int{2,3,5,7,11,13}
    fmt.Println(q)

    r := []bool{true, false, false, true}
    fmt.Println(r)

    s := []struct {
        i int
        b bool
    }{
        {2, true},
        {3, false},
    }
    fmt.Println(s)
}


//切片的默认上下界
func main(){
    s := []int{2, 3, 5, 7, 11, 13}

    s = s[1:4]
    fmt.Println(s)

    s = s[:2]
    fmt.Println(s)

    s = s[1:]
    fmt.Println(s)
}


//切片的长度和容量

func printSlice(s []int){
    fmt.Printf("len = %d cap = %d\n", len(s), cap(s))
}


func main(){
    s := []int{2, 3, 5, 7, 11, 13}
    printSlice(s)

    s = s[:0]
    printSlice(s)
    fmt.Println(s)

    s = s[:4]
    printSlice(s)
    fmt.Println(s)

    s = s[2:]
    printSlice(s)
    fmt.Println(s)
}


//nil切片
func main(){
    var s []int
    fmt.Println(s, len(s), cap(s))
    if s == nil{
        fmt.Println("nil~")
    }
}


//make创建切片
func printSlice(name string, s []int){
    fmt.Printf("%s len=%d cap=%d\n", name, len(s), cap(s))
}

func main(){
    a := make([]int, 5)
    printSlice("a", a)

    b := make([]int, 0, 5)
    printSlice("b", b)

    c := b[:2]
    printSlice("c", c)

    d := c[2:5]
    printSlice("d", d)
}


//切片的切片，二维切片，类似于二维数组
func main(){
    board := [][]string {
        []string {"_", "_", "_"},
        []string {"_", "_", "_"},
        []string {"_", "_", "_"},
    }

    board[0][0] = "X"
    board[2][2] = "O"
    board[1][2] = "X"
    board[1][0] = "O"
    board[0][2] = "X"

    for i := 0; i < len(board); i++ {
        fmt.Printf("%s\n", strings.Join(board[i], " "))
    }

}

//切片追加元素，类似于动态数组
func printSlice(s []int){
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

func main(){
    var s []int
    printSlice(s)

    s = append(s, 1)
    printSlice(s)

    s = append(s, 2, 3, 4)
    printSlice(s)
}

```

* 构建二维切片

```

//r 行数， c 列数
result := make([][]int, r)
for i := range result {
    result[i] = make([]int, c)
}

```

* binary watch 算法

```

func readBinaryWatch(num int) []string {

    var result []string
    for i := 0; i < 12; i++ {
        for j := 0; j < 60; j++ {
            if binary1Count(i) + binary1Count(j) == num {
                //formate string and slice append
                result = append(result, fmt.Sprintf("%d:%02d", i, j))
            }
        }
    }

    return result
}

//bianry opration
func binary1Count(num int) int {
    count := 0
    for num > 0 {
        if num & 1 == 1 {
            count++
        }
        num = num >> 1
    }
    return count
}

```



3 range

```

//range迭代切片元素
var powSlice = []int{1, 2, 4, 8, 16}
func main(){
    for index, value := range powSlice {
        fmt.Printf("2**%d = %d\n", index, value)
    }
}


//省略index或value
func main(){
    var pow = make([]int, 10)
    for i := range pow {
        pow[i] = 1 << uint(i)
    }

    for _, value := range pow {
        fmt.Printf("%d\n", value)
    }
}

```


