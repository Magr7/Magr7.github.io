---
title: Git飞行日记
---
我是优秀的文档搬运工～～～
###### 编辑提交(editting commits)
+ 我刚才提交了什么?
- 如果你用 git commit -a 提交了一次变化(changes)，而你又不确定到底这次提交了哪些内容。 你就可以用下面的命令显示当前HEAD上的最近一次的提交(commit):
  ``` bash
  (main)$ git show
  ```
- 或者
  ``` bash
  $ git log -n1 -p
  ```
- 我的提交信息(commit message)写错了
- 如果你的提交信息(commit message)写错了且这次提交(commit)还没有推(push), 你可以通过下面的方法来修改提交信息(commit message):
  ``` bash
  $ git commit --amend --only
  ```
- 这会打开你的默认编辑器, 在这里你可以编辑信息. 另一方面, 你也可以用一条命令一次完成:
``` bash
$ git commit --amend --only -m 'xxxxxxx'
```
- 如果你已经推(push)了这次提交(commit), 你可以修改这次提交(commit)然后强推(force push), 但是不推荐这么做。
- 我提交(commit)里的用户名和邮箱不对
- 如果这只是单个提交(commit)，修改它：
``` bash
$ git commit --amend --author "New Authorname <authoremail@mydomain.com>"
```
- 如果你需要修改所有历史, 参考 'git filter-branch'的指南页.
- 我想从一个提交(commit)里移除一个文件
- 通过下面的方法，从一个提交(commit)里移除一个文件:
``` bash
$ git checkout HEAD^ myfile
$ git add -A
$ git commit --amend
```
- 这将非常有用，当你有一个开放的补丁(open patch)，你往上面提交了一个不必要的文件，你需要强推(force push)去更新这个远程补丁。
  
More info: [Github](https://github.com/k88hudson/git-flight-rules)