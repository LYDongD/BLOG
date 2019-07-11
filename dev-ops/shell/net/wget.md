## wget

#### wget和curl的区别

* wget下载并将字节流写入文件
* curl下载并将字节流写入stdout，需要重定向到指定文件

#### 操作

```
//重命名文件
wget http://www.baidu.com -O baidu.html

//抓取整个网站
wget --mirror www.ponymew.com

//递归
wget -r -N -l 2 http://www.baidu.com

//限速
wget --limit-rate 1k http://www.baidu.com

```


