# 舒适美观的mac终端, iTerm2+zsh+powerlevel9k+vim+virtualenv

## 目录

* 前言
* homebrew
* iTerm2
* oh-my-zsh
* 配色
* 毛玻璃
* 字体
* powerlevel9k
* zsh插件
* vim设置

> * 配色
> * 代码折叠
> * 插件管理和使用

>> * vim-powerline
>> * 多行注释

* python配置
> * 修改pip源
> * virtualenv配置
> * virtualenvwrapper使用

## 前言

> ~~最近mac不知道怎么了, 估计是新品又要到来了, 水果决定解决老机型过于流畅的bug, 出现各种问题, 比如屏幕底部会突然花屏, 一次约0.1s, 或者是界面卡死之类的. 还有就是插上扩展坞网速就为零. 所以趁着中秋, 重装一下, 然后这些bug都没了(我太难了.jpg). 顺带写下这篇配置篇, 省得以后麻烦.~~
> 更新了vim配置内容

------

## homebrew

> 每次提到homebrew, 除了**必备神器**之外, 还有就是*谷歌: 我们90％的工程师使用您编写的软件(Homebrew), 但是您却无法在面试时在白板上写出翻转二叉树这道题, 这太糟糕了.*(手动滑稽)

> 安装也很简单:

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## iTerm2

> 你可以从[官网下载iterm2](https://www.iterm2.com), 也可以用homebrew:

```
brew install iTerm2
```
 
 > 然后你会发现一个非常非常朴素的终端, 基本和mac自带的终端差不多, 不多说, 上一张素颜照:
 
<img width="1552" alt="屏幕快照 2019-09-13 下午1 16 32" src="https://user-images.githubusercontent.com/21376904/64861459-92d3bd80-d662-11e9-9796-bb0c8d63c48f.png">
 
 > 接下来, 你就会和我一起, 将它打造成一个性冷淡御姐, 你懂我意思吧(老奸巨猾.jpg)
 
----

## oh-my-zsh

> 指令安装:

```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

> 然后从bash切换到zsh, 其实我安装完成之后, 它自动切换了:

```
chsh -s /bin/zsh
```

![image](https://user-images.githubusercontent.com/21376904/64861429-818ab100-d662-11e9-9b83-d3bf7823b908.png)

-----

## 配色

> iTerm2自带了一些配色, 但那肯定是不够的.

```
mkdir ~/iterm2 ; cd ~/iterm2
git clone https://github.com/mbadolato/iTerm2-Color-Schemes
```

> 建一个目录, 叫什么都行, 然后下载这个配色方案包, 之后通过command + ,打开配置, 导入刚才下载的配色:

![image](https://user-images.githubusercontent.com/21376904/64861797-7c7a3180-d663-11e9-870c-2a1d7c09aff6.png)
![image](https://user-images.githubusercontent.com/21376904/64861690-2e652e00-d663-11e9-989e-b9c39a22b166.png)

> 然后你就收获了满满的幸福:

<img width="237" alt="Snipaste_2019-09-13_14-31-02" src="https://user-images.githubusercontent.com/21376904/64861870-b0eded80-d663-11e9-821b-1d5ef575fcf7.png">

> 然后你可以使用```mv iterm2 .iterm2```指令隐藏这个文件夹, 也可以不隐藏, 看你喜欢了.

-----

## 毛玻璃

> 然后可以调整一下透明度和模糊度, b格满满的毛玻璃效果就出现了:

![image](https://user-images.githubusercontent.com/21376904/64861994-04f8d200-d664-11e9-8ba5-a88cdc5d59e2.png)
<img width="1438" alt="Snipaste_2019-09-13_14-38-31" src="https://user-images.githubusercontent.com/21376904/64862030-17730b80-d664-11e9-91f3-0c9fb15c5dc6.png">

------

## 字体

> 字体其实是非常非常重要的, 回忆一下window终端的糟糕字体吧, 其实字体是非常影响整个系统的观感的, 从软件的角度来说也是如此.

> 这里安装[nerd-fonts字体](https://github.com/ryanoasis/nerd-fonts), 它的好处是还支持图标.

<img width="905" alt="Snipaste_2019-09-13_20-27-25" src="https://user-images.githubusercontent.com/21376904/64862379-f363fa00-d664-11e9-9887-41fa180c9cf4.png">

```
brew tap caskroom/fonts
brew cask install font-hack-nerd-font
```

> 然后在配置文件里面勾选, 注意, ascii和非ascii要一样大, 不一样会造成之后图标有些不对齐:

<img width="930" alt="Snipaste_2019-09-13_20-28-44" src="https://user-images.githubusercontent.com/21376904/64862447-1d1d2100-d665-11e9-9aad-f1e8c391f535.png">

-----

## powerlevel9k

> [powerlevel9k](https://github.com/Powerlevel9k/powerlevel9k)真的是一个很酷的东西.

```
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```

> 然后打开zsh的配置文件, 将主题设置进去:

```
vim ~/.zshrc
ZSH_THEME="powerlevel9k/powerlevel9k"
```

> 退出来之后更新一下zsh```source ~/.zshrc```. powerlevel9k本身还有许多设置内容, 这里我简单设置一下, 大家可以按需设置.

```
POWERLEVEL9K_MODE="nerdfont-complete"
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(ssh dir vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status root_indicator background_jobs virtualenv)
```

> 我的设置第一行代表用之前的nerd-fonts字体

> 第二行设置左边的图标显示内容, 分别是ssh, 目录和git等版本管理

> 第三行设置右, 依次是状态, 是否是root, 作业指示器, py的环境. 更多设置, 可以参看[这篇文章](https://www.helplib.com/GitHub/article_121915)

-----

## zsh插件

> 多的不说, 语法高亮和指令提示肯定要的.

```
brew install zsh-syntax-highlighting
brew install zsh-autosuggestions
```

> 然后在.zshrc里面补上如下内容:

```
source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source /usr/local/share/zsh-autosuggestions/zsh-autosuggestions.zsh
```

----

## vim设置

> 我个人还是很喜欢用vim的, 只要不是太复杂的环境, 我都尽量使用vim进行代码编辑. 来看看一些设置和配置吧. 当然, 你可以直接[打造成IDE](https://github.com/yangyangwithgnu/use_vim_as_ide)
> homebrew进行安装:

```
brew install vim
```

> 然后我先贴出全部的vimrc配置文件内容:

```
" vundle 环境设置
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
" 插件列表开始
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'altercation/vim-colors-solarized'
Plugin 'tomasr/molokai'
Plugin 'vim-scripts/phd'
Plugin 'Lokaltog/vim-powerline'
Plugin 'octol/vim-cpp-enhanced-highlight'
Plugin 'nathanaelkane/vim-indent-guides'
Plugin 'derekwyatt/vim-fswitch'
Plugin 'kshenoy/vim-signature'
Plugin 'vim-scripts/BOOKMARKS--Mark-and-Highlight-Full-Lines'
Plugin 'majutsushi/tagbar'
Plugin 'vim-scripts/indexer.tar.gz'
Plugin 'vim-scripts/DfrankUtil'
Plugin 'vim-scripts/vimprj'
Plugin 'dyng/ctrlsf.vim'
Plugin 'terryma/vim-multiple-cursors'
Plugin 'scrooloose/nerdcommenter'
Plugin 'vim-scripts/DrawIt'
Plugin 'SirVer/ultisnips'
Plugin 'derekwyatt/vim-protodef'
Plugin 'scrooloose/nerdtree'
Plugin 'fholgado/minibufexpl.vim'
Plugin 'gcmt/wildfire.vim'
Plugin 'sjl/gundo.vim'
Plugin 'Lokaltog/vim-easymotion'
Plugin 'suan/vim-instant-markdown'
Plugin 'lilydjwg/fcitx.vim'
" 插件列表结束
call vundle#end()
filetype plugin indent on

" 定义快捷键的前缀, 即<Leader>
let mapleader=";"

" 设置状态栏主题风格
let g:Powerline_colorscheme='solarized256'

" 基于缩进或语法进行代码折叠
set foldmethod=syntax
" 启动 vim 时关闭折叠代码
set nofoldenable

" 打开语法高亮
syntax on

" 使用配色方案
colorscheme gruvbox 
 
" 打开文件类型检测功能
filetype on

" 不同文件类型采用不同缩进
filetype indent on
 
" 允许使用插件
filetype plugin on
filetype plugin indent on

" 关闭vi模式
set nocp
 
" 与mac共享剪贴板
set clipboard+=unnamed

" 取消VI兼容
set nocompatible
 
" 显示行号
set nu

" 历史命令保存行数 
set history=1000
 
" 当文件被外部改变时自动读取
set autoread 

" 取消自动备份及产生swp文件
set nobackup
set nowb
set noswapfile
 
" 允许使用鼠标点击定位
set mouse=a
" 允许区域选择
set selection=exclusive
 
" 高亮光标所在行
set cursorline
" 取消光标闪烁
set novisualbell
 
" 总是显示状态行
set laststatus=2
" 状态栏显示当前执行的命令
set showcmd
 
" 标尺功能, 显示当前光标所在行列号
set ruler
" 设置命令行高度为2
set cmdheight=2
 
" 粘贴时保持格式
" set paste
" 高亮显示匹配的括号
set showmatch
 
" 在搜索的时候忽略大小写
set ignorecase
 
" 高亮被搜索的句子
set hlsearch
 
" 在搜索时, 输入的词句的逐字符高亮
set incsearch
" 继承前一行的缩进方式
set autoindent
 
" 为c程序提供自动缩进
set smartindent
" 使用c样式的缩进
set cindent
 
" 制表符为4
set tabstop=4
" 统一缩进为4
set softtabstop=4
set shiftwidth=4
 
" 允许使用退格键
set backspace=eol,start,indent
set whichwrap+=<,>,h,l
" 取消换行
set nowrap
 
" 在被分割的窗口间显示空白
set fillchars=vert:\ ,stl:\ ,stlnc:\
 
" 光标移动到buffer的顶部和底部时保持3行距离
set scrolloff=3
" 设定默认解码
set fenc=utf-8
set fencs=utf-8,usc-bom,euc-jp,gb18030,gbk,gb2312,cp936
 
" 设定编码
set enc=utf-8
set fileencodings=ucs-bom,utf-8,chinese
set langmenu=zh_CN.UTF-8
language message zh_CN.UTF-8
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim
 
" 自动补全
filetype plugin indent on
set completeopt=longest,menu

" 自动补全命令时候使用菜单式匹配列表
set wildmenu
autocmd FileType ruby,eruby set omnifunc=rubycomplete#Complete
autocmd FileType python set omnifunc=pythoncomplete#Complete
autocmd FileType javascript set omnifunc=javascriptcomplete#CompleteJS
autocmd FileType html set omnifunc=htmlcomplete#CompleteTags
autocmd FileType css set omnifunc=csscomplete#CompleteCSS
autocmd FileType xml set omnifunc=xmlcomplete#CompleteTags
autocmd FileType java set omnifunc=javacomplete#Complet
```


----

### 配色

> 其实之前iTerm已经配色过了的, 但是vim有自己的独立配色. 用法也很简单:
> 用如下命令创建.vim/colors目录, 然后下载配色文件:

```
mkdir -p ~/.vim/colors ; cd ~/.vim/colors
curl -O https://raw.githubusercontent.com/nanotech/jellybeans.vim/master/colors/jellybeans.vim
```

> 打开~/.vimrc, 加入```colorscheme jellybeans```, 比方说, 我用了这个jellybeans.vim主题配色.
> 这里再推荐一个gruvbox主题, 效果如下:

<img width="1440" alt="Snipaste_2019-09-13_21-33-52" src="https://user-images.githubusercontent.com/21376904/64866553-39718b80-d66e-11e9-9a85-205ea2d80cf1.png">

-----

### 代码折叠

> 使用自带代码折叠

```
" 基于缩进或语法进行代码折叠
"set foldmethod=indent
set foldmethod=syntax
" 启动 vim 时关闭折叠代码
set nofoldenable
```

> * 进入vim命令模式, za即可折叠当前块
> * zM关闭所有折叠
> * zR打开所有折叠

![](https://img-blog.csdnimg.cn/2019092410593853.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3M3OTMyMjM3MDY=,size_16,color_FFFFFF,t_70)

-----

### 插件管理和使用

> 安装管理工具vundle:

```
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

> 打开.vimrc, 加入下列插件内容:

```
" vundle 环境设置
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
" vundle 管理的插件列表必须位于 vundle#begin() 和 vundle#end() 之间
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'altercation/vim-colors-solarized'
Plugin 'tomasr/molokai'
Plugin 'vim-scripts/phd'
Plugin 'Lokaltog/vim-powerline'
Plugin 'octol/vim-cpp-enhanced-highlight'
Plugin 'nathanaelkane/vim-indent-guides'
Plugin 'derekwyatt/vim-fswitch'
Plugin 'kshenoy/vim-signature'
Plugin 'vim-scripts/BOOKMARKS--Mark-and-Highlight-Full-Lines'
Plugin 'majutsushi/tagbar'
Plugin 'vim-scripts/indexer.tar.gz'
Plugin 'vim-scripts/DfrankUtil'
Plugin 'vim-scripts/vimprj'
Plugin 'dyng/ctrlsf.vim'
Plugin 'terryma/vim-multiple-cursors'
Plugin 'scrooloose/nerdcommenter'
Plugin 'vim-scripts/DrawIt'
Plugin 'SirVer/ultisnips'
Plugin 'derekwyatt/vim-protodef'
Plugin 'scrooloose/nerdtree'
Plugin 'fholgado/minibufexpl.vim'
Plugin 'gcmt/wildfire.vim'
Plugin 'sjl/gundo.vim'
Plugin 'Lokaltog/vim-easymotion'
Plugin 'suan/vim-instant-markdown'
Plugin 'lilydjwg/fcitx.vim'
" 插件列表结束
call vundle#end()
filetype plugin indent on
```

> 进入vim, 命令模式下键入```:PluginInstall```, 即可自动安装:

![](https://img-blog.csdnimg.cn/20190924100947645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3M3OTMyMjM3MDY=,size_16,color_FFFFFF,t_70)

#### vim-powerline

> 利用插件Lokaltog/vim-powerline, 设置状态栏风格:

```
" 设置状态栏主题风格
let g:Powerline_colorscheme='solarized256'
```

![](https://img-blog.csdnimg.cn/20190924100921273.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3M3OTMyMjM3MDY=,size_16,color_FFFFFF,t_70)

#### 多行注释

> 多行注释肯定要的. 首先添加<Leader>按钮, 你可以添加自己顺手的, 我这里是;:

```
" 定义快捷键的前缀, 即<Leader>
let mapleader=";"
```

> 然后<Leader>cc注释选中区域, 你可以用v进入选中模式选择多行.
> <Leader>cu取消注释

----

## python配置

### 修改pip源

> 首先改一下pip的源:

```
mkdir .pip ; cd .pip
vim pip.conf
```

> 替换阿里源:

```
[global]
index-url=http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
```

> 或者你喜欢的:

> * http://pypi.douban.com/
> * http://pypi.hustunique.com/
> * http://pypi.sdutlinux.org/
> * http://pypi.mirrors.ustc.edu.cn/

----

### virtualenv配置

> 我使用的是virtualenv, 如果你是其它环境也是可以的:

```
pip3 install virtualenv
```

> 使用```virtualenv --version```看下是否安装成功.

### virtualenvwrapper使用

> Virtaulenvwrapper是对virtualenv的封装, 可以更方便地管理虚拟环境:

```
pip3 install virtualenvwrapper
```

> 建立一个环境目录, 比方说```mkdir ~/pyEnv```
> 打开.zshrc文件, 输入如下内容:

```
export WORKON_HOME=~/pyEnv
source /usr/local/bin/virtualenvwrapper.sh
```

> ```source ~/.zshrc```更新配置文件, 会有如下图内容:

![image](https://user-images.githubusercontent.com/21376904/64868074-34620b80-d671-11e9-8295-4eff4cabcd13.png)

> 之后就能欢乐地建立环境了:

> * 建立环境: mkvirtualenv env1
> * 删除环境: rmvirtualenv env1
> * 切换环境到其它环境: workon env2
> * 退出环境: deactivate
> * 列出所有环境: lsvirtualenv -b
> * 查看环境里的轮子: lssitepackages

<img width="1440" alt="Snipaste_2019-09-13_22-02-57" src="https://user-images.githubusercontent.com/21376904/64868616-46907980-d672-11e9-864a-13e4abaea4c4.png">
