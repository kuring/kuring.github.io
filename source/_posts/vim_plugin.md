---
title: vim插件安装
Status: private
url: vim_plugin
tags: vim
date: 2013-09-02 13:19:16
---


本文的安装环境为ubuntu13.04。为了以后便于查阅，本文将相关插件的使用放到了文章的开始部分。这里不作插件的相关介绍，相关介绍看文章底部的参考文章。

# 插件使用
本插件快捷键会跟随下文安装内容一块同步。

## ctags
在源码目录执行`ctags -R`可生成ctags文件。该文件在源码修改后并不会改变，需要重新生成ctags文件。
ctrl+]：转到函数定义处。
ctrl+T：回到执行`ctrl+]`的地方。

## taglist
`:TlistOpen`：打开taglist窗口
`:TlistClose`：关闭taglist窗口。
`:TlistToggle`：在打开和关闭间切换。

## NERD tree
`:NERDTree`：打开窗口。

## winmanager
`wm`：打开和关闭taglist和NERD tree窗口。

## a.vim
`:A`：在新Buffer中切换到c/h文件
`:AS`：横向分割窗口并打开c/h文件
`:AV`：纵向分割窗口并打开c/h文件
`:AT`：新建一个标签页并打开c/h文件
`F12`：代替`:A`命令

## MiniBufExplorer
`<Tab>`：向前循环切换到每个buffer名上
`<S-Tab>`：向后循环切换到每个buffer名上
`<Enter>`：在打开光标所在的buffer
`d`：删除光标所在的buffer

# 插件安装
## 安装ctags
执行： `sudo apt-get install ctags`。

## 安装taglist
1. 下载页面：http://www.vim.org/scripts/script.php?script_id=273。下载后得到taglist_46.zip文件。
2. 执行`unzip taglist_46.zip`解压文件。
3. 将解压出的文件复制到~/.vim目录下。`sudo cp ~/tmp/ ~/.vim/`。
4. 在~/.vimrc文件中添加如下：
```
let Tlist_Show_One_File = 1            "不同时显示多个文件的tag，只显示当前文件的
let Tlist_Exit_OnlyWindow = 1          "如果taglist窗口是最后一个窗口，则退出vim
let Tlist_Use_Right_Window = 1         "在右侧窗口中显示
```
参考网址：http://www.cnblogs.com/mo-beifeng/archive/2011/11/22/2259356.html

## 安装文件浏览器NERD tree
1. 下载页面：http://www.vim.org/scripts/script.php?script_id=1658。
2. 将下载后的nerdtree.zip文件解压到~/.vim目录下。 

## 安装winmanager
1. 下载页面：http://www.vim.org/scripts/script.php?script_id=95
2. 将下载后的winmanager.zip文件解压到~/.vim目录下
3. 修改.vimrc文件，添加：
```
let g:winManagerWindowLayout='FileExplorer|TagList'
nmap wm :WMToggle<cr>
```
这样利用winmanager工具将taglist和NERD tree工具整合到了一个块，输入wm可以打开和关闭窗口。

## 安装cscope
1. 下载页面：http://cscope.sourceforge.net，下载后得到文件cscope-15.8a.tar.gz。
2. ./configure
3. make。可能会出现错误，执行如下命令：
```
apt-get install libncurses-dev
sudo apt-get install flex
sudo apt-get install byacc
```
然后执行`make clean`后重新make。
4. sudo make install

## 安装在h/c文件之间切换插件a.vim
1. 下载页面：http://www.vim.org/scripts/script.php?script_id=31。
2. 将下载的a.vim文件复制到~/.vim/plugin文件夹下。
3. 在~/.vimrc文件中添加`nnoremap <silent> <F12> :A<CR>`
4. 下面内容为快捷键列表：
:A switches to the header file corresponding to the current file being edited (or vise versa) 
:AS splits and switches 
:AV vertical splits and switches 
:AT new tab and switches 
:AN cycles through matches 
:IH switches to file under cursor 
:IHS splits and switches 
:IHV vertical splits and switches 
:IHT new tab and switches 
:IHN cycles through matches 
<Leader>ih switches to file under cursor 
<Leader>is switches to the alternate file of file under cursor (e.g. on  <foo.h> switches to foo.cpp) 
<Leader>ihn cycles through matches

## 安装快速浏览和操作Buffer
1. 下载页面：http://www.vim.org/scripts/script.php?script_id=159
2. 将下载的 minibufexpl.vim文件丢到 ~/.vim/plugin 文件夹中即可
3. 在~/.vimrc文件中增加如下行：
```
let g:miniBufExplMapCTabSwitchBufs = 1
let g:miniBufExplMapWindowNavVim = 1
let g:miniBufExplMapWindowNavArrows = 1
```
4. 快捷键：
<Tab>	向前循环切换到每个buffer名上
<S-Tab>	向后循环切换到每个buffer名上
<Enter>	在打开光标所在的buffer
d	删除光标所在的buffer

# 参考文章
* [经典vim插件功能说明、安装方法和使用方法介绍](http://blog.csdn.net/tge7618291/article/details/4216977)
* [手把手教你把Vim改装成一个IDE编程环境](http://blog.csdn.net/wooin/article/details/1858917)