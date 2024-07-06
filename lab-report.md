



# 哈尔滨工业大学CS33503数据库系统实验报告

姓名：曲本磊

学号：2022112479

班级：2203501

专业：数据科学与大数据技术

学期：2024春

## 声明

本人承诺该实验全部由本人独立完成，没有抄袭他人代码。若被证实本人存在抄袭现象或为他人抄袭提供帮助，本人愿意承担全部责任（包括但不限于扣分、取消考试资格、上报学部等）。

## 目录

[TOC]

## 实验1：存储管理实验

### 任务1.1：磁盘存储管理器

在这一部分，你需要列举并介绍你实现的全部方法。你需要按照方法所在的类进行组织。在介绍每个方法的具体实现时，需要包含以下内容：

1. 方法的声明。给出方法的声明（注意：是方法声明，不是方法定义）。如果这个方法是你自己声明的，请说明它的功能以及为何要声明这个方法，何时调用这个方法。
2. 方法实现思路。根据方法实现的难度，可以采用不同的介绍形式。对于简单的方法，简要介绍方法的实现思路即可。对于复杂的方法，如果执行过程非常复杂，可以借助流程图或伪代码进行介绍。
3. 方法实现难点。如果你在实现这个方法的过程中遇到了较大的困难，不妨介绍一下你遇到的是什么困难，你最终的解决办法是什么。

#### DiskManager类的实现

```c++
void DiskManager::write_page(int fd, page_id_t page_no, const char *offset, int num_bytes);
```
将页面写入到磁盘中，需要使用lseek()函数定位文件头，利用句柄fd和偏离量。再通过write（）函数写入。

注意错误类型要选择正确。

```c++
void DiskManager::read_page(int fd, page_id_t page_no, char *offset, int num_bytes);
```
将页面写入到磁盘中，需要使用lseek()函数定位文件头，利用句柄fd和偏离量。再通过read（）函数写入。

注意错误类型要选择正确。

```c++
void DiskManager::create_file(const std::string &path) ;
```
这段C++代码是用来打开或创建一个文件的。它使用了`open`函数，这是一个在POSIX兼容的操作系统（如Linux和macOS）中常用的系统调用，用于打开和可能创建一个文件。

`open`函数的第一个参数是文件的路径，这里通过`path.c_str()`获取，其中`path`是一个`std::string`类型的变量，表示文件的路径。`c_str()`函数用于获取`std::string`的C风格字符串表示，因为`open`函数需要一个C风格的字符串作为路径参数。

第二个参数是一个位掩码，用于指定文件的打开方式和行为。在这个实现中，使用了`O_CREAT`和`O_WRONLY`两个选项的按位或（`|`）组合。`O_CREAT`表示如果指定的文件不存在，则创建它；`O_WRONLY`表示文件将以只写模式打开。这意味着如果文件已经存在，这段代码将打开它以便写入；如果文件不存在，将创建一个新文件，然后以只写模式打开。

```c++
void DiskManager::destroy_file(const std::string &path);
```

使用unlink函数来删除文件，但需要注意的是，如果指定的文件不存在，或者调用进程没有足够的权限来删除该文件，unlink函数将失败，并返回`-1`。成功执行时，它会返回`0`。
。
```c++
int DiskManager::open_file(const std::string &path);
```

这段代码是一个C++项目中的一部分，用于管理磁盘上的文件。具体来说，它定义了一个名为DiskManager的类，该类中的open_file方法用于打开一个指定路径的文件。这个方法首先检查给定路径的文件是否存在，然后检查该文件是否已经被打开，最后如果文件未打开，它会尝试打开文件，并更新内部的文件描述符映射。

首先，open_file方法接收一个std::string，表示要打开文件的路径。方法内部首先调用is_file(函数来检查路径指向的文件是否存在。is_file函数使用stat宏来判断路径是否指向一个常规文件。如果文件不存在，open_file方法会抛出一个FileNotFoundError异常，并返回`-1`。

接下来，open_file方法检查path2fd_（一个从文件路径到文件描述符的映射）中是否已经包含了给定的路径，以判断文件是否已经被打开。如果文件已经打开，方法会抛出一个InternalError异常，并返回`-1`。

如果文件未打开，open_file方法会使用open系统调用尝试以读写模式（O_RDWR）打开文件。[`open`](vscode-file://vscode-app/d:/vs%E9%85%8D%E7%BD%AE/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "../../../../../usr/include/x86_64-linux-gnu/bits/fcntl2.h")函数的第一个参数是文件路径的C风格字符串表示，可以通过std::string的c_str方法获得。成功打开文件后，open函数会返回一个文件描述符，这是一个非负整数，用于标识打开的文件。

最后，open_file方法会更新两个映射：path2fd_和fd2path_。path2fd_映射用于记录文件路径到文件描述符的映射，而fd2path_映射则记录文件描述符到文件路径的映射。这样做可以方便地通过文件路径或文件描述符查找对应的另一项。方法最终返回打开的文件的文件描述符。

```c++
void DiskManager::close_file(int fd);
```

将path2fd_和fd2path_两个映射里该文件删除，再使用close。


#### 实验发现

基础linux磁盘操作。


### 任务1.2：缓冲池替换策略

#### LRUReplacer类的实现

在每个函数内部，首先使用了C++17的`std::scoped_lock`来自动管理互斥锁latch_。`std::scoped_lock`是一个作用域锁，它在构造时自动上锁，并在析构时自动解锁，从而避免了死锁的发生并保证了线程安全。

```c++
bool LRUReplacer::victim(frame_id_t* frame_id);
```

这段代码是实现最近最少使用（LRU）替换策略的一部分，用于在缓存管理中选择一个页面进行淘汰。LRU策略的核心思想是淘汰最长时间未被访问的页面，因为这样的页面在未来被访问的可能性较小。

首先，LRUReplacer::victim()函数接收一个指向frame_id_t类型的指针frame_id作为参数，该函数的目的是找到并返回一个被淘汰的页面的标识符。

接下来，函数检查当前是否有可以被淘汰的页面，这是通过调用Size()函数来完成的，该函数返回LRUlist_的大小，即当前缓存中页面的数量。如果Size()大于0，表示有页面可以被淘汰。

如果有可被淘汰的页面，代码首先从LRUlist_中获取最后一个元素，即最久未被访问的页面，然后从LRUhash_中移除该页面。LRUlist_是一个双向链表，用于按访问顺序存储页面，而LRUhash_是一个哈希表，用于快速查找页面是否存在于缓存中。最后，将被淘汰的页面的标识符赋值给`*frame_id`，并返回`true`表示成功找到并淘汰了一个页面。

如果没有可被淘汰的页面，即LRUlist_为空，函数将frame_id设置为`nullptr`并返回`false`，表示没有页面被淘汰。

```c++
void LRUReplacer::pin(frame_id_t frame_id);
```

`pin`函数的目的是将指定的帧（frame）标记为“固定”的，意味着这个帧不应该被替换。

如果函数在未固定列表和哈希表中找到了这个frame_id，则会将其从未固定列表与哈希表中删除。

```c++
void LRUReplacer::unpin(frame_id_t frame_id);
```

如果未固定列表和哈希表中没有该frame_id，则将其添加进去。
#### 实验发现

课上讲的LRU策略，在这一任务中实现。


### 任务1.3：缓冲池管理器

#### BufferPoolManager类的实现

```c++
bool BufferPoolManager::find_victim_page(frame_id_t* frame_id);
```

#### 实验发现

内容

### 任务2.1：记录操作


### 任务2.2 记录迭代器

## 实验2 索引管理器

### 任务1 B+树的查找

### 任务2 B+树的插入

### 任务3 B+树的删除

### 任务4 B+树索引并发控制
