## 指针和结构体

1 指针

```

//基本用法：获取指针和通过指针操作值
func main(){

    i, j := 42, 2701

    p := &i
    fmt.Println(*p)
    *p = 21;
    fmt.Println(i)

    p = &j
    *p = *p / 37
    fmt.Println(j)
}

```

2 结构体

```

//字段的集合
type Vertex struct {
    X int
    Y int
}

func main(){
    fmt.Println(Vertex{1,2})

    //支持点语法访问字段
    v := Vertex{1,2}
    v.X = 4
    fmt.Println(v.X, v.Y)

    //支持指针访问字段
    p := &v 
    p.X = 1e9
    fmt.Println(v)

}

//多种构建结构体的方式
type Vertex struct {
    X, Y int
}

var (
    v1 = Vertex{1, 2}
    v2 = Vertex{X: 3} // Y default 0
    v3 = Vertex{} // X,Y default 0
    p = &Vertex{1,2}
)

func main(){
    fmt.Println(v1, p, v2, v3, v3)
}



```
