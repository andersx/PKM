# Cygwin 配置

配置方法转载自[Pargmatistic Guy][1]

什么是[Cygwin][2]？Cygwin能让你在Windows环境中使用Linux shell工具。由于Linux和Windows是两类不同的操作系统，所以[安装Cygwin][3]之后，我们还要做一番配置，方便使用。

要解决的问题

- 一些 Bug ：
	- [x] 中文乱码
	- [ ] 文件权限问题
	- vi 配置

* 可选优化：
	- [x] 高亮显示
	- [x] 右键粘贴
	- [x] 光标按单词移动
	- [x] 自动补全不区分大小写

看看实际效果

![final][4]
## 具体操作
### 1. 中文乱码
Windows命令的输出中文乱码，原因是Windows命令输出的编码是GBK。cygwin控制台Mintty的编码缺省是UTF-8。在命令窗口的标题的右键菜单上有[options]项，把其中的的选项的 [Text] 把编码改成GBK即可。

![charecter][4]

### 2. 文件权限问题
**现象**
Windows的文件的cygwin下没有权限：

```
	$ rm foo.txt
	error: open("foo.txt"): Permission denied
	error: unable to index file foo.txt
	$ ll foo.txt
	----------+ 1 Jerry None 486 Dec 24 14:16 foo.txt
```
文件的权限显示的是----------+，没有读写的权限。

**解决方法**
编辑/etc/fstab，在末尾加上下面的一行：
```
	none /cygdrive cygdrive binary,noacl,posix=0,user 0 0
```
关闭所有cygwin进程，再重启cygwin命令行。
显示文件权限已经正常-rw-r--r--：
```
	$ ll foo.txt
	-rw-r--r-- 1 Jerry None 486 Dec 24 14:16 foo.txt
```
注意！ 如果改了/etc/fstab但是没有生效，可以重启一下机器！
### 3. vi配置

在${HOME}/.vimrc文件中加上： # 没有.vimrc文件就新建。

```
	set number
	set hlsearch
	set fileencoding=utf-8
	set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
	 
	set nocompatible
	set backspace=indent,eol,start
	 
	syntax enable
```
说明：

syntax enable：打开语法高亮。cygwin的vi缺省没有打开。
set nocompatible和set backspace：配置backspace键，缺省backspace不起作用。
set fileencoding和set fileencodings：缺省文件编码和自动识别文件编码顺序
set number：显示行号
set hlsearch：搜索到内容高亮
### 4. 高亮显示
调整${HOME}/.bashrc文件，把注释掉别名打开：

```
	alias df='df -h'
	alias du='du -h'
	 
	alias whence='type -a'                        # where, of a sort
	alias grep='grep --color'                     # show differences in colour
	alias egrep='egrep --color=auto'              # show differences in colour
	alias fgrep='fgrep --color=auto'              # show differences in colour
	 
	alias ls='ls -h --color=tty'                 # classify files in colour
	alias dir='ls --color=auto --format=vertical'
	alias vdir='ls --color=auto --format=long'
	alias ll='ls -l'                              # long list
	alias la='ls -A'                              # all but . and ..
	alias l='ls -CF'                              #
	alias wch='which -a'
```
这样调整后，可以ls、grep、dir输出彩色显示。
另外加上命令的-h选项，这样文件大小以K、M、G显示，方便人阅读。
git输出（比如log、status）彩色显示，使用下面的命令配置：
	
	git config --global color.ui auto
### 5. 右键粘贴
还是Mintty的选项中

![right][6]
### 6. 自动补全不区分大小写
~/.bashrc文件中添加：
```
shopt -s nocaseglob
```
~/.inputrc文件中添加：
```
set completion-ignore-case on
```
### 7. 光标按单词移动/删除
想达到`Ctrl-Arrow Keys`, `Ctrl-Backspace`, `Ctrl-Delete`的效果
inputrc文件中添加：

```
	# Ctrl+Left/Right to move by whole words
	"\e[1;5C": forward-word
	"\e[1;5D": backward-word
	 
	# Ctrl+Backspace/Delete to delete whole words
	"\e[3;5~": kill-word
	"\C-_": backward-kill-word
```
## 使用Cygwin的一些技巧: 
### 1. 路径转换
cygwin的路径和Windows的路径表示不一样。
要注意的是，cygwin下的cd命令可以直接使用Windows的路径表示
```
	$ cd 'C:\Windows\System32\drivers\etc
```
cygwin中有cygpath命令来完成转换

```
	$ cygpath -au 'C:\Windows\System32drivers\etc'
	$ cygpath -aw '/cygdrive/c/Windows/System32/drivers/etc'
```
### 2. 打开文件
在命令行下打开文件，可用Windows自带的系统资源管理器
```
	$ explorer $WPATH
```
也可以写成脚本，脚本命名成xpr，放到PATH上。
```
	#!/bin/bash
	 
	cygwin=false;
	case "`uname`" in
	    CYGWIN*) cygwin=true ;;
	esac
	 
	if [ "$1" = "" ]; then
	    XPATH=. # 缺省是当前目录
	else
	    XPATH=$1
	    if $cygwin; then
	        XPATH="$(cygpath -C ANSI -w "$XPATH")";
	    fi
	fi
	 
	explorer $XPATH
```
打开资源管理器目录并选中的脚本，可以这个脚本命名成xpf，放到PATH上。
```
	#!/bin/bash	 
	cygwin=false;
	case "`uname`" in
	    CYGWIN*) cygwin=true ;;
	esac
	 
	if [ "$1" = "" ]; then
	    XPATH=. # 缺省是当前目录
	else
	    XPATH=$1
	    if $cygwin; then
	        XPATH="$(cygpath -C ANSI -w "$XPATH")";
	    fi
	fi
	 
	explorer '/select,' $XPATH
```	
### 3. 盘符的链接
到D盘，要/cygdrive/d，可以新建符号链接/d，这样可以减少录入
```bash
	ln -s /cygdrive/c /c
	ln -s /cygdrive/d /d
	ln -s /cygdrive/e /e
```
### ４.输出到clipboard
Windows下使用clip命令
示例：
```
	echo Hello | clip 
	# 将字符串Hello放入Windows剪贴板
	 
	dir | clip
	# 将dir命令输出（当前目录列表）放入Windows剪贴板
	 
	clip < README.TXT   
	# 将readme.txt的文本放入Windows剪贴板
	 
	echo | clip 
	# 将一个空行放入Windows剪贴板，即清空Windows剪贴板
```
linux下使用xsel命令

[1]: http://oldratlee.com/post/2012-12-22/stunning-cygwin
[2]: http://cygwin.com/
[3]: http://cygwin.com/	
[4]: http://m3.img.libdd.com/farm5/d/2012/1222/14/25349F08D1AD96A31D21A4372CEBC17E_B500_900_475_328.PNG
[5]: http://m1.img.libdd.com/farm4/d/2012/1223/11/27AC746CC0F02E1F3FC5E487A9E3EF6B_B500_900_400_285.PNG
[6]: http://m1.img.libdd.com/farm5/d/2012/1222/14/131F0E35F9DDC37D1E84F738D943CF23_B500_900_400_285.PNG