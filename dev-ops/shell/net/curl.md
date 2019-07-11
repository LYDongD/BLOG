## curl

常用参数

* -X [method] : 请求方法
* -H [header] : 指定请求头
* -o [file] : redirect 到文件
* -I : 仅返回响应头信息

```

//request for put and specific header
curl -X PUT "localhost:9200/twitter/_doc/1" -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'

//only print header
curl -I www.baidu.com

//redirect
curl https://raw.githubusercontent.com/LYDongD/graphic/master/markdown/producer.png -o producer.png

```

> 追踪网站变化

```
#!/bin/bash
  
#param check 
if [ $# -ne 1 ]; then
    echo "$0 needs 1 param: URL\n";
    exit -1;
fi

first_time=0

#if last.html do not exist, saying first_time = true
if [ ! -e "last.html" ]; then
    first_time=1
fi

curl --silent $1 -o recent.html

if [ $first_time -ne 1 ]; then
    changes=$(diff -u last.html recent.html)
    if [ -n "$changes" ]; then
        echo -e "Changes:\n"
        echo "$changes"
    else
        echo -e "\nno changes"
    fi
else
    echo "first time archiving..."
fi

cp recent.html last.html

```
