## Reader

1 Reader

* io.Reader接口下的方法，读取到切片并返回读取的字节数和错误
* 当读取到数据流末尾时，它会返回一个EOF的错误

func (T) Read([]byte) (n int, err error)


```
func main(){

    r := strings.NewReader("hello, reader!")

    b := make([]byte, 8)
    for {
        n, err := r.Read(b)
        fmt.Printf("n = %v, error = %v, b = %v\n", n, err, b)
        fmt.Printf("b[:n] = %q\n", b[:n])

        if err == io.EOF {
            break
        }
    }
}


//rot13算法：实现Reader并包含一个Reader，实现rot13算法，类似于包装器
func (rot13 *rot13Reader) Read(b []byte) (n int, err error){

    m := make(map[byte]byte)
    input := "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
    output := "NOPQRSTUVWXYZABCDEFGHIJKLMnopqrstuvwxyzabcdefghijklm"
    for idx := range input {
        m[input[idx]] = output[idx] 
    }

    n, err = rot13.r.Read(b)
    for i := 0; i < n; i++ {
        if val, ok := map[b[i]] {
            b[i] = val
        }
    }

    return n, nil
}


```
