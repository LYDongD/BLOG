## 映射

1 映射

```

//使用make构建一个映射
type Vertex2 struct {
    Lat, Long float64
}

//key为string, value为struct
var m map[string] Vertex2

func main() {
    m = make(map[string] Vertex2)
    m["pony-liam"] = Vertex2{
        40.345, -23.11,
    }
    fmt.Println(m)
}


//声明初始化
var m = map[string] Vertex2{
    "pony" : Vertex2{1.5, 2.0},
    "liam" : Vertex2{-2.9, 4.5},
}

//可省略value类型名
var m = map[string] Vertex2{
    "pony" : {1.5, 2.0},
    "liam" : {-2.9, 4.5},
}


//hash的访问，插入，修改和删除
func main() {
    m := make(map[string]int)

    m["Answer"] = 2
    fmt.Println("the value: ", m["Answer"])

    delete(m, "Answer")
    fmt.Println("the value: ", m["Answer"])

    value, ok := m["Answer"]

    fmt.Println("the value: ", value, "Present? ", ok)

}


//实现一个字符统计
func WordCount(s string) map[string]int {

    wordCountMap := make(map[string]int)

    for _, char := range s {
        wordCountMap[string(char)]++
    }
    return wordCountMap
}

```


