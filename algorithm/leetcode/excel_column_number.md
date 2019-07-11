## Excel Sheet Column Number

### 问题

```
Given a column title as appear in an Excel sheet, return its corresponding column number.

For example:

    A -> 1
    B -> 2
    C -> 3
    ...
    Z -> 26
    AA -> 27
    AB -> 28 
    ...
Example 1:

Input: "A"
Output: 1

Example 2:

Input: "AB"
Output: 28

Example 3:

Input: "ZY"
Output: 701

```

### 思路

* 字符排列规律等价于26进制
* 26进账转10进制

```
public static int titleToNumber(String s) {

    int result = 0;
    for (char character : s.toCharArray()){
        int characterIndex = character - 'A' + 1;
        result = result * 26 + characterIndex;
    }

    return result;
}

```