## 目录操作

#### 罗列目录

```
# $NF -> tail column
ls -l | grep "^d" | awk '{print $NF}'

# find
find ./ -type d -maxdepth 1 -print

# others
ls -d */
ls -F | grep /


#print directory tree
tree ./ -H http://localhost -o out.html

```


#### 目录切换

1 多目录切换

```
# use pushd/popd stack
pushd /home/liam
cd /home/liam

pushd /etc/nginx
cd /etc/nginx

pushd /root/shell

#check dirs stack
dirs

#switch to /home/liam
pushd +3 

```

2 双目录切换

```
# switch back -> cd -
cd /root/shell
cd-

```
