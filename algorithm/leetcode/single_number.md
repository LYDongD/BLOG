## Single Number

### 问题

```
Given a non-empty array of integers, every element appears twice except for one. Find that single one.

Note:

Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

Example 1:

Input: [2,2,1]
Output: 1

Example 2:

Input: [4,1,2,1,2]
Output: 4

```


### 思路

利用XOR异或运算的性质：

* 交换律
* 结合律
* X ^ X = 0, X ^ 0 = X

一直进行XOR运算，运用交换律和结合律，相同数值结合为0，最后剩下的数即为single number

### 代码

```
class Solution {
    public int singleNumber(int[] nums) {
        int result = 0;
        for (int i = 0; i < nums.length; i++){
            result = result ^ nums[i];
        }
        return result;
    }
}


```

