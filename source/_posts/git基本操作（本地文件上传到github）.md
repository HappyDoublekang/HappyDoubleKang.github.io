---
title: git基本操作（本地文件上传到github）
date: 2019-03-14 10:12:25
tags: [Git]
---

> clone项目到本地

使用Git的第一步永远是git clone，clone下来的文件夹名称默认为url最后一个斜杠后面的名称，如果不想用默认的名称，我们还可以通过在url后面传递参数来修改默认的名称，如下代码。

<!-- more -->

```javascript
git clone https://github.com/HappyDoublekang/VueProject.git
```

> 建立分支

一般线上的项目会有master和dev分支，而master分支上的东西是不允许随便乱动的，开发只在dev分支上，因此我们需要建立一个dev分支，并把远程仓库里的代码拉取到本地

* git checkout -b dev  （检查有没有dev这个分支）
* git pull origin dev  （提交到dev这个分支）

```javascript
git checkout -b dev
git pull origin dev
```

在切换好分支后，便可以进行正常的开发，调试等步骤

> 提交

在开发完毕，功能测试结束后，便要将本地代码提到远程仓库中

* git add -A  （全部选择）
* git commit -m "description" （暂存代码并添加代码描述）

```javascript
git add -A
git commit -m "description"
```

> 拉取并解决冲突

利用pull命令，从远程仓库中拉取最新的代码

* git pull origin dev  （从dev分支拉代码）

```javascript
git pull origin dev
```

此时可能会出现冲突，如果出现了冲突先解决冲突，在解决完冲突后，利用rebase命令提交修改冲突后的文件

* git add -A  （全部选择）
* git rebase --continue  （继续暂存）

```javascript
git add -A
git rebase --continue
```

> push至远程仓库

* git push origin dev  （正式推送到dev分支）

```javascript
git push origin dev
```

> 打tag发版本号

在代码提交至远程仓库并要上线时，首先需要将dev分支合并到master分支，然后在master分支上打tag标签，依次有以下一些步骤

* git checkout master  （切至主线分支）
* git merge dev  （合并dev分支）
* git tag -a v1.2.2.0 -m "v1.2.2.0"  （添加版本号）
* git push origin master  （提交到仓库）
* git push --tag  （将版本信息提交到仓库）

```javascript
git checkout master                //切至主线分支
git merge dev                      //合并dev分支
git tag -a v1.2.2.0 -m "v1.2.2.0"  //添加版本号
git push origin master             //提交到仓库
git push --tag                     //将版本信息提交到仓库
```

## 常用技巧操作

### 撤销操作

当git add文件后，发现这些文件并不想添加至暂存区，可以使用git reset命令

* git reset HEAD xxx  （撤回暂存的xxx）

```javascript
git reset HEAD /app/module/main.js
```

当文件修改后，并不想保留文件的修改，而是希望撤销至最近一次commit的版本，使用git checkout

* git checkout -- xxx  （撤销修改回退）

```javascript
git checkout --/app/module/main.js
```

当提交已经commit至本地仓库后，想要取消本次commit，使用git reset --hard <hash>，hash表示的是每次提交产生的hash值

* git reset --hard <hash>  （撤销此次'hash'提交）

```javascript
git reset --hard 6c8139366cbafdfdad01e30b43c5c04e04b6ba86
```

### 本地开发的项目上传至github中

有的时候我们是现在本地开发好项目，可能觉得这个项目比较好，才会想着上传到github上。

首先在github上创建一个新的仓库，创建完成后会生成一个远程仓库的地址，假如为
https://github.com/HappyDoublekang/VueProject.git

然后在本地的项目中运行git命令，依次如下

* git init  （初始化）
* git add README.md  （添加注释md）
* git commit -m "first commit"  （第一次暂存）
* git remote add origin https://github.com/HappyDoublekang/VueProject.git （项目怼进这个仓库）
* git push -u origin master  （提交保存）

```javascript
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/HappyDoublekang/VueProject.git
git push -u origin master
```

### 删除远程和本地分支

假如某一次操作，我们将分支的名字输错了，并push至远程仓库，此时我们应该将远程仓库的这个分支给删掉，需要用到一下命令

* git push origin --delete serverfix  （删除远程分支）
* git branch --d serverfix            （删除本地分支）

```javascript
git push origin --delete serverfix
git branch --d serverfix
```

### 关于pull和fetch

首先我们要知道的是pull=fetch + merge

如果我们使用pull命令，在本地开发完后，git add，git commit，然后git pull从远程拉取代码，这个时候可能出现冲突，在合并冲突后，git add，git commit，这个时候会重新产生一次提交
如果我们使用fetch+merge，在本地开发完后，git add，git commit，然后git fetch从远程拉取代码，这个时候并不会出现冲突，本地代码会出现一个FETCH-HEAD引用，表示最新的代码，然后git merge远程分支，这个时候才会出现冲突，在合并冲突后，git add，git commit，这个时候也会产生一次新的提交











