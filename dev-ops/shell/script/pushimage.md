## 推送图片到github，并生成图片地址 

```
#!/bin/bash

img=$1

mv $img ~/Desktop/plantuml/markdown/

cd ~/Desktop/plantuml/markdown/

if [ $# -ne 1 ]; then
    echo "$0 needs image name as param"
    exit -1
fi


#push img
git add $img
git commit -m 'add new img'
git push origin master


#return img url
baseUrl="https://raw.githubusercontent.com/LYDongD/graphic/master/markdown/"
url=$baseUrl$img
echo $url

```
