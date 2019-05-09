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

# git merge

## merge tool : beyond compare

```bash
--- BEGIN LICENSE KEY ---  
H1bJTd2SauPv5Garuaq0Ig43uqq5NJOEw94wxdZTpU-pFB9GmyPk677gJ  
vC1Ro6sbAvKR4pVwtxdCfuoZDb6hJ5bVQKqlfihJfSYZt-xVrVU27+0Ja  
hFbqTmYskatMTgPyjvv99CF2Te8ec+Ys2SPxyZAF0YwOCNOWmsyqN5y9t  
q2Kw2pjoiDs5gIH-uw5U49JzOB6otS7kThBJE-H9A76u4uUvR8DKb+VcB  
rWu5qSJGEnbsXNfJdq5L2D8QgRdV-sXHp2A-7j1X2n4WIISvU1V9koIyS  
NisHFBTcWJS0sC5BTFwrtfLEE9lEwz2bxHQpWJiu12ZeKpi+7oUSqebX+  
--- END LICENSE KEY -----
```

## git ssh

[教程](https://help.github.com/en/articles/using-ssh-over-the-https-port)

#  场景专项问题解决

##  为突然出现的需求新开branch

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

## git push 冲突

```bash
# make mergetool not to create orig file
git config --global mergetool.keepBackup false
```

```bash
$ git push origin master
To github.com:tmbmyeue/git-drift.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@github.com:tmbmyeue/git-drift.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

$ git pull origin master
remote: Enumerating objects: 7, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 4 (delta 1), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (4/4), done.
From github.com:tmbmyeue/git-drift
 * branch            master     -> FETCH_HEAD
   84364aa..1c033a7  master     -> origin/master
error: Your local changes to the following files would be overwritten by merge:
        note-md/test.md
Please commit your changes or stash them before you merge.
Aborting
Updating 84364aa..1c033a7

$ git merge origin/master
$ git mergetool

$ git push origin master
Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 4 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (8/8), 681 bytes | 170.00 KiB/s, done.
Total 8 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 1 local object.
To github.com:tmbmyeue/git-drift.git
   1c033a7..314ccaa  master -> master

```





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

