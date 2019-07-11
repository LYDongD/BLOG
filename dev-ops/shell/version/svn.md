## svn 版本管理

#### svn flow, 基于目录管理

* trunk
* branch
* tag

由于svn不存在本地仓库，所有操作都是基于远程集中仓库，需要先提交后操作

```

svn info 查看仓库地址：http://liyuedong@200.200.200.40/svn/repositories/server/code/clife-V4/clife-bigdata

1 创建branch分支

svn cp http://liyuedong@200.200.200.40/svn/repositories/server/code/clife-V4/clife-bigdata/trunk  http://liyuedong@200.200.200.40/svn/repositories/server/code/clife-V4/clife-bigdata/branches/test-branch -m “create branch“

2 拉取分支

svn checkout http://liyuedong@200.200.200.40/svn/repositories/server/code/clife-V4/clife-bigdata/branches/test-branch 
或
如果svn管理整个trunk,branch和tag，则svn update

3 合并

进入待并入项目
svn merge http://liyuedong@200.200.200.40/svn/repositories/server/code/clife-V4/clife-bigdata/trunk

4 查看版本差异

svn status
svn diff


5 查看某个分支是谁提交的

svn info xxxx

6 查看某个分支的提交记录

//最近5条
svn log -l 5 xxx

```


#### svn分支合并

```
1 合并branch到trunk

先查看branch的起始版本，进入trunk目录，将起始版本到当前版本的改动合并到trunk:

svn merge -r243944:HEAD http://liyuedong@200.200.200.40/svn/repositories/server/code/clife-V4/clife-bigdata/clife-bigdata-business/clife-bigdata-business-scene/branches/status-query-support
svn ci -m 'merget v2.4.0 to trunk'

```
