# Vim

对于我个人而言，非常喜欢使用 Vim。喜欢到什么程度呢？无 VIM 不编码。这一篇文档描述了 Vim 的一些基本用法，掌握这些内容，基本上就可以在各种环境下使用 Vim 了。

## 安装 {id="install"}

Vim 的安装还是比较简单的，大部分的 Linux 发行版的都预装了它。但是如果你希望使用新版的 Vim8，可能需要卸载旧版从新编译安装一下。唯一需要注意的是，编译选项中需要开启 Python3 解释器的支持。基本的安装过程如下:
```shell
sudo dnf install ncurses-devel -y
git clone https://github.com/vim/vim.git
cd vim/src
## 启用对 pyhton3 解释器的支持
./configure --enable-python3interp
make distclean  # if you build Vim before
make
sudo make install
```
安装完成之后，会在 `src` 目录下编译出一个二进制的文件 vim ，把这个文件直接移动到 PATH 目录下就可以了:

```Shell
$ mv src/vim /usr/bin/vim
```

## 基本使用 {id="usage"}

下面将介绍一些 Vim 的基本使用，一定要把基本的使用多多练习，为深入的内容打好基础。

### 移动光标 {id="move"}

首先要学习的一定是如何移动光标，不允许使用方向键，而是在普通模式下使用 `hjkl` 四个字母按键:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/ZE2ZyGg4OxJQ1Yd28uQ7.png" alt="move in vim"/>

| 场景        | 操作            | 英文助记         |
|-----------|---------------|--------------|
| 跳转到行首     | 普通模式下: `^`    |              |
| 跳转到行尾     | 普通模式下: `$`    |              |
| 跳转到往后某个字符 | 普通模式下: `t<c>` | To Char      |
| 跳转到往前某个字符 | 普通模式下: `F<c>` | Forward Char |
| 跳转到文件头    | 普通模式下: `gg`   |              |
| 跳转到文件尾    | 普通模式下: `G`    |              |
| 跳转到指定行    | 命令模式下输入行号     |              |

### 删除操作

常见的删除操作如下表所示:

| 场景       | 操作                 | 英文助记               |
|----------|--------------------|--------------------|
| 向前删除一个字符 | 普通模式下: `X`         |                    |
| 向后删除一个字符 | 普通模式下: `x`         |                    |
| 删除一个单词   | 普通模式下: `daw`       | Delete a Word      |
| 删除多个单词   | 普通模式下: `d<n>w`     | Delete n Word      |
| 删除到指定字符  | 普通模式下: `dt<c>`     | Delete To Char     |
| 删除一行     | 普通模式下: `dd`        |                    |
| 删除全部内容   | 命令模式下使用 `:%d`      |                    |
| 删除连续的多行  | 命令模式下使用`:<n>,<m>d` | From n To m Delete |
| 删除所有的空行  | 命令模式下使用`:g/^$/d`   | Delete             |

> 注意: 删除操作等同于剪贴操作，如果是剪切操作,再删除之后按下 `p`键就可以实现粘贴。

### 插入模式 {id="insert"}

下表记录了一些快捷切换到插入模式的操作:

| 场景             | 操作         | 英文助记   |
|----------------|------------|--------|
| 进入插入模式         | 普通模式下: `i` | Insert |
| 光标移动到上一行，并插入   | 普通模式下: `O` |        |
| 光标移动到下一行，并插入   | 普通模式下: `o` |        |
| 光标移动到下个字符后，并插入 | 普通模式下: `a` | Append |

### 查找与替换 {id="search-and-replace"}

查找的相关场景的操作如下表所示:

| 场景     | 操作                           | 英文助记 |
|--------|------------------------------|------|
| 从上往下查找 | 普通模式下输入: `/<word>`           |      |
| 从下往上查找 | 普通模式下输入: `?<word>`           |      |
| 下一个搜索项 | 搜索模式下输入: `n`                 | Next |
| 上一个搜索项 | 搜索模式下输入: `N`                 | Next |
| 快速查找   | 光标定位到查找的单词，然后输入: `SHIFT + *` |      |

替换的相关场景的操作如下表所示:

| 场景             | 操作                                            | 英文助记 |
|----------------|-----------------------------------------------|------|
| 当前行查找替换, 匹配第一个 | `s/<find>/<replace>/`                         |      |
| 当前行查找替换, 匹配所有  | `s/<find>/<replace>/g`                        |      |
| 整个文件查找替换，匹配第一个 | `%s/<find>/<replace>/`                        |      |
| 整个文件查找替换，匹配所有  | `%s/<find>/<replace>/g`                       |      |
| 指定范围查找替换,匹配第一个 | `<start line>,<end line>s/<find>/<replace>/`  |      |
| 指定范围查找替换，匹配所有  | `<start line>,<end line>s/<find>/<replace>/g` |      |

## 分屏 {id="split-screen"}

默认情况下，我们打开的都是单个文件。那么如果打开多个文件呢？那就需要将其分屏显示。在打开文件的时候，给 vim 加上 `-o`  或者 `-O` 来指定多个文件，并分屏显示，前者表示水平分割后者表示垂直分割:
```Shell
vim -O index.html style.css
```

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/H9V0FQclfdphcnTyzIfX.png" alt="split screen in vim"/>

我们也可以对当前文件进行分割, 使用快捷键 `CTRL+S` 或者 `CTRL+V` , 前者是垂直分割，后者是水平分割。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/pbdmzuRSavILHtIRx64W.png" alt="split screen in vim"/>

如果是要对不同的文件进行分割，就需要在命令模式下使用 `split <FileName>` 或者 `vsplit <FileName>` , 同样的道理，前者是垂直分割或者是水平分割:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/0XwgL760tuYSrekYKWQu.png" alt="split screen in vim"/>

> 注意: split 可以简写为 sp, 而 vsplit 可以简写成 vs。

如何操作分配呢？如下表所示:

| 对分屏的操作          | 如何操作                                                                      |
|-----------------|---------------------------------------------------------------------------|
| 关闭其他分配，仅仅留下当前分屏 | 在 Command 模式下输入 only, 或者使用快捷键 `CTRL + W` , 然后输入 `o`， 也可以按两次 `CTRL+W` 顺序跳转 |
| 退出当前所在分屏        | 在 Command 模式下输入 q 或者 quit                                                 |
| 在分屏之间跳转         | 使用快捷键 `CTRL+W` , 然后输入 `hjkl` 四个字母中的一个表示方向                                 |
| 扩大或者缩小分屏范围      | 使用快捷键 `CTRL+W` , 然后输入 `+` 表示扩大，输入 `-` 表示缩小                                |
| 调整分屏顺序          | 使用快捷键 `CTRL+W` , 然后输入 `r`                                                 |
| 新建一个文件          | 在 Command 模式下输入 new                                                       |

更换配色方案是一件简单的事情，确实最能够吸引眼球的事情。我们可以从 GitHub 上找到对应的配置方法，多半只是一个以 `.vim` 结尾的文件而已，然后将这个文件放在 `~/.vim/colors` 目录下使用 Git Clone 即可。

而我选择的配置方案是 [morhetz/gruvbox](https://github.com/morhetz/gruvbox)。效果图如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/6onJ3w5iWRSZFvULcbxr.png" alt="using morhetz/gruvbox code style"/>

## 使用插件管理工具-VimPlug

VimPlug 是一个崇尚极简主义的一个包管理工具。我们可以使用如下命令进行安装:
```shell
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```
然后我们在 vim 的配置文件( Linux 位于 `~/.vimrc` ）中加入如下配置:
```python
call plug#begin('~/.vim/plugged')
" 下面这些是插件，可以从 Github 找
Plug 'fatih/vim-go'
Plug 'ycm-core/YouCompleteMe'
Plug 'preservim/nerdtree'
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'davidhalter/jedi-vim'
Plug 'tiagofumo/vim-nerdtree-syntax-highlight'
Plug 'mattn/emmet-vim'
call plug#end()
```
并不需要理解这些配置是什么意思，主要的目的就是为了启用 Minpac 这个插件管理工具。安装插件使用`:PlugInstall` 命令，在 Vim 的命令模式下。

## 语法补足 {id="you-complete-me"}

使用强大的 YouCompleteMe 可以补足语法，它是 Ycmd 的客户端。所以，首先我们需要安装 Ycmd：

<code-block lang="shell">
git clone https://github.com/ycm-core/ycmd.git
cd ycmd
git submodule update --init --recursive
dnf install gcc gcc-g++ -y
python3 build.py --all
## 启动 ycmd 服务
python3 ycmd/__main__.py --port 8888 --options_file ycmd/default_settings.json >> /dev/null &amp;
## 安装 YouCompleteMe, --go-completer 表示启用 Go 自动完成
python3 ~/.vim/plugged/YouCompleteMe/install.py --go-completer
</code-block>

## 显示目录树-NERDTree {id="using-NERDTree"}

首先我们需要安装 `preservim/nerdtree` 插件，这是一个非常好用的目录树插件。

在安装后，我们使用 Vim 打开一个文件后，使用命令 `:NERDTree` 就可以打开目录树，如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/SCfJn9bptqVJNYFMTMjv.png" alt="NERDTree"/>

如何切换左边的目录和右边的文件呢？

- 切换目录，使用 `<C-W-h>`
- 切换文件，使用 `<C-W-l>`

如何对目录进行操作呢？

- 使用 `P` 可以跳到最顶层的根目录，使用 `p` 可以跳到父级目录。
- 在任意的目录节点上，使用 `m` 可以调出操作菜单，对节点进行增删改。
- 使用 `R` 可以刷新目录节点。
- 默认情况下，不会显示隐藏的目录，可以使用 `I` 显示隐藏或显示的目录或文件。
- 当光标定位在文件节点，我们可以在右边的窗口打开它。使用 `i` 是在水平切割的窗口打开，使用 `s` 是在垂直切割的窗口中打开，而使用 `t` 是在新的标签页中打开。
- 标签页的切换可以使用 `gt` 或者指定数字的标签页 `1gt` 、 `2gt` 。

如何调正分栏的大小呢？使用 `<C-W>+` 或者 `<C-W>-` 。如果要在打开 Vim 的时候自动开启目录，在 Vimrc 中如下配置:
```shell
autocmd VimEnter * NERDTree
```
## 状态栏美化-Airline

Vim 默认的状态栏并不好看，我们需要对其进行美化。首先我们安装两个插件:
```
call minpac#add('vim-airline/vim-airline')
call minpac#add('vim-airline/vim-airline-themes')
```
然后，我们在 `.vimrc` 配置文件中加入一行配置：
```
let g:airline_theme="luna"
```
最后，我们重新打开 Vim 就可以看到如下的状态栏样式, 相比原来的要好看的多:
![image.png](https://cdn.nlark.com/yuque/0/2020/png/502915/1597741119944-dda426b8-ba84-4c21-910c-0fd3285ad369.png#averageHue=%233a414e&height=54&id=Y7KV6&originHeight=108&originWidth=3797&originalType=binary&ratio=1&rotation=0&showTitle=false&size=116130&status=done&style=stroke&title=&width=1898.5)

## 再见，HTML 标签

我刚开始学习编程的时候，是在 Windows 的记事本上输入 HTML 标签，那种输入速度现在来看真的是蜗牛。在 Vim 中如何快速输入 HTML 标签呢？有一个插件叫做 `vim-emmet`, 请按照上文中介绍的包管理器安装它。

在安装之后我们就可以快速输入 HTML 标签了，比如:
```
html:5
```
然后按下 `ctrl+y` , 接着输入 `,` （英文输入法下的逗号）,就可以触发了，自动生成如下的标签模板:
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title></title>
</head>
<body>

</body>
</html>
```
具体的 Emmet 插件的语法，可以参考 [官方文档](https://docs.emmet.io/abbreviations/syntax/)。

## 代码跳转-Ctags

使用 CTags 可以实现代码的跳转，首先安装:

```shell
yum install ctags
```
在需要的目录下，生成 tags 文件:

```shell
ctags -R
```
跳转到光标定位的 tag，使用`CTRL+]`，跳回来使用 `CTRL+o`, 列出所有可能 `:ts`。

## 我的 vimrc

下面我留下我的 vim 配置文件，供大家参考:
```
set ts=4
set nu
set background=dark
set t_Co=256
syntax on
set encoding=utf-8
colorscheme gruvbox
filetype on
let g:airline_theme="luna"
:set fillchars+=vert:\|
highlight VertSplit ctermbg=999 ctermfg=230

call plug#begin('~/.vim/plugged')
" 下面这些是插件，可以从 Github 找
Plug 'fatih/vim-go'
Plug 'ycm-core/YouCompleteMe'
Plug 'preservim/nerdtree'
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'davidhalter/jedi-vim'
Plug 'tiagofumo/vim-nerdtree-syntax-highlight'
Plug 'mattn/emmet-vim'
call plug#end()

let g:NERDTreeFileExtensionHighlightFullName = 1
let g:NERDTreeExactMatchHighlightFullName = 1
let g:NERDTreePatternMatchHighlightFullName = 1
```