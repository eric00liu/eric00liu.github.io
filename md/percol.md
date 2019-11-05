---
title: percol是一个交互式的grep工具
date: 2016/3/12 11:02:44
tags:
- grep
- linux命令
- 效率
---
percol是一个交互式的grep工具, github主页在此：https://github.com/mooz/percol
grep是linux中使用率颇高的一个命令，主要是作用是从stdin或是文件中接收输入，在其中搜索你想找的关键词。
先来回忆一下grep是如何工作的, 例如
#### 找出sys.log文件中所有包含字符串’aaa’的所有行：    

```
grep 'aaa'  sys.log
```
该命令执行后，会将所有包含’aaa’的行都打印到屏幕上。

percol号称交互式grep工具，那么它的使用流程不太一样。
如果想在sys.log文件搜索’aaa’字符串，直接执行命令：
```
percol sys.log
```

便进入percol编辑状态，此时再输入’aaa’, 会列出包含’aaa’的所有行，如果想搜索其他字符串，则只需清空编辑行，重新输入待搜索的字符串即可。
#### grep从管道中获取输入，例如利用ps和grep来查nginx进程：
```
ps aux | grep nginx

```
如果用percol来做这件事，那么命令是这样的
```
ps aux | percol
```
照例进入percol编辑状态，此时再输入nginx，便列出所有nginx进程。
到这里，以上两个例子只是最简单的场景应用。

## 用percol来增强历史命令搜索
如果你经常用ctrl+r来搜索之前执行过的命令，那么一定要试试percol增强版
打开 ~/.zshrc，将下面的代码放入文件里
```
function exists { which $1 &> /dev/null }if exists percol; then
function percol_select_history() {        local tac
    exists gtac && tac="gtac" || { exists tac && tac="tac" || { tac="tail -r" } }
    BUFFER=$(fc -l -n 1 | eval $tac | percol --query "$LBUFFER")
        CURSOR=$#BUFFER         # move cursor
        zle -R -c               # refresh
}

zle -N percol_select_history
bindkey '^R' percol_select_history

fi

```
是之立即生效 
```
$ source ~/.zshrc
```
这时再试试 ctrl+r，是不是很爽，反正是把我爽到了。
个人感受：进入percol的编辑状态后，使用体验跟windows系统里的everything很像。

## 安装percol
### PyPI自动安装
```
$ sudo pip install percol

```
### 手动安装
先将代码下载到任意地方
```
$ git clone git://github.com/mooz/percol.git
$ cd percol

```

然后执行命令安装

```
$ sudo python setup.py install

```



