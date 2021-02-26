---
layout: post
title: 'git高级分支使用'
description: '如何使用分支'
categories: [Git]
tags: [Git命令]
image: /assets/img/blog/git.jpg
related_posts:
  - _posts/2020-05-27-git-branch.md
---

- Table of Contents
{:toc .large-only}

## Git分支
git 分支适用于多人联合开发，是多数公司常用的功能之一 。  

<br/>

如果你希望为项目增加新功能，但很可能会影响当前可正常工作的代码。与其将特性加入到其它人正在使用的 `master` 分支，更好的方法是在仓库的其它分支中变更代码，就需要用不同的分支隔离不同的功能，使各个功能在开发过程中互不影响，最终在代码稳定之后，合并到一起，完成整个项目的开发。   

<br/>

可以将分支理解为树 master就是树干  子分支就是枝叶，在子分支上修改代码不会更改主分支。  

<br/>

实际开发时 每人操作一个分支来写各自的功能，当功能完成的时候 就将他的分支和主分支合并，形成了单个模块逐一开发。  

<br/>

这样的好处就是 避免代码被覆盖 从而造成不必要的后果    

<br/>

## 如何使用分支功能

在创建仓库的时候选择生产开发模型  

当仓库允许多分支情况才可以创建分支进行操作

![分支选择](/assets/img/git/select_branch.png)  

<br/>

## 分支创建

查看所有分支 当前分支以 * 开头

```powershell
git branch  
```

新建一个分支，并切换到该分支下

```powershell
git checkout -b name 
```

切换指定分支下

```powershell
git checkout name 
```

切换到上一个分支

```powershell
git checkout -
```



## 提交分支

将本地的分支提交到线上库中

```powershell
git push --set-upstream origin name 
```



## 删除分支

删除本地分支

```powershell
git branch -d name 
```

删除远程分支

```powershell
git push origin --delete branch-name
```



## 合并分支
我们使用rebasel来合并分支
rebase 是将基础分支与当前分支的差异的commit获取到
然后在基础分支的最新的提交后面把差异的commit逐一提交
这样就会保持提交树的干净整洁

```powershell
# 进入到功能分支
git checout feature
# 获取与基础分支的差异commit
git rebase master

# 切换回基础分支
git checkout master

# 将功能分支上多出的commit合并到基础分支
git merge feature
```

当合并分支后就不用commit 提交了  

可以直接使用push将master主分支推送到线上   



  <br>

## 日常如何使用分支

```powershell
git add -A
git commit -m'first-commit'
git push origin branch  指定提交到分支下

```



