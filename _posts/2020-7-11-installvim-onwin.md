---
layout: post
title: '在windows系统下如何安装vim编辑器'
description: '如何在 Windows上安装vim及配置'
categories: Vim
image: /assets/img/blog/vim.png
related_posts:
  - _posts/2020-7-14-ubuntu-install-vim.md
---

- Table of Contents
{:toc .large-only}

## Vim编辑器

vim是**Linux**下最常用的编辑工具,很多时候我们把vim称为vi。

在ubuntu新版的版本中使用``vi``命令来开启vim。

vim 是一个上手极难 但一熟练就会大大提高效率的编辑器，

作为一个老牌编辑器，有着长达30多年的历史，经历了时间的验证，既然存活到现在，它一定有受人喜欢的一面。

## 在window系统下安装vim 

官网下载gvim8，注意根据系统类型选择32或者64位，这里我们选择64位的，下载地址是：https://tuxproject.de/projects/vim/x64/

![download_vim](/assets/img/vim/download_vim.png)

<br/>

下载安装到C:/vim目录下

 配置好环境变量，这样就可以在系统任意位置启动vim

![环境变量](/assets/img/vim/environment_variable.png)

## vim配置

在vim/下，建立一个_vimrc文件，这是vim的配置文件，所有的设置都在这里编写

将以下内容添加到_vimrc文件中

```shell
" An example for a vimrc file.
"
" Maintainer:	Bram Moolenaar <Bram@vim.org>
" Last change:	2019 Dec 17
"
" To use it, copy it to
"	       for Unix:  ~/.vimrc
"	      for Amiga:  s:.vimrc
"	 for MS-Windows:  $VIM_vimrc
"	      for Haiku:  ~/config/settings/vim/vimrc
"	    for OpenVMS:  sys$login:.vimrc

" When started as "evim", evim.vim will already have done these settings, bail
" out.
if v:progname =~? "evim"
  finish
endif

" Get the defaults that most users want.
source $VIMRUNTIME/defaults.vim

if has("vms")
  set nobackup		" do not keep a backup file, use versions instead
else
  set backup		" keep a backup file (restore to previous version)
  if has('persistent_undo')
    set undofile	" keep an undo file (undo changes after closing)
  endif
endif

if &t_Co > 2 || has("gui_running")
  " Switch on highlighting the last used search pattern.
  set hlsearch
endif

" Put these in an autocmd group, so that we can delete them easily.
augroup vimrcEx
  au!

  " For all text files set 'textwidth' to 78 characters.
  autocmd FileType text setlocal textwidth=78
augroup END

" Add optional packages.
"
" The matchit plugin makes the % command work better, but it is not backwards
" compatible.
" The ! means the package won't be loaded right away but when plugins are
" loaded during initialization.
if has('syntax') && has('eval')
  packadd! matchit
endif

set encoding=utf-8
set fileencodings=utf-8,chinese,latin-1
if has("win32")
    set fileencoding=chinese
else
    set fileencoding=utf-8
endif

set autoindent
set nu!
set shiftwidth=4

source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim

language messages zh_CN.utf-8

colo koehler
set guifont=monaco:h11:cANSI

set ts=4
set expandtab

map <F5> :! python.exe %
```

这些都是一些最基本的配置，比如设置编码解决中文乱码问题、自动缩进以及缩进宽度、菜单栏中文字体问题、主题和字体、以及四个空格代替制表符等等，注意一点这个配置里我将运行python脚本的快捷键设置成了f5。

## 启动vim
进入windows命令行，输入gvim启动编辑器，然后键入命令:version，看到版本号就没有问题了

![version](/assets/img/vim/version.png)

## 安装插件管理器

 虽然现在Vim已经可以正常使用了，但是没有插件的加成，开发效率就不是那么高，所以我们现在来安装一些常用的插件。

  安装pathogen.vim插件（一个vim插件管理器）

  地址是：https://github.com/tpope/vim-pathogen 直接Clone或者下载压缩包

  将Clone或者解压后的pathogen.vim文件放到C:/vim/autoload目录下

修改用户目录下的_vimrc配置文件，将下面的配置加进去

```
execute pathogen#infect()
```

这样就可以安装其他所有的插件了
## 安装项目管理插件
我们安装一个项目管理插件(project)，它可以帮助我们把项目整体导入vim编辑器内，通过点击文件进行修改，这样就不用每次编辑都要在命令行输入命令才能编辑了，大体上，这个插件可以帮我们快速修改整个项目。

  同样的，先进行下载，地址是：https://www.vim.org/scripts/script.php?script_id=69

![20200704160732_39181](/assets/img/vim/20200704160732_39181.png)

<br/> 

将解压后的doc目录中的project文件拷贝到vim安装目录的doc目录下

  将plugin目录下的project.vim拷贝到vim安装目录的plugin目录下

  在命令行输入gvim启动编辑器

  输入:Project

  按住反斜杠 输入大写C，因为是输入命令，所以不会在编辑内显示，但是执行成功后会弹出窗口)

 Enter the Name of the Entry: 输入项目名

  Enter the Absolute Directory to Load: 输入项目的文件目录路径（项目目录需要事先存在）

  Enter the CD parameter: 这个和项目目录路径一样即可

  Enter the File Filter: 设置管理的文件类型，*.py,*.txt等等，可以设置多个，不设置（直接回车）默认为所有类型

  再次使用：打开vim后输入:Project
  使用回车打开或关闭标签。
  添加或者修改文件后可以使用\R进行刷新项目。

  这样我们就可以在vim里管理我们的项目了

![project](/assets/img/vim/project.png)

<br/> 

每次导入项目后，你都可以在用户目录的.vimprojects文件中进行修改或者删除项目，非常灵活

![vimprojects](/assets/img/vim/vimprojects.png)

## 安装代码补全插件
好了，项目导入后就可以愉快的开发了，但是我们发现vim默认没有代码补全，怎么办呢，聪明如你一定已经猜到可以用插件搞定，使用pydiction，下载地址：https://github.com/rkulla/pydiction

Clone或者下载压缩包之后，发现里面有after文件夹、complete-dict、pydiction.py

 将after里面的python_pydiction.vim文件拷贝到 vim安装目录下的ftpplugin里面，将complete-dict、pydiction.py 拷贝到ftpplugin目录下。

 随后在_vimrc里面添加 

```
复制filetype plugin onlet g:pydiction_location='C:vimftplugincomplete-dict'let g:pydiction_menu_height = 3
```

  这就搞定了，使用方法是，敲入两个字母之后使用tab键进行补全，效果是下面这样

![补全](/assets/img/vim/completion.png)

<br/> 

 还不错吧，有的时候，你甚至想用vim来编辑前端的页面，没有任何问题，使用autocomplpop插件，下载地址：https://vim.sourceforge.io/scripts/script.php?script_id=1879

  解压后，将plugin下的脚本文件(.vim)、doc下的帮助文件(.txt)和autoload下的(.vim)文件分别拷贝至vim的 plugin、doc和autoload目录

  这个插件甚至不需要配置，只需要在输入/insert模式下即可自动根据当前文档内的内容进行自动补全

![html补全](/assets/img/vim/html_completion.png)

## 基本用法 

Vim 有两种模式——Normal 模式和 Insert 模，所有命令都是在 Normal 模式下执行。启动 Vim 后，默认进入 Normal 模式，可以按 i 键进入 Insert 模式，或者 s 删除当前字符并进入 Insert 模式，退出 Insert 模式进入 Normal 按 ESC 。



```
i insert 输入
v 行选中
ctrl+v 列选中G 至文末
gg 至文首
:q 未修改退出
:q! 强制不保存退出
:x / :wq 保存并退出
J 合并多行
d 删除当前所选
dd 删除多行并存在剪贴板中（剪切）
y 复制当前所选
yy 复制整行
p 粘贴
u 撤销操作
w 光标移动到下一个单词处
b 光标移动到上一个单词处
^ 光标移动到行首
$ 光标移动到行尾
kjhl 或者上下左右键移动光标
shift+上下键 翻页
shift+左右 光标乙至上/下一个单词（以空格/标点区分单词）词首
u 撤销上一步操作
zo/zn/zc 折叠/展开代码块
:vsp 新建工作区
ctrl+w 松手后再按 方向键 切换工作区
:MR 选择最近打开的文件（需安装插件）
F12 运行当前文件
# 搜索光标处短语
:set paste 进入粘贴模式
:%s/target/something/g 替换全部 target 字段
:s/target/something/g 替换选中区域 target 字段
```

![vi-vim-cheat-sheet-sch](/assets/img/vim/vi-vim-cheat-sheet-sch.gif)

