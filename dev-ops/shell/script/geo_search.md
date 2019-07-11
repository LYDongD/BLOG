## 分析一段基于地理位置的查询附近柜机的脚本

查询附近柜机，单位km

```

#入参
lat="$1"
lon="$2"
range="\"$3\""
pageNo="$4"
pageSize="$5"

#参数校验
if [ x"$lat" == x"" ]
then
        log "lat can't be blank."
        exit -1
fi

if [ x"$lon" == x"" ]
then
        log "lon can't be blank."
        exit -1
fi

if [ x"$3" == x"" ]
then
        log "range can't be blank."
        exit -1
fi

if [ x"$pageNo" == x"" ]
then
        log "pageNo not set, default was 0."
        pageNo=0
fi

if [ x"$pageSize" == x"" ]
then
        log "pageSize not set, default was 100"
fi

#封装日志打印函数
function log() {
    echo `date '+%Y-%m-%d %H:%M:%S'` "$*"
}

log "./search_nearby_cabinet_in_es.sh" "$*"

#查询参数，涉及排序，查询，source过滤返回字段script_fields和生成脚本字段
json='{
        "from": '"${pageNo}"',
        "size": '"${pageSize}"',
        "sort": [ 
    {
      "_geo_distance": {
        "unit": "km",
        "order": "asc",
        "geo": [
          '"${lon}"',
          '"${lat}"'
        ],
        "distance_type": "sloppy_arc",
        "mode": "min"
      }
    }
    ],
    "_source": ["cabinetCode", "throwAddress"],
    "query": {
                "geo_distance": {
                  "distance": '"${range}"',
                  "geo": {
                                "lat": '"${lat}"',
                                "lon": '"${lon}"'
                  }
    }
    },
    "script_fields" : {
                "distance" : {
                  "script" : {
                        "inline": "doc[\u0027geo\u0027].arcDistance(params.lat,params.lon) * 0.001",
                        "lang": "painless",
                        "params": {
                                "lat": '"${lat}"',
                                "lon": '"${lon}"'
                        }
                   }
                 }
    }
}'

#打印参数
log "$json"

#请求es，查询
curl -o result.txt -XPOST -H "Content-Type:application/json" "http://10.204.1.65:9200/cabinetbase/edmedinfo/_search?pretty=1" -d"${json}"


```

### 1 函数的定义和参数

* function [func_name]( ) {}
* 取外部参数：$1 $2…$*(以一个单字符串显示所有向脚本传递的参数)\
* echo `xxxx` 执行一个命令并打印，不加`` 只会打印命令本身

```
function log() {
    echo `date '+%Y-%m-%d %H:%M:%S'` "$*"
}

```

### 2 参数变量的字符串比较使用前缀x

[参考](https://blog.csdn.net/JCRunner/article/details/51565212#%E4%B8%8D%E4%BD%BF%E7%94%A8%E5%8F%8C%E5%BC%95%E5%8F%B7%E4%BD%BF%E7%94%A8%E5%89%8D%E7%BC%80x)

```
if [ x"$3" == x"" ]
then
        log "range can't be blank."
        exit -1
fi

```

### 3 curl 命令的使用

[参考](https://blog.ponymew.com/dev-ops/shell/wang-luo-gong-ju/curl	)

```
curl -o result.txt -XPOST -H "Content-Type:	/json" "http://10.204.1.65:9200/cabinetbase/edmedinfo/_search?pretty=1" -d"${json}"

```

### 4 es 查询api

[地理位置排序](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-sort.html)
[脚本生成字段](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-scripting-using.html)
[_source用法](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-source-filtering.html)
[地理位置查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-distance-query.html)
