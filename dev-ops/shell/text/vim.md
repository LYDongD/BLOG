## vim

### 跳转和切换

```

//行跳转
gg  跳转到首行
shift go  跳转至尾行
:set number 显示行号
:set nonumber 不显示行号
12 shift go: 跳转至12行
                

//缩进设置
:set ai
:set no ai 
:set shiftwidth=4 //缩进宽度


//翻页
ctrl f: pagedown
ctrl b: pageup


//前后台切换

方法1:
ctrl z
fg

方法2:
:sh
exit


//光标移动图谱：操作单位 + 操作数量
字符（h、l）→ 
单词 (w、W、b、B、e) → 
行 (j、k、0、^、$、:n) → 
句子（(、)）→ 
段落（{、}）→ 
屏 (H、M、L) → 
页（Ctrl-f、Ctrl-b、Ctrl-u、Ctrl-d) → 
文件（G、gg、:0、:$)
花括号检测和跳转 %


//显示历史命令
q:

``` 

### 文本编辑

```

字符 （x、c、s、r、i、a）→ 
单词 (cw、cW、cb、cB、dw、dW、db、dB) → 
行 (dd、d0、d$、I、A、o、O) → 
句子（(、)）→ 
段落（{、}）


//撤销与恢复
U
ctrl +  r


//缩进
>>
<<


```


### 多行操作

```
//多行插入字符
1 ctrl v
2 选中多行
3 I（大写)
4 输入插入字符
5 esc esc (连续输入两个)



//删除全文
1 gg 跳转至首行
2 dG 删除当前游标下所有行

```

### 删除

* 删除当前行光标之后的字符： d$
 

### 搜索与替换

```
//全文替换
:%s/old/new/g

//单行替换
:s/old/new/g

```

### 高亮和历史

```

//取消高亮
:syntax clear
或:noh

//查看历史
q:

```


### 多文件操作

```
#打开多个文件
vim -o/O file1 file2

#如果已经打开了一个文件，想打开另一个文件
:vs filename
:sp filename

#窗口切换
ctrl w

```

### 插件

1 golang自动补全

```
//下载vim-go
...

//开启或关闭自动补全
Ctrl x/o

//提示
Ctrl n/p

```

### 配置大全

```
" 缩进
set smartindent " 基于autoindent的一些改进
set ts=4    " set 4 space for a tab
set sw=4    " 使用每层缩进的空格数
set softtabstop " 开启了et后使用退格，每次退格删除X个空格
set expandtab  " 换行自动缩进对齐



```
