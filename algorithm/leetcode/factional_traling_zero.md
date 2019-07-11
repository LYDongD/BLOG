## Factorial Trailing Zeroes

### 问题

```
Given an integer n, return the number of trailing zeroes in n!.

Example 1:

Input: 3
Output: 0
Explanation: 3! = 6, no trailing zero.
Example 2:

Input: 5
Output: 1
Explanation: 5! = 120, one trailing zero.
Note: Your solution should be in logarithmic time complexity.

```

### 思路

由于阶层数极大，如果先计算阶层会导致内存溢出

* 分解因子，求 0 = 2 * 5 分组的个数
* 对于阶层而言，如果包含因子5，总是能找到因子2与之相乘；
* 求因子5的个数
* 因子5的个数 = n / 5 + n / 25 + n / 125 + ...
	* 每5个数产生一个因子5
	* 每25个数额外产生一个因子5(因为25 = 5 * 5)
	* 每125又再额外产生一个因子5(因为125 = 25 * 5)
	* 以此类推：因子5的个数 = n / 5 + n / 25 + n / 125 + ...

```

public static int trailingZeroes(int n) {

    int count = 0;
    while (n > 0){
        count += n / 5;
        n = n / 5;
    }

    return count;
}


```