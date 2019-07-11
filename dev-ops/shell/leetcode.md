## leetcode-cli usage

参考(https://github.com/skygragon/leetcode-cli)


* 常用命令

```
//login
leetcode user -l 

//check problem list
leetcode list 
    
//check problem
leetcode show 443
leetcode show -c 443

//submit:file name shoud be the same as problem name which can check by cache
leetcode cache 
leetcode submit [file_path] 

```

* 一行代码查看最近一道未完成的题目

```
//check latest easy undone problem
leetcode list -q eD | sort -k 1 | awk '{print $1}' | sed 's/\[\(.*\)\]/\1/g' | head -n 1 | xargs leetcode show

```
