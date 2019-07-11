## Valid Palindrome


#### 问题

```

Given a string, determine if it is a palindrome, considering only alphanumeric characters and ignoring cases.

Note: For the purpose of this problem, we define empty string as valid palindrome.

Example 1:

Input: "A man, a plan, a canal: Panama"
Output: true

Example 2:

Input: "race a car"
Output: false

```

### 思路

* 准备2个指针，同时从头尾迭代比较
* 遇到非数字和字母则跳过
* 比较头尾指针字符

### 实现

```

class Solution {
    public boolean isPalindrome(String s) {
        if (s.length() == 0){
            return true;
        }

        int leftToRight = 0;
        int rightToLeft = s.length() - 1;

        while (leftToRight != rightToLeft){

            if (!Character.isLetterOrDigit(s.charAt(leftToRight))){
                leftToRight++;
                continue;
            }

            if (!Character.isLetterOrDigit(s.charAt(rightToLeft))){
                rightToLeft--;
                continue;
            }

            if (Character.toLowerCase(s.charAt(leftToRight)) != Character.toLowerCase(s.charAt(rightToLeft))){
                return false;
            }

            leftToRight++;
            rightToLeft--;

            if (leftToRight > rightToLeft){
                break;
            }
        }

        return true;
    }
}


```