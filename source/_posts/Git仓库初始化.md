---
title: Git仓库初始化
---
经常会面临当新建一个仓库怎么把已有项目的代码push过去，就忘记怎么搞了，这里记录一下。
+ 在命令行创建新的仓库
``` bash
echo "# seven" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M master
git remote add origin <你的仓库地址>
git push -u origin master
```
+ 从命令行推送现有存储库
``` bash
git remote add origin <你的仓库地址>
git branch -M master
git push -u origin master
```