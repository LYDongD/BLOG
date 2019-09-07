## 版本管理


### git flow

#### commit log 提交规范

使用git commit ，弹出vim编辑器，提交多行文本

```
head:   [type]: subject
body:   some description
footer:  some mark
```

* feat：新功能（feature）
* fix：修补bug
* docs：文档（documentation）
* style： 格式（不影响代码运行的变动）
* refactor：重构（即不是新增功能，也不是修改bug的代码变动）
* test：增加测试
* chore：构建过程或辅助工具的变动

设置git 提交模板

```
vim  ~/.gitconfig

[commit]
template = ~/.gitmessage

vim  ~/.gitmessage

设置模板如下：

# head: <type>(<scope>): <subject>
# - type: feat, fix, docs, style, refactor, test, chore
# - scope: can be empty (eg. if the change is a global or difficult to assign to a single component)
# - subject: start with verb (such as 'change'), 50-character line
#
# body: 72-character wrapped. This should answer:
# * Why was this change necessary?
# * How does it address the problem?
# * Are there any side effects?
#
# footer: 
# - Include a link to the ticket, if any.
# - BREAKING CHANGE
#

```

#### git conf

1 查看当前仓库或全局git配置

```
git config --local --list
git config --global --list

```

2 配置用户

```
git config user.name "xxx"
git config user.email "xxx"

```

3 关联远程分支

```
git branch --set-upstream-to origin/master

```

#### log

```
#查看某个分支的提交记录，以图形化方式，并省略其他信息（不加分支名默认查看当前分支，如需查看远程分支，需添加远程仓库名，例如默认的origin）
git log --oneline --graph master

##查看前5行的日志
git log --oneline --graph -n 5

```

#### 版本回退

```
#查看版本提交记录
git log -n 10 --oneline

#回退到指定版本
git reset --hard [版本id]

#回退到上一个版本
git reset -hard HEAD^

#如果想复原，git log已找不到先前的版本，则通过reflog查看操作日志，查找以前的commit id
git reflog

```

##### branch

```
#创建新的分支
git checkout -b test

#查看所有分支
git branch -av

#删除分支
git branch -d test （删除本地)
git push origin --delete test/checkPage2 (删除远程）

#拉取远程分支到一个新的本地分支，并自动切换
git checkout -b feature/CBS_V2.3_FAULT origin/feature/CBS_V2.3_FAULT

#对比分支差异，例如比较本地分支和远程分支的差异文件统计
git diff --stat release/CBS_V2.2 remotes/origin/master
git diff master remotes/origin/master

## 分支合并
git merge xxx

```

##### tag管理

```
#查看tag
git tag

#查看tag详情
git show [tag]

```

### 查看提交文件和内容

```
git log -n 10 --oneline
git show [commit version]

```
#### script 

mark 图片路径前缀

```
https://raw.githubusercontent.com/LYDongD/graphic/master/markdown/

```
--

## 如何用SSH登录github

> 1 生成一对公私钥，并保存到指定文件

ssh-keygen -t rsa

> 2 上传公钥至github

在个人设置菜单里

> 3 本地设置github站点需要校验的私钥文件地址

vim ~/.ssh/config

```
host github.com
HostName github.com
IdentityFile ~/tmp.txt
User git

```
