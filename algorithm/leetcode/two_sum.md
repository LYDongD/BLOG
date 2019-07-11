##  Two Sum II - Input array is sorted

### 问题

```

Given an array of integers that is already sorted in ascending order, find two numbers such that they add up to a specific target number.

The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2.

Note:

Your returned answers (both index1 and index2) are not zero-based.
You may assume that each input would have exactly one solution and you may not use the same element twice.
Example:

Input: numbers = [2,7,11,15], target = 9
Output: [1,2]
Explanation: The sum of 2 and 7 is 9. Therefore index1 = 1, index2 = 2.

```
### 思路

1. 两个指针分别从头和尾往中间走
2. 如果和比目标小，走左边指针
3. 如果和笔目标大，走右边指针
4. 与目标相当或两指针汇合则退出


```

public int[] twoSum(int[] numbers, int target) {
        
        if (numbers == null) return null;

        int low = 0;
        int high = numbers.length - 1;
        
        while (low != high) {
            if (numbers[low] + numbers[high] < target) {
                low++;
            } else if (numbers[low] + numbers[high] > target) {
                high--;
            } else {
                return new int[]{low + 1, high + 1};
            }
        }

        return null;
    }
```
