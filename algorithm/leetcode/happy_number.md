## Happy Number

### 问题

```

Write an algorithm to determine if a number is "happy".

A happy number is a number defined by the following process: Starting with any positive integer, replace the number by the sum of the squares of its digits, and repeat the process until the number equals 1 (where it will stay), or it loops endlessly in a cycle which does not include 1. Those numbers for which this process ends in 1 are happy numbers.

Example: 

Input: 19
Output: true
Explanation: 
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1


```

### 思路

* 开心数迭代更新若干次后，位平方和=1
* 不开心数迭代更新若干次后，位平方和值出现重复，形成loop回环
* 缓存出现过的和值，一旦出现重复，则判断为不开心数

```

boolean isHappy(int n) {
        Set<Integer> historyNum = new HashSet<>();
        while (true) {
            int sum = 0;
            while (n != 0) {
                sum += (n % 10) * (n % 10);
                n = n / 10;
            }

            if (sum == 1) {
                return true;
            }

            if (historyNum.contains(sum)) {
                return false;
            }

            historyNum.add(sum);
            n = sum;
        }
 }

```

### 数学证明

**为什么不开心数一定会形成loop**

* 和数是有界的

```
设该数为O1，有D位，假设O1为9999, 包含4位，位平方和未9^2 * D < 100D = 400, 结果比原数小。说明位平方和不会一直增加，一定会到达一个上界

```

* 由于和数是有界的，假设为N，那么a ∈ [2, n], 在有限范围内，离散地无限迭代，一定会发生重复，从而形成loop

