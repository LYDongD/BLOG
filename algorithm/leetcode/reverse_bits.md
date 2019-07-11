## Reverse Bits

### 问题

```
Reverse bits of a given 32 bits unsigned integer.

Example:

Input: 43261596
Output: 964176192
Explanation: 43261596 represented in binary as 00000010100101000001111010011100, 
             return 964176192 represented in binary as 00111001011110000010100101000000.
Follow up:
If this function is called many times, how would you optimize it?

```

### 思路

* 将位的倒置转化为位移运算
* 从低到高位不断取出最后一位
* 结果从0开始不断左移，如果最后一位是1，则+1


```
public int reverseBits(int n) {
        int result = 0;
        for (int i = 0; i < 32; i++){
            //get the last bit
            int lastBit = n & 1;

            //fill the left bit
            if (lastBit == 1){
                result = (result << 1) + 1;
            }else {
                result = result << 1;
            }

            //handle next last bit
            n = n >> 1;
        }

        return result;
    }
}

```

