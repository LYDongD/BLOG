## Image

1 Image

* image包定义了Image接口

```
type Image interface {
    ColorModel color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}

```

```

func main(){
    m := image.NewRGBA(image.Rect(0, 0, 100, 100))
    fmt.Println(m.Bounds())
    fmt.Println(m.At(0,0).RGBA())
}


//自定义Image实现Image接口

type Image struct {}

func (i Image) ColorModel() color.Model {
    return color.RGBAModel
}

func (i Image) Bounds() color.Rectangle {
    return image.Rect(0, 0, 200, 200)
}

func (i Image) At(x, y int) color.Color {
    return color.RGBA(uint8(x), uint8(), uint8(255), uint8(255))
}

```
