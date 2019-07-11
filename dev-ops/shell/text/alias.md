## 别名alias

> 定义别名

1 开启shell时使别名脚本生效

```
vim ~/.bash_profile
source ~/.custom_alias

```

2 在~/.custom_alias中定义别名即可

* 以函数的方式定义
* alias关键字定义

```
#以函数的方式定义别名
#进入目录并展示内容
function cdd() {
    cd $1
    ls
}

#git提交到远程仓库
function gitpush() {
    echo $1
    B=$(git branch | sed -n -e 's/^\*\(.*\)/\1/p')
    git add -A .
    git commit -m $1
    git push -u origin $B
}

#google
function google() {
    open -na "Google Chrome" --args "https://www.google.com/search?q=$*"
}

function googlestack() {
    open -na "Google Chrome" --args "https://www.google.com/search?q=site:stackoverflow.com $*"
}

#desktop
function desktop() {
    cd ~/Desktop
}

function shortcuts() {
    cat ~/.custom_aliases # the file with your aliases
}

#直接定义别名
alias grun='java org.antlr.v4.gui.TestRig'

```
