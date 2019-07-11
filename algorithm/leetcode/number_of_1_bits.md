## Number of 1 Bits

### 问题

```
Write a function that takes an unsigned integer and returns the number of '1' bits it has (also known as the Hamming weight).

Example 1:

Input: 11
Output: 3
Explanation: Integer 11 has binary representation 00000000000000000000000000001011 
Example 2:

Input: 128
Output: 1
Explanation: Integer 128 has binary representation 00000000000000000000000010000000

```

### 思路1

* 从低位到高位不断取最后一位
* 统计1的个数

该算法的复杂度是O(n)

```

 public int hammingWeight(int n) {
         int count = 0;
         while (n != 0) {
            if ((n & 1) == 1) {
                count++;
            }

            n = n >>> 1;
        }

        return count;

    }

```

### 思路2 

* 每次消除一个1，直到全部都是0
* n & (n - 1) 可消除最低位的1

该算法的平均复杂度为O(n/2)

```
 public int hammingWeight(int n) {
         int count = 0;
         while (n != 0) {
            count++;
            n = n & (n - 1);
        }

        return count;
    }

```

