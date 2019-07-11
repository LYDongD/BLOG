## 结构体

#### 使用struct实现类型的继承和重载

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

func main() {

    //inherit
    mark := Student{Human: Human{"mark", 22, 100}, weight: 120, speciality: "math"}
    fmt.Println(mark.name)
    fmt.Println(mark.age)
    //override
    fmt.Println(mark.Human.weight)
    fmt.Println(mark.weight)
    fmt.Println(mark.speciality)

    mark.Skills = []string{"music", "basketball"}
    fmt.Println(mark.Skills)
    mark.Skills = append(mark.Skills, "speech", "golang")
    fmt.Println(mark.Skills)

    mark.int = 3
    fmt.Println(mark.int)
}

```
