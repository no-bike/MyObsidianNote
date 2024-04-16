#### 设置用户名和邮箱
```
git config --global user.name "YOUR USERNAME"
git config --global user.email "YOUR EMAIL"
```
这两行代码输入后没有任何输出即为设置成功
**可通过如下代码检查是否设置成功**
```
git config user.name
git config user.email
```
这两行代码会输出你的git用户名和email

#### 创建版本库 repository
windows系统为了避免奇怪问题请务必让本版本库所有父目录都是英文

**创建空目录**
```
mkdir learngit
cd learngit
pwd       //用于查看自己当前的位置
```
随后，通过
```
git init
```
指令，将learngit初始化成本地git仓库
文件中会出现一个 .git 目录，用于存储版本库，不了解不要动
如果没看到.git目录可以使用`ls -ah`查看隐藏目录

假设我们写了一个learngit.txt文件
通过下面两步可以将该文件提交到库中
1. 添加到暂存区
```
git add learngit.txt
```
add可以将很多文件添加到暂存区，在commit的时候统一提交
2. 提交
```
git commit -m "Write a learngit.txt"
```
-m后面的字符串是对该次提交的说明，最好有意义

#### 版本控制

**查看仓库当前状态**
```
git status
```
若修改过文件，通过该指令会把修改过的文件列出，如`modified: learngit.txt`，在提交到暂存区之前会将其放在工作区

如果想要**查看具体修改**了什么内容
```
git diff
```


##### 版本回退

**查看当前分支提交记录**
```
git log
```
若觉得版本消息过于冗杂，可使用
```
git log --oneline
```
以一行的方式展示提交记录

在Git中，用`HEAD`表示当前版本，也就是最新提交，上一个版本是`HEAD^`,上上一个版本是`HEAD^^`
如果要回退到上一个版本
```
git reset --hard HEAD^
```
即可，这时输入`git log`只能看到从最初到当前版本的提交记录
当然，如果回退以后还想再返回没回退的版本，只需要那个版本的**版本号**
如果找不到了，可以使用
```
git reflog
```
**查看**自己的**之前**自己的每一步**操作**及操作版本的版本号

**工作区与暂存区**概念如下：
工作区就是除了.git以外在这个仓库里我们能看到的其他所有文件及文件夹
![[Pasted image 20240413104351.png]]
![[Pasted image 20240413104454.png]]

##### 注意
`git commit` 指令只会把暂存区里的文件提交到版本库里
如果对一个文件修改第一次后add进暂存区，又第二次更改了这个文件，
此时`git commit`只会提交第一次修改的文件
解决方案有两个，
一是`add`->`commit`->`add`->`commit`，把两次修改作为两个版本
二是`add`->`add`->`commit`在暂存区合并两次修改以后作为一个版本提交

**PS：有原文件以后后续的更改提交是以补丁的形式而不是文件**

##### 撤销修改

若文件还在**工作区**中
```
git checkout -- learngit.txt
```
将文件在工作区中的修改全部撤销
如果在暂存区中有该文件之前的版本，则会撤销到暂存区中的版本
若没有，则会撤销回当前版本库中的版本
**`--`绝对不能漏掉，不然会切换到另一个分支

若文件在**暂存区**中
有两个指令让暂存区文件返回工作区
```
git reset HEAD learngit.txt
```
这个指令会将暂存区的`learngit.txt`文件返回到工作区中，如果工作区中对该文件进行了其他修改，则会将两个**修改合并**

```
git restore --staged learngit.txt
```
这个指令会使暂存区中的文件**覆盖**掉工作区的`learngit.txt`修改，不会保留工作区对该文件的任何修改


git 2.44.0 版本
丢弃工作区更改的命令如下：
`git restore <file>
撤销暂存区提交的命令如下：
`git restore --staged <file>
上面指令依旧可用


#### 远程仓库

在github创建仓库后会有提示喵~
比如下方，在本地仓库里连接了远程仓库，并命名为origin
```
git add remote origin git@github.com:no-bike/MyObsidianNote
```

其中，no-bike为github用户名

查看已连接的远程仓库及命名
```
git remote -v
```

删除远程库
以删除origin为例
```
git remote rm origin
```

