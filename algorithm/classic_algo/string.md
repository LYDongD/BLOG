## 字符串匹配

> BF

```
| BF -> brute force
	| 从左往右迭代主串，标记起始索引，截取和模式串等长的子串
	| 子串和模式串比较，相同，则返回当前起始索引
	| 如果到最后一个起始索引都没有找到匹配的子串，则返回-1
	| 最后一个起始索引为 n - m , 迭代次数为n - m +  1

	| complexity
		| time: O(n*m)


```

```
//BF search pattern index, return the first match subs start index
func bfSearch(main string, pattern string) int {

    //defensive
    if len(main) == 0 || len(pattern) == 0 || len(main) < len(pattern) {
        return -1
    }

    for i := 0; i <= len(main)-len(pattern); i++ {
        subStr := main[i : i+len(pattern)]
        if subStr == pattern {
            return i
        }
    }

    return -1
}

```

> RK

```
| RK 
	| O(n) char compare  -> hash code compare
		| hash code compute
			| k-进制法 
				| 无冲突算法：k进制的10进制
					| 只需要计算一个子串O(n)，其他子串可由上一个子串推导
				| 有冲突算法
					| hash相等之后，再进行字符比较


```

> BM

```
| BM
	| optimization thinking
		| add moving steps per time
	| steps moving principle
		| 坏字符原则 -> 确定坏字符在模式串中的位置
			| 从主串中截取模式串长度的子串
			| 从后往前比较子串和模式串
			| 坏字符在主串中对应模式串的位置 j,  坏字符在模式串中的位置k
			| 移动的位置为 j - k , 如果k不存在，则为-1
			| 直到主串来到最后一个合法的起始位置 n - m
		| 好后缀原则
			| 从后往前比较主串和模式串，找出坏字符索引，好后缀的起始位为j + 1
			| 在模式串中查找好后缀 
				| 找到好后缀(好后缀完全重合)，起始位为k, 则移动j + 1 - k 
				| 找不到好后缀
					| 模式前缀与好后缀的子后缀匹配(好后缀部分重合)， 假设部分重合起始位为v, 则移动  v
					| 好后缀完全不重合，移动m

			| pattern suffix handle
				| suffix[] -> suffix length — match substr start index
				| prfix[] -> suffix length - if prefix exist

				| default suffix[] : -1
				| i : 0 -> i
					| j:  j = i ->0
					| k: 0 -> length
						| p[j]  == p[m-1-k] 
						 | j - - , k++ 
					| j == -1 -> prefix = true


```

```
  
import (
    "fmt"
    "math"
)

//bc: pattern char index hash mapping
func generateBC(pattern string) []int {

    bc := make([]int, 256)

    for index := range bc {
        bc[index] = -1
    }

    for index, char := range pattern {
        bc[int(char)] = index
    }

    return bc
}

//generate suffix and prefix array for pattern
func generateGS(pattern string) ([]int, []bool) {
    m := len(pattern)
    suffix := make([]int, m)
    prefix := make([]bool, m)

    //init
    for i := 0; i < m; i++ {
        suffix[i] = -1
        prefix[i] = false
    }

    for i := 0; i < m-1; i++ {
        j := i
        k := 0
        for j >= 0 && pattern[j] == pattern[m-1-k] {
            j--
            k++
            suffix[k] = j + 1
        }

        if j == -1 {
            prefix[k] = true
        }
    }

    return suffix, prefix
}

//todo
func moveByGS(patternLength int, badCharStartIndex int, suffix []int, prefix []bool) int {

    //length of good suffix
    k := patternLength - badCharStartIndex - 1

    //complete match
    if suffix[k] != -1 {
        return badCharStartIndex + 1 - suffix[k]
    }

    //partial match
    for t := patternLength - 1; t > badCharStartIndex+1; t-- {
        if prefix[t] {
            return t
        }
    }

    //no match
    return patternLength

}

func bmSearch(main string, pattern string) int {
    //defensive
    if len(main) == 0 || len(pattern) == 0 || len(pattern) > len(main) {
        return -1
    }

    bc := generateBC(pattern)
    suffix, prefix := generateGS(pattern)

    n := len(main)
    m := len(pattern)

    // i : start index of main string
    step := 1
    for i := 0; i <= n-m; i = i + step {
        subStr := main[i : i+m]
        k, j := findBadChar(subStr, pattern, bc)

        stepForBC := j - k
        //j is bad char occur index
        if j == -1 {
            return i
        }

        stepForGS := -1
        if j < m-1 {
            stepForGS = moveByGS(m, j, suffix, prefix)
        }

        //k is bad char index in pattern
        step = int(math.Max(float64(stepForBC), float64(stepForGS)))
    }

    return -1
}

func findBadChar(subStr string, pattern string, bc []int) (int, int) {

    j := -1
    k := -1
    badChar := rune(0)

    for index := len(subStr) - 1; index >= 0; index-- {
        if subStr[index] != pattern[index] {
            j = index
            badChar = rune(subStr[index])
            break
        }
    }

    //if bad character exist, then find it's index at pattern
    if j > 0 {
        k = bc[int(badChar)]
    }

    return k, j
}


```
