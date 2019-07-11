## 方法和接口


* 在java中，类即对象
* 在go中，没有类，或认为类可以是struct，float64等类型的类
* golang的方法可以定义为"类"或指针的方法

1 方法


* 值接收
* 指针接收
* 建议采用指针作为方法的接受者
    * 可修改类型实体的属性
    * 避免大结构复制带来的开销

```

//golang中的结构体类似于java中的类
type Vertex struct {
    X, Y float64
}

//为结构体定义方法, 结构体是方法的接受者
func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X * v.X + v.Y * v.Y)
}

func main() {
    v := Vertex{3, 4}
    fmt.Println(v.Abs())
}



//也可以为其他类型定义方法
type MyFloat float64

func (f MyFloat) Abs() float64 {
    if f < 0 {
        return float64(-f)
    }

    return float64(f)
}


//指针接收者可以修改接受者的属性，否则只是操作副本
func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}
```

* 方法的接受者为byte,struct和slice

```
const (
    WHITE = iota
    BLACK
    BLUE
    RED
    YELLOW
)

type Color byte

type Box struct {
    width, height, depth float64
    color                Color
}

type BoxList []Box

func (b Box) Volumn() float64 {
    return b.width * b.height * b.depth
}

func (b *Box) SetColor(c Color) {
    b.color = c
}

func (bl BoxList) BiggestColor() Color {
    v := 0.00
    k := Color(WHITE)
    for _, box := range bl {
        if bv := box.Volumn(); bv > v {
            v = bv
            k = box.color
        }
    }

    return k
}

func (bl BoxList) PaintBlack() {
    for i := range bl {
        bl[i].SetColor(BLACK)
    }
}

func (c Color) String() string {
    strings := []string{"WHITE", "BLACK", "BLUE", "RED", "YELLOW"}
    return strings[c]
}


```


2 方法的继承

```
type Skills []string

type Human struct {
    name   string
    age    int
    weight int
}

type Student struct {
    Human
    Skills
    int
    weight     int
    speciality string
}

func (h *Human) SayHi() {
    fmt.Printf("i am %s, %d this year\n", h.name, h.age)
}



```


3 接口

```

//接口定义了方法签名，用法类似于java的接口，可实现多态
type Abser interface {
    Abs() float64
}

// *Vertex 实现了该接口，无需显示声明implements
func (v *Vertex) Abs() float64 {
    return math.Sqrt(v.X * v.X + v.Y * v.Y)
}



//接口值，保存底层类型的值，调用方法时执行底层类型的同名方法(多态)
type I interface {
    M()
}

type T struct {
    S string
}

type F float64

func (t *T) M() {
    fmt.Println(t.S)
}

func (f F) M() {
    fmt.Println(f)
}

//接口值(value, type)元祖，使用describe函数打印接口值
//&{%!V(string=hello)} -- *main.T
//%!V(main.F=3.141592653589793) -- main.F
func main() {
    var i I

    i = &T{"hello"}
    describe(i)
    i.M()

    i = F(math.Pi)
    describe(i)
    i.M()

}

func describe(i I) {
    fmt.Printf("%V -- %T\n", i, i)
}


//优雅的处理空指针,否则会抛出：runtime error: invalid memory address or nil pointer dereference
func (t *T) M() {
    if t == nil {
        fmt.Println("<nil>")
        return
    }
    fmt.Println(t.S)
}

//空接口
func main() {
    var i interface{}
    describe(i)
    
    //空接口可以保持任何类型的值
    i = 42
    describe(i)

    i = "hello"
    describe(i)
}

func describe(i interface{}) {
    fmt.Printf("%V -- %T\n", i, i)
}


```

3 类型断言

```

//断言实现方式类似于映射
func main(){
    var i interface{} = "hello"

    s := i.(string)
    fmt.Println(s)

    s, ok := i.(string)
    fmt.Println(s, ok)

    f, ok := i.(float64)
    fmt.Println(f, ok)

    //这里可能抛出error
    f = i.(float64)
    fmt.Println(f)
}


```

4 类型选择

```
//使用switch选择接口底层类型
func do(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Twice %v is %v\n", v, v * 2)
    case string:
        fmt.Printf("%q is %v bytes long\n", v, len(v))
    default:
        fmt.Printf("unkown type: %T!\n", v)
    }
}

func main(){
    do(23)
    do("hello")
    do(true)
}

```

5 接口使用实例

* fmt包的Stringer接口，定义了String签名方法, 使用fmt.Printfln时调用该方法
* 重新实现对象的String()方法，可实现个性化输出


godoc -http=:8088，查看fmt.Println源码

```

//1 define Stringer interface
type Stringer interface {
	String() string
}

//2 reciever any type by void interface
func Println(a ...interface{}) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}

//3 inner method will call String() or Error() to print 
func (p *pp) handleMethods(verb rune) (handled bool) {
    ...
    switch v := p.arg.(type) {
	case error:
	   handled = true
	   defer p.catchPanic(p.arg, verb)
	   p.fmtString(v.Error(), verb)
	   return

	case Stringer:
		handled = true
		defer p.catchPanic(p.arg, verb)
		p.fmtString(v.String(), verb)
		return
    }
}


```

```

type Person struct {
    Name string
    Age int
}

func (p Person) String() string {
    return fmt.Sprintf("%v(%v years)", p.Name, p.Age)
}

func main(){
    p := Person{"liam", 24}
    z := Person{"pony", 22}
    fmt.Println(p, z)
}

```

byte 数组实现Stringf 接口

```

func (ip IPAddress) String() string{
    s := ""
    for i, value := range ip {
        s = s + fmt.Sprintf("%d", value)
        if i < len(ip) - 1 {
            s = s + ":"
        }
    }

    return s
}

func main(){
    
    
    hosts := map[string] IPAddress {
        "loopback" : {127, 0, 0, 1},
        "googleDNS" : {8,8,8,8},
    }

    for name, ip := range hosts {
        fmt.Printf("%v : %v\n", name, ip)
    }
}

```
