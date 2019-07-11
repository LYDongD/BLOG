## 反射

* 反射是运行时能够检查并修改对象状态的一种机制
    * 检查属性、方法、类型等
    * 构建对象
    * 修复属性、方法等
    * 调用方法


```
  
import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.27
    v := reflect.ValueOf(x)
    fmt.Println("type: ", v.Type())
    fmt.Println("kind: ", v.Kind())
    fmt.Println("value: ", v.Float())

    p := reflect.ValueOf(&x)
    v = p.Elem()
    v.SetFloat(7.1)
    fmt.Println("value: ", v.Float())

}


```
