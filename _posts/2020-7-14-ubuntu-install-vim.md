---
layout: post
title: '在Ubuntu20.04下安装配置vim'
subtitle: '如何在Ubuntu20.04下安装及配置vim'
date: 2020-07-14
categories: [Ubuntu,Vim]
image: /assets/img/blog/install-vim-ubuntu.png
accent_image: /assets/img/blog/install-vim-ubuntu.png
related_posts:
  - _posts/2020-7-16-ubuntu-add-usergroup.md
  - _posts/2020-7-11-installvim-onwin.md

---

- Table of Contents
{:toc .large-only}


在使用 Linux 系统的时候 不免的被vim 编辑器所吸引到

这就来安装vim 尝尝鲜

参考：https://www.linuxidc.com/Linux/2019-08/159772.htm

## 首先安装 vim 

```powershell
sudo apt install vim
```

## 安装插件管理工具 vundle

**安装vundle**

```powershell
git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
```

安装好了之后，需要在 配置文件中添加vundle支持

## 配置 vim

vim 的配置是在 用户目录下的 ~/.vimrc 文件中完成的，

```powershell
vim ~/.vimrc
```

## vim 配置信息
使用vim 打开/新建 ~/.vimrc 并写入以下内容

```shell
# file: '~/.vimrc'
filetype off
set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

if filereadable(expand("~/.vimrc.bundles"))
  source ~/.vimrc.bundles
endif
```

## 编辑bundles
可以看到 设置中指向了 ~/.vimrc.bundles 来读取插件

还需要 编辑 .vimrc.bundles

```powershell
vim ~/.vimrc.bundles
```

然后添加以下内容

```shell
# file: '~/.vimrc.bundles'
if &compatible
  set nocompatible
end

filetype off
set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

" Let Vundle manage Vundle
Bundle 'gmarik/vundle'


" Define bundles via Github repos
" 标签导航
Bundle 'majutsushi/tagbar'
Bundle 'vim-scripts/ctags.vim'
" 文件搜索
Bundle 'kien/ctrlp.vim'
" 目录树导航
Bundle "scrooloose/nerdtree"
" 美化状态栏
Bundle "Lokaltog/vim-powerline"
" 主题风格
Bundle "altercation/vim-colors-solarized"

" python自动补全
Bundle 'davidhalter/jedi-vim'

" 括号匹配高亮
Bundle 'kien/rainbow_parentheses.vim'
" 可视化缩进
Bundle 'nathanaelkane/vim-indent-guides'
if filereadable(expand("~/.vimrc.bundles.local"))
  source ~/.vimrc.bundles.local
endif

filetype on
```
## 安装插件
我们通过 Bundle 指定各个插件在GitHub的地址，填写规则是‘‘用户名/仓库名‘’

上部分已经指定好了各个插件的路径，接下来就是安装插件，在shell 中输入vim 进入

在命令行模式下 输入 ``:BundleInstall``

```powershell
~                                                                      
~                                                                          
~                                                                         
~                                                                          
~                                          
~                   
:BundleInstall
```

然后进入下载页面，可能会很长时间，取决于自己的网速

```powershell
  " Installing plugins to /home/bywlop/|
  .vim/bundle                          |~      
. Plugin 'gmarik/vundle'               |~                                     
+ Plugin 'majutsushi/tagbar'           |~                                      
+ Plugin 'vim-scripts/ctags.vim'       |~                                    
+ Plugin 'scrooloose/syntastic'        |~                                  
+ Plugin 'kien/ctrlp.vim'              |~                               
+ Plugin 'scrooloose/nerdtree'         |~           
+ Plugin 'Lokaltog/vim-powerline'      |~                           
+ Plugin 'altercation/vim-colors-solari|~                          
  zed'                                 |~                                 
+ Plugin 'davidhalter/jedi-vim'        |~                                 
> Plugin 'klen/python-mode'            |~                                  
  Plugin 'kien/rainbow_parentheses.vim'|~                                      
  Plugin 'nathanaelkane/vim-indent-guid|~                                       
  es'                                  |~                                   
  Helptags                             |~                                        
                                       |~                                         
~                                      |~ 
NORMAL  RO [Vundle] Installer - PRV   │  unix │ utf-8 │ vundle  100%  LN  15:1     [未命名]  100% │ LN   0:1  
Done! 
                                                   
```
## 安装ctags
由于tagbar依赖于ctags，所以我们还需要用指令安装ctags：

```powershell 
sudo apt-get install ctags
```

```powershell
(base) bywlop@bywlop-F117-B:~/$ sudo apt-get install ctags
[sudo] bywlop 的密码： 
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
注意，选中 'exuberant-ctags' 而非 'ctags'
下列【新】软件包将被安装：
  exuberant-ctags
升级了 0 个软件包，新安装了 1 个软件包，要卸载 0 个软件包，有 143 个软件包未被升级。
需要下载 129 kB 的归档。
解压缩后会消耗 345 kB 的额外空间。
```

## 插件配置

下载好插件之后，还需要对各个插件进行配置

再次打开 ``~/.vimrc``

```shell
# file: '~/.vimrc'
filetype off
set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

if filereadable(expand("~/.vimrc.bundles"))
  source ~/.vimrc.bundles
endif

" tagbar标签导航
nmap <Leader>tb :TagbarToggle<CR>
let g:tagbar_ctags_bin='/usr/bin/ctags'
let g:tagbar_width=30
autocmd BufReadPost *.cpp,*.c,*.h,*.hpp,*.cc,*.cxx call tagbar#autoopen()
let g:jedi#auto_initialization = 1

" 主题 solarized
let g:solarized_termtrans=1
let g:solarized_contrast="normal"
let g:solarized_visibility="normal"

" 目录文件导航NERD-Tree
" \nt 打开nerdree窗口，在左侧栏显示
nmap <leader>nt :NERDTree<CR>
let NERDTreeHighlightCursorline=1
let NERDTreeIgnore=[ '\.pyc$', '\.pyo$', '\.obj$', '\.o$', '\.so$', '\.egg$', '^\.git$', '^\.svn$', '^\.hg$' ]
let g:netrw_home='~/bak'
"close vim if the only window left open is a NERDTree
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") && b:NERDTreeType == "primary") | q | end

" ctrlp文件搜索
" 打开ctrlp搜索
let g:ctrlp_map = '<leader>ff'
let g:ctrlp_cmd = 'CtrlP'
" 相当于mru功能，show recently opened files
map <leader>fp :CtrlPMRU<CR>
" set wildignore+=*/tmp/*,*.so,*.swp,*.zip     " MacOSX/Linux"
let g:ctrlp_custom_ignore = {
    \ 'dir':  '\v[\/]\.(git|hg|svn|rvm)$',
    \ 'file': '\v\.(exe|so|dll|zip|tar|tar.gz)$',
    \ }
"\ 'link': 'SOME_BAD_SYMBOLIC_LINKS',
let g:ctrlp_working_path_mode=0
let g:ctrlp_match_window_bottom=1
let g:ctrlp_max_height=15
let g:ctrlp_match_window_reversed=0
let g:ctrlp_mruf_max=500
let g:ctrlp_follow_symlinks=1


Plugin 'vim-airline/vim-airline'
"vim-airline配置:优化vim界面"
"let g:airline#extensions#tabline#enabled = 1
" airline设置
" 显示颜色
set t_Co=256
set laststatus=2
" 使用powerline打过补丁的字体
let g:airline_powerline_fonts = 1
" 开启tabline
let g:airline#extensions#tabline#enabled = 1
" tabline中当前buffer两端的分隔字符
let g:airline#extensions#tabline#left_sep = ' '
" tabline中未激活buffer两端的分隔字符
let g:airline#extensions#tabline#left_alt_sep = ' '
" tabline中buffer显示编号
let g:airline#extensions#tabline#buffer_nr_show = 1


" 括号匹配高亮
let g:rbpt_colorpairs = [
    \ ['brown',       'RoyalBlue3'],
    \ ['Darkblue',    'SeaGreen3'],
    \ ['darkgray',    'DarkOrchid3'],
    \ ['darkgreen',   'firebrick3'],
    \ ['darkcyan',    'RoyalBlue3'],
    \ ['darkred',     'SeaGreen3'],
    \ ['darkmagenta', 'DarkOrchid3'],
    \ ['brown',       'firebrick3'],
    \ ['gray',        'RoyalBlue3'],
    \ ['black',       'SeaGreen3'],
    \ ['darkmagenta', 'DarkOrchid3'],
    \ ['Darkblue',    'firebrick3'],
    \ ['darkgreen',   'RoyalBlue3'],
    \ ['darkcyan',    'SeaGreen3'],
    \ ['darkred',     'DarkOrchid3'],
    \ ['red',         'firebrick3'],
    \ ]
let g:rbpt_max = 40
let g:rbpt_loadcmd_toggle = 0

" 可视化缩进
let g:indent_guides_enable_on_vim_startup = 0  " 默认关闭
let g:indent_guides_guide_size            = 1  " 指定对齐线的尺寸
let g:indent_guides_start_level           = 2  " 从第二层开始可视化显示缩进

map <F5> :w<cr>:r!python3 %<cr>




" 括号引号补全
inoremap ( ()<Esc>i
inoremap [ []<Esc>i
inoremap { {<CR>}<Esc>O
inoremap ) <c-r>=ClosePair(')')<CR>
inoremap ] <c-r>=ClosePair(']')<CR>
inoremap } <c-r>=CloseBracket()<CR>
inoremap " <c-r>=QuoteDelim('"')<CR>
inoremap ' <c-r>=QuoteDelim("'")<CR>
 
function ClosePair(char)
	if getline('.')[col('.') - 1] == a:char
		return "\<Right>"
	else
		return a:char
	endif
endf
 
function CloseBracket()
	if match(getline(line('.') + 1), '\s*}') < 0
		return "\<CR>}"
	else
		return "\<Esc>j0f}a"
	endif
endf
 
function QuoteDelim(char)
	let line = getline('.')
	let col = col('.')
	if line[col - 2] == "\\"
		"Inserting a quoted quotation mark into the string
		return a:char
	elseif line[col - 1] == a:char
		"Escaping out of the string
		return "\<Right>"
	else
		"Starting a string
		return a:char.a:char."\<Esc>i"
	endif
endf

 
 

" html自动补全
autocmd BufNewFile *  setlocal filetype=html
function! InsertHtmlTag()
	let pat = '\c<\w\+\s*\(\s\+\w\+\s*=\s*[''#$;,()."a-z0-9]\+\)*\s*>'
	normal! a>
	let save_cursor = getpos('.')
	let result = matchstr(getline(save_cursor[1]), pat)
	"if (search(pat, 'b', save_cursor[1]) && searchpair('<','','>','bn',0,  getline('.')) > 0)
	if (search(pat, 'b', save_cursor[1]))
		normal! lyiwf>
		normal! a</
		normal! p
		normal! a>
	endif
	:call cursor(save_cursor[1], save_cursor[2], save_cursor[3])
endfunction
inoremap > <ESC>:call InsertHtmlTag()<CR>a<CR><Esc>O
```

你可以根据自己的喜好设置快捷键，<leader>是按键\，根据我的配置。在Vim的正常模式下：

- 依次按键\tb，就会调出标签导航；
- 依次按键\ff，就会调出文件搜索；
- 依次按键\nt，就会调出目录导航。

## 运行配置

这些基础配置已经完成，但是我想在Vim下像在IDE中一样，按一个键就运行当前编辑的Python文件，并查看运行结果可以吗？

没问题！

在~/.vimrc最后一行追击代码如下：

" 运行文件
```shell
# file: '~/.vimrc'
map <F5> :w<cr>:r!python3 %<cr>
```
上述代码的意思就是，在Vim的正常模式下，按F5就会保存文件并使用Python3运行当前文件，并将结果输出到当前界面。

注意，:!python3表示运行系统命令Python3，如果你没有安装Python2和Python3共存，此处只写python即可。

这样我们就可以边编辑边查看运行结果了，见本文最上面截图。

运行完之后，依然可以在Vim的正常模式下按u，撤回这个输出操作，这样输出结果就撤回了，我们就可以继续编写自己的代码了。

## 总结

Vim很好用，很强大，用上了有种爱不释手的感觉。插件不用安装太多，适合自己的就行，根据自己的需求进行配置，编辑快捷键，真的很方便。

你也来配置一个属于自己的Vim吧！