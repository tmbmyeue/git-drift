# git alias

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit

# more useful case
git config --global alias.last 'log -1 HEAD'
git config --global alias.unstage 'reset HEAD --' 
# which will mean
git unstage fileA
git reset HEAD -- fileA
```

# git branch

```bash
# watch which the branch point to 
git log --oneline --decorate
    f30ab (HEAD, master, testing) add feature #32 - ability to add new
    34ac2 fixed bug #1328 - stack overflow under certain conditions
    98ca9 initial commit of my project

```

`git`的`blob`对象保存文件快照，`tree`对象记录了目录结构和`blob`对象索引，`commit`对象包括了指向`tree`对象的指针和所有的提交信息。当创建`branch`时本质上是创建一个包含所指对象的校验和（长度为40的SHA-1值字符串）的文件（附加一个换行），所以速度很快。这种方式也让回溯共同祖先变得很简单。

##  场景专项问题解决

为一个完整版本的项目做一次`commit`，当有并行的目标或需求需要实现的时候，就可以新建`branch`来并行解决，当他们都解决完后，可以用merge将其合并。

```bash
git checkout -b branchname
# same as git branch branchname and git checkout it 
```

```bash
$ git ls-remote
From git@github.com:tmbmyeue/git-drift.git
c01ac73fe700a1c1ba360d3738f2cb16ee136c60        HEAD
c01ac73fe700a1c1ba360d3738f2cb16ee136c60        refs/heads/master

```

![123](D:\科研中心\git-drift\note-md\assets\remote-branches-2.png)

![`git fetch` æ´æ°ä½ çè¿ç¨ä»åºå¼ç¨ã](assets/remote-branches-3.png)





# some cases I cannot understand

## how to delete the space git has convered

1. [记一次删除Git记录中的大文件的过程](https://www.hollischuang.com/archives/1708)
2. 

```bash
作者：Intopass
链接：https://www.zhihu.com/question/29769130/answer/45546231
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

二：高级办法
注意高级办法会导致push冲突，需要强制提交，其他人pull也会遇到冲突，建议重新克隆。
！！！注意这些操作都很危险，建议找个示例库进行测试，确保自己完全掌握之后再实际操作。

1.完全重建版本库
$ rm -rf .git
$ git init
$ git add .
$ git cm "first commit"
$ git remote add origin <your_github_repo_url>
$ git push -f -u origin master

2.有选择性的合并历史提交
$ git rebase -i <first_commit>

会进入一个如下所示的文件
  1 pick ba07c7d add bootstrap theme and format import
  2 pick 7d905b8 add newline at file last line
  3 pick 037313c fn up_first_char rename to caps
  4 pick 34e647e add fn of && use for index.jsp
  5 pick 0175f03 rename common include
  6 pick 7f3f665 update group name && update config

将想合并的提交的pick改成s，如
  1 pick ba07c7d add bootstrap theme and format import
  2 pick 7d905b8 add newline at file last line
  3 pick 037313c fn up_first_char rename to caps
  4 s 34e647e add fn of && use for index.jsp
  5 pick 0175f03 rename common include
  6 pick 7f3f665 update group name && update config

这样第四个提交就会合并进入第三个提交。
等合并完提交之后再运行
$ git push -f
$ git gc --prune=now
```

