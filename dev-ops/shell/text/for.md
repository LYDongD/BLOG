## 循环控制

#### 循环匹配读取外部参数

* shift 将$2, $3等匹配到的变量赋值给$1
* case var in pattern1) action ;; .... ;; esac 匹配 -> 执行 -> break
 

```
if [ $# -ne 3 ];
then
    echo "Usage: $0 URL -d DIRECTORY"
    exit -1
fi


#param parse
for i in {1..4}
do
    case $1 in
    -d)
       shift; #if match -d, then shift to next param to set dir
       directory=$1;
       shift; #if match dir, shift to next param to set url
       ;; # break

    *) #default
       url=${url:-$1};
       shift;
       ;;
    esac
done


mkdir -p $directory
baseurl=$(echo $url | egrep -o "https?://[a-z.]+")
echo "url: ${url}"
echo "baseUrl: ${baseurl}"


```
