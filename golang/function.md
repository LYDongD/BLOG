## 函数

1 函数作为一种类型，可实现值传递

```
//define function type
type testInt func(a int) bool

//type implementation
func isOdd(a int) bool {
    if a%2 == 0 {
        return false
    }

    return true
}

func isEven(a int) bool {
    if a % 2 == 0 {
        return true
    }

    return false
}

//use func type as parameter
func filter(nums []int, f testInt) []int {

    var result []int
    if len(nums) == 0 {
        return result
    }

    for _, num := range nums {
        if f(num) {
            result = append(result, num)
        }
    }

    return result
}

func main() {

    nums := []int{1, 2, 3, 4, 5, 7}
    oddNums := filter(nums, isOdd)
    fmt.Println("odd nums: ", oddNums)

    evenNums := filter(nums, isEven)
    fmt.Println("even nums: ", evenNums)

}

```
