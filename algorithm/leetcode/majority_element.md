## Majority Element

### 问题

```

Given an array of size n, find the majority element. The majority element is the element that appears more than ⌊ n/2 ⌋ times.

You may assume that the array is non-empty and the majority element always exist in the array.

Example 1:

Input: [3,2,3]
Output: 3
Example 2:

Input: [2,2,1,1,1,2,2]
Output: 2

```
### 思路1

* 使用map统计元素出现的次数
* 超过一半时返回该元素

该算法需要耗费额外的存储空间

### 思路2

* 主要元素超过了一半，看成1
* 其他元素不足一半，可看成-1
* 如果相同+1，不同-1，剩下的元素是主要元素

```

 public int majorityElement(int[] nums) {

        int count = 0;
        int majority = 0;
        for (int num : nums) {
            if (count == 0) {
                majority = num;
                count++;
                continue;
            }

            if (majority == num) {
                count++;
            } else {
                count--;
            }
        }

        return majority;
    }

```