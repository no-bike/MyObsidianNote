



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

这段代码是缓冲池管理器（BufferPoolManager）中用于寻找一个可以被替换的页面（victim page）的函数实现。在数据库系统中，缓冲池管理器负责管理内存中页面的缓存，以减少对磁盘的访问次数。当需要加载一个新页面到缓冲池中，但缓冲池已满时，就需要选择一个页面进行替换。

函数`find_victim_page`的主要逻辑分为两部分：

1. **检查空闲列表**：首先，函数检查`free_list_`，这是一个包含空闲帧（frame）的列表。如果这个列表不为空，说明缓冲池中还有空间，不需要替换页面。函数从`free_list_`中取出一个空闲帧的ID，赋值给`frame_id`参数，并从列表中移除该帧ID，然后返回`true`表示已找到一个可用的帧。

2. **使用LRU替换策略选择淘汰页面**：如果`free_list_`为空，表示缓冲池已满，需要淘汰一个页面。这时，函数使用`replacer_`（一个实现了LRU替换策略的对象）的`victim`方法来选择一个被淘汰的帧ID。如果成功找到，函数遍历`page_table_`（一个映射页面ID到帧ID的哈希表）来找到对应的页面ID。找到后，检查该页面是否被修改过（即`is_dirty_`标志为`true`），如果是，则将页面的内容写回磁盘。最后，从`page_table_`中移除该页面的条目，并返回`true`表示成功找到并处理了一个被淘汰的页面。

整个函数的返回值表示是否成功找到一个可以被替换的帧。如果在空闲列表中找到了帧，或者通过LRU替换策略成功选择了一个被淘汰的页面，函数返回`true`；如果两者都没有成功，返回`false`。

这个函数体现了数据库系统中页面替换策略的实现，特别是在缓冲池已满时如何选择一个页面进行替换，以及如何处理被替换页面的数据（比如需要将修改过的数据写回磁盘）。

```c++
void BufferPoolManager::update_page(Page* page, PageId new_page_id, frame_id_t new_frame_id);
```

这段代码是在一个名为`BufferPoolManager`的类中定义的`update_page`函数，其目的是更新缓冲池中的一个页面。在数据库系统中，缓冲池管理器负责管理内存中页面的缓存，以减少对磁盘的访问次数。这个函数处理三个主要任务：

1. **处理脏页**：如果传入的页面（`page`）被标记为脏（即`is_dirty_`为`true`），这意味着页面的内容自从被加载到缓冲池以来已经被修改过。在这种情况下，需要将页面的内容写回磁盘，以确保磁盘上的数据是最新的。这通过调用`disk_manager_`的`write_page`方法完成，之后将页面的`is_dirty_`标志重置为`false`，表示页面现在是干净的。

2. **更新页面表**：页面表（`page_table_`）是一个映射，它将页面ID映射到帧ID。更新页面表涉及两个步骤：首先，将新的页面ID（`new_page_id`）映射到新的帧ID（`new_frame_id`），这通过直接在`page_table_`中插入或更新条目来完成。然后，从页面表中删除旧的页面ID，这通过调用`erase`方法并传入旧页面ID完成。

3. **重置页面数据并更新页面ID**：最后，函数通过调用`disk_manager_`的`read_page`方法，使用新的页面ID从磁盘读取页面内容到`page`的数据区域。这相当于用新页面的内容替换了当前页面的内容。然后，更新`page`的页面ID为新的页面ID。

这个函数的设计考虑到了数据库系统中页面生命周期管理的需要，特别是在页面内容更新或页面替换场景中的处理。通过将脏页写回磁盘、更新页面映射表以及重置页面内容，`update_page`函数确保了缓冲池中的数据与磁盘上的数据保持一致，同时也支持了页面的有效管理和替换策略。

```c++
Page* BufferPoolManager::fetch_page(PageId page_id);
```
这段代码是BufferPoolManager类中的fetch_page函数，它的主要作用是从缓冲池中获取一个指定ID的页面。如果页面已经在缓冲池中，它会直接返回该页面；如果不在，它会尝试找到一个可以被替换的页面（即所谓的victim page），然后从磁盘中读取所需页面的数据到这个被替换的页面中。

首先，函数通过std::scoped_lock创建了一个作用域锁lock，以确保在操作过程中缓冲池的状态不会被其他线程改变，保证了线程安全。

接下来，函数尝试在page_table_中查找是否存在目标页面ID。如果找到，说明目标页面已经在缓冲池中，此时会增加该页面的固定计数（pin_count_），并通过replacer_的pin方法标记该页面为已固定，最后返回该页面的指针。

如果在page_table_中没有找到目标页面，函数会尝试通过find_victim_page方法获取一个可用的帧ID。如果find_victim_page)返回false，说明没有可用的帧，函数返回nullptr.

在成功获取到一个可用的帧后，函数会检查这个帧中当前存储的页面是否为脏页（即is_dirty_为`true`）。如果是脏页，需要先将其写回磁盘，以保证数据的一致性。然后，从page_table_中移除这个页面的记录，并将新页面的ID与帧ID关联起来。

之后，函数通过disk_manager_的read_page方法从磁盘读取目标页面的数据到新页面中，并设置新页面的ID和固定计数（pin_count_)。最后，通过replacer_的pin方法标记新页面为已固定，并返回新页面的指针。

```C++
bool BufferPoolManager::unpin_page(PageId page_id, bool is_dirty);
```

这段代码是C++中的一个方法实现，属于`BufferPoolManager`类。它的主要功能是对缓冲池中的页面进行“解钉”操作。在数据库管理系统中，缓冲池管理器负责在内存和磁盘之间高效地交换数据页。"解钉"（unpin）是指将页面从缓冲池中释放，使其可以被替换出去。这个过程通常在页面不再被访问时进行。

1. **锁定**: 方法首先通过`std::scoped_lock`锁定一个互斥量`latch_`，以确保线程安全。这种锁会在当前作用域结束时自动释放，避免了忘记解锁的风险。

2. **查找页面**: 接下来，方法尝试在`page_table_`中查找给定的`page_id`。如果找不到，说明请求的页面不在缓冲池中，因此方法返回`false`。

3. **处理页面**: 如果页面存在于`page_table_`中，方法会获取对应的帧ID，并通过这个帧ID找到实际的页面对象。然后，检查页面的`pin_count_`（钉住计数）：
   - 如果`pin_count_`已经为0，表示页面已经是解钉状态，直接返回`false`。
   - 如果`pin_count_`大于0，表示页面被钉住，需要将其减一。如果减一后`pin_count_`变为0，表示页面现在可以被替换，此时会调用`replacer_`的`Unpin`方法来更新替换策略。

4. **更新脏页标志**: 最后，根据传入的`is_dirty`参数更新页面的`is_dirty_`标志。如果页面在被访问期间被修改了，`is_dirty`应该为`true`，这样在页面被替换出缓冲池时，系统会知道需要将其写回磁盘。

整个方法通过返回`true`或`false`来指示操作是否成功。这个过程对于数据库的性能至关重要，因为它影响到数据的读写效率以及缓冲池的利用率。

```C++
Page* BufferPoolManager::new_page(PageId* page_id);
```
这段代码是C++中BufferPoolManager类的new_page方法的实现，用于在缓冲池中分配一个新页面。这个过程涉及到多个步骤，包括从磁盘管理器分配新的页面ID、查找或淘汰一个现有的帧来存放新页面、将被淘汰页面的数据写回磁盘（如果该页面被修改过）、初始化新页面的内容，并最终返回这个新页面的指针。

1. **锁定**: 方法开始时，使用std::scoped_lock锁定互斥量latch_，确保线程安全。
    
2. **分配新的页面ID**: 通过调用磁盘管理器的allocate_page方法，为新页面分配一个唯一的页面ID。
    
3. **检查所有帧是否被钉住**: 遍历缓冲池中的所有页面，检查是否所有页面的pin_count_都不为0。如果是，说明所有帧都被使用中，无法分配新页面，因此返回nullptr。
    
4. **查找或淘汰帧**: 调用find_victim_page方法尝试找到一个可用的帧。如果找不到，同样返回nullptr
    
5. **处理被淘汰的页面**: 如果找到的帧中有页面（即被淘汰的页面），且该页面被修改过is_dirty_为true，则将其数据写回磁盘，并将is_dirty_标志重置为false。
    
6. **初始化新页面**: 将新分配的页面ID赋给被选中的帧，设置pin_count_为1（表示页面被钉住），并通过replacer_的pin方法更新替换策略。同时，更新page_table_以反映页面ID与帧ID的映射关系。
    
7. **清零新页面数据**: 使用memset将新页面的数据区域清零，确保不含有旧数据。
    
8. **写回新页面数据**: 将新页面的数据（此时为全0）写回磁盘，确保磁盘上的数据是最新的。
    
9. **返回新页面**: 最后，返回新页面的指针。

```c++
bool BufferPoolManager::delete_page(PageId page_id);
```

这段代码是C++中的一个函数实现，属于`BufferPoolManager`类。其主要目的是从缓冲池管理器中删除一个指定的页面（Page）。这个过程涉及到多个步骤，包括查找页面、检查页面的状态、将页面数据写回磁盘、更新内部数据结构等。下面是对这个函数的逐步解析：

1. **加锁**：使用`std::scoped_lock`自动管理互斥锁`latch_`，确保在函数执行期间，当前对象的状态不会被其他线程修改，从而保证线程安全。
    
2. **查找页面**：首先，在`page_table_`中查找给定的`page_id`。如果这个`page_id`不存在于`page_table_`中，说明没有需要删除的页面，函数直接返回`true`。
    
3. **检查pin_count**：如果找到了页面，会检查其`pin_count_`。`pin_count_`表示页面被引用的次数。如果`pin_count_`不为0，说明还有其他地方正在使用这个页面，因此不能安全地删除，函数返回`false`。
    
4. **写回磁盘**：如果页面可以被删除（即`pin_count_`为0），函数会检查页面的`is_dirty_`标志。如果页面被修改过（`is_dirty_`为`true`），则需要将页面的内容写回磁盘，以保证数据的一致性。写回磁盘后，将`is_dirty_`标志重置为`false`。
    
5. **更新内部数据结构**：接下来，函数会更新页面的元数据，将页面编号`page_no`设置为`INVALID_PAGE_ID`，并将`pin_count_`重置为0。这表示页面已经不再被缓冲池管理。
    
6. **从页表中删除页面**：从`page_table_`中删除对应的`page_id`条目，这样就从管理结构中移除了这个页面。
    
7. **回收页面**：将页面对应的帧编号`frame_id`加入到`free_list_`中，表示这个帧现在是空闲的，可以被后续的页面请求使用。
    
8. **返回成功**：完成上述所有步骤后，函数返回`true`，表示页面已经成功删除。

#### 实验发现

将页面提前读入缓冲池中，可以提高I/O效率。
通过LRU策略决定将满的缓冲池中选出一个页面淘汰以读入新的需要的页面。

### 任务2.1：记录操作

#### RmFileHandle类的实现

```C++
std::unique_ptr<RmRecord> RmFileHandle::get_record(const Rid& rid, Context* context) const {}
```
这段代码是C++编写的，用于处理记录管理系统中的文件操作。主要功能是从文件中获取指定的记录（RmRecord），这个过程涉及到两个主要的步骤：首先，通过页面编号（page_no）获取页面句柄（page_handle）；其次，使用这个页面句柄来初始化一个记录对象，并将其返回。

在get_record函数中，首先调用fetch_page_handle函数来获取指定页面的页面句柄。这个函数接受一个页面编号作为参数，首先检查这个编号是否有效（即是否在文件的页面数范围内），如果无效则抛出`PageNotExistError`异常。如果页面编号有效，它会使用缓冲池管理器（buffer_pool_manager_）来获取对应的页面，并以此创建一个RmPageHandle

一旦获取到页面句柄，get_record函数接着创建一个RmRecord对象，这个对象用于存储记录的数据和大小。在将数据复制到RmRecord对象之前，函数会检查位图（Bitmap），确保指定的槽位（slot_no）中确实有记录存在。如果位图显示该槽位为空，则抛出RecordNotFoundError)异常。如果记录存在，函数会使用memcpy函数将数据从页面句柄指向的槽位复制到`RmRecord`对象的数据区，并设置记录的大小。

最后，函数返回这个初始化好的`RmRecord`对象的智能指针。这种设计使得函数的调用者不需要关心记录的具体存储和管理细节，只需要通过记录的标识（`Rid`）就可以获取到记录的内容。

```C++
Rid RmFileHandle::insert_record(char* buf, Context* context);
```

`insert_record`函数的目的是在文件的指定位置（由`Rid`对象指定的页面号和槽号）插入一条记录。首先，它检查指定的页面号是否超出了文件当前的页面数。如果是，那么它会调用create_new_page_handle函数来创建一个新页面。接着，通过fetch_page_handle函数获取目标页面的句柄。使用`memcpy`函数将记录数据从输入缓冲区复制到目标页面的指定槽位。之后，更新页面头部的记录数，并在位图中标记该槽位已被使用。最后，如果插入记录后页面变满（即页面的记录数达到每页最大记录数），则更新文件头部的第一个空闲页面号。

```c++
void RmFileHandle::delete_record(const Rid& rid, Context* context);
```

`delete_record` 函数的目的是删除一个指定的记录。它首先通过调用fetch_page_handle。接着，使用`Bitmap::reset`方法更新页面的位图，标记指定槽位（`slot_no`）的记录为已删除。然后，减少页面头（`page_hdr`）中的记录数（`num_records`）。如果页面从满变为未满（即记录数减少到页面最大记录数减一），则调用release_page_handle函数处理页面释放逻辑。

```c++
void RmFileHandle::update_record(const Rid& rid, char* buf, Context* context);
```

`update_record`函数的目的是更新文件中特定记录的内容。它接收三个参数：一个`Rid`类型的`rid`，它可能代表记录的唯一标识符；一个字符指针`buf`，指向要更新的新记录内容；一个`Context`类型的指针`context`，可能用于传递操作的上下文信息。

函数的实现分为两个主要步骤：

1. 首先，它调用fetch_page_handle函数，传入`rid`的页面编号（page_no），以获取该记录所在页面的句柄（RmPageHandle类型）。这个页面句柄包含了操作页面所需的所有信息。
2. 然后，它检查位图（`bitmap`），确认记录槽（`slot_no`）是否已设置。如果未设置，表示页面不存在，抛出PageNotExistError异常。如果记录槽已设置，它使用`memcpy`函数将新记录内容从`buf`复制到该槽位，大小为file_hdr_.record_size.

```C++
RmPageHandle RmFileHandle::fetch_page_handle(int page_no) const{}
```

fetch_page_handle函数的目的是获取指定页面编号的页面句柄。它接收一个整数page_no作为参数，表示要获取句柄的页面编号

```c++
RmPageHandle RmFileHandle::create_new_page_handle();
```

用于处理与“页面”相关的操作，这在数据库管理系统或文件系统中是一个常见的概念。具体来说，它定义了一个名为RmFileHandle::create_new_page_handle的成员函数，该函数的目的是在文件中创建一个新的页面，并返回一个与新页面相关联的页面句柄(RmPageHandle)。

首先，函数通过指定文件描述符fd_和一个无效的页面IDINVALID_PAGE_ID来初始化一个PageId结构体。这表明新页面尚未分配一个有效的页面编号。

接下来，它调用buffer_pool_manager_的new_page方法来实际创建一个新页面，这个方法会分配一个新的页面并返回一个指向该页面的指针。这里使用了缓冲池管理器，这是一种优化技术，可以减少对磁盘的直接访问，从而提高性能。

然后，函数初始化一个RmPageHandle对象page_handle，但初始时并没有关联到具体的页面上（因为new_page可能会失败）。如果new_page成功返回了一个非空指针，表示页面创建成功，那么函数会更新page_handle，使其指向新创建的页面，并初始化页面头部和位图等信息。

页面头部(page_hdr)包含了一些元数据，比如下一个空闲页面的编号和当前页面中记录的数量。这里将下一个空闲页面编号设置为RM_NO_PAGE，表示没有下一个空闲页面，同时记录数量设置为0，因为这是一个新页面。

最后，函数更新文件头部信息(file_hdr_)，如果这是文件中的第一个页面，它还会更新第一个空闲页面的编号。无论如何，它都会增加文件中页面的总数。

```c++
void RmFileHandle::release_page_handle(RmPageHandle&page_handle)
```
1. 判断file_hdr_中是否还有空闲页

     1.1 没有空闲页：使用缓冲池来创建一个新page；可直接调用create_new_page_handle()

    1.2 有空闲页：直接获取第一个空闲页
2. 生成page handle并返回给上层
#### 实验发现

实践了每一个页面头的结构与实现。
### 任务2.2 记录迭代器

#### RmScan类实现

```c++
RmScan(const RmFileHandle *file_handle) : file_handle_(file_handle)
```
初始化file_handle和rid.

```c++
void RmScan::next();
```

在一个文件中扫描并找到存放了记录的非空闲位置。这个过程通过两个主要的成员函数`next`和`is_end`来实现。

`next`函数的目的是找到文件中下一个存放了记录的非空闲位置，并用`rid_`成员变量来指向这个位置。`rid_`是一个结构体，包含了页号(`page_no`)和槽号(`slot_no`)，用于唯一标识文件中的一个位置。

函数首先进入一个`while`循环，循环条件是当前的页号小于文件头中记录的总页数(`num_pages`)。在循环内部，首先通过`file_handle_`的`fetch_page_handle`方法获取当前页号对应的页面句柄(`page_handle`)。然后，使用`Bitmap::next_bit`静态方法查找当前页面的位图中，从当前槽号开始的下一个设置为`true`的位，即下一个非空闲的记录位置。如果找到了这样的位置，即`rid_.slot_no`小于每页记录数(`num_records_per_page`)，函数就返回，`rid_`已经更新为下一个非空闲位置的标识。

如果当前页中没有找到非空闲位置，即所有记录都是空闲的，那么就将页号增加1，槽号设置为-1，表示从下一页的开始位置继续查找。如果增加后的页号等于或超过了总页数，表示已经扫描完整个文件，此时将`rid_`设置为特殊值`RM_NO_PAGE`和-1，表示没有更多的记录可以扫描，然后退出循环。

`is_end`函数用于判断是否已经到达文件的末尾，即是否已经扫描完所有的记录。这个函数非常简单，只需要检查`rid_.page_no`是否等于`RM_NO_PAGE`。如果是，表示没有更多的记录可以扫描，返回`true`；否则，表示还有记录可以扫描，返回`false`。

总的来说，这段代码通过`next`函数逐页逐记录地扫描文件，直到找到非空闲的记录或者扫描完整个文件。`is_end`函数则用于提供一个简单的方式来检查是否已经完成了扫描。

#### 实验发现

通过遍历rid的方式找到空闲或非空闲的位置。
## 实验2 索引管理器

### 任务1 B+树的查找

#### 结点内的查找

```cpp
class IxNodeHandle {
    // 结点内的查找
    int lower_bound(const char *target) const;
    int upper_bound(const char *target) const;
    bool leaf_lookup(const char *key, Rid **value);
    page_id_t InternalLookup(const char *key);
}
```

为了实现整个B+树的查找，首先需要实现B+树单个结点内部的查找。

需要实现以下函数：

- `int lower_bound(const char *target) const;`

	用于在当前结点中查找第一个大于或等于`target`的key的位置。
	这段代码是C++编写的，目的是在一个索引节点中查找第一个大于等于给定目标值(`target`)的键(`key`)的位置。这是在数据库索引管理中常见的操作，特别是在B树或B+树这类索引结构中。该函数属于`IxNodeHandle`类，这个类很可能是用来处理索引节点的。

	为了实现这个查找功能，代码采用了二分查找算法。二分查找是一种在有序数组中查找特定元素的高效算法，其时间复杂度为O(log n)，其中n是数组中元素的数量。这种方法通过比较数组中间元素与目标值的大小，来缩小搜索范围，直到找到目标值或搜索范围为空。

	在这段代码中，首先初始化了两个变量`low`和`high`，分别表示查找范围的起始和结束位置。`page_hdr->num_key - 1`表示当前节点中键的总数减去1，即最后一个键的位置，这是因为数组索引是从0开始的。

	接下来，进入一个`while`循环，循环条件是`low`小于等于`high`，表示当前的查找范围不为空。在循环内部，首先计算中间位置`mid`，然后使用`ix_compare()`函数比较中间位置的键与目标值`target`。`ix_compare()`函数很可能是一个自定义的比较函数，用于比较两个键的大小，考虑到索引可能涉及多种数据类型和长度，`ix_compare()`函数需要接受键的类型和长度作为参数。

	如果中间位置的键正好等于目标值，那么直接返回该位置。如果中间位置的键大于目标值，说明目标值应该在当前查找范围的左半部分，因此将`high`更新为`mid - 1`。反之，如果中间位置的键小于目标值，说明目标值应该在当前查找范围的右半部分，因此将`low`更新为`mid + 1`。



- `int upper_bound(const char *target) const;`

	用于在当前结点中查找第一个大于`target`的key的位置。
	为了实现这个功能，代码采用了二分查找算法。二分查找是一种高效的查找方法，特别适用于在有序集合中查找元素。它通过比较中间元素与目标值的大小，来逐步缩小搜索范围，直到找到目标元素或者搜索范围为空。

	在这段代码中，首先初始化了两个变量`low`和`high`，分别代表查找范围的起始和结束位置。这里的`low`初始化为1，而`high`初始化为`page_hdr->num_key - 1`，即当前节点中键的总数减去1。这是因为索引节点中的键是从1开始编号的，而不是从0开始。

	接下来，代码进入一个`while`循环，循环条件是`low`小于等于`high`，表示当前的查找范围不为空。在循环内部，首先计算中间位置`mid`，然后使用`ix_compare()`函数比较中间位置的键与目标值`target`。`ix_compare()`函数是一个自定义的比较函数，用于比较两个键的大小，它接受键的类型和长度作为参数，这是因为索引可能包含多种数据类型的键。

	如果中间位置的键正好等于目标值，那么直接返回`mid + 1`，因为我们要找的是第一个大于目标值的键。如果中间位置的键大于目标值，说明目标值应该在当前查找范围的左半部分，因此将`high`更新为`mid - 1`。反之，如果中间位置的键小于目标值，说明目标值应该在当前查找范围的右半部分，因此将`low`更新为`mid + 1`。

- `bool leaf_lookup(const char *key, Rid **value);`

	​用于叶子结点根据key来查找该结点中的键值对。值`value`作为传出参数，函数返回是否查找成功。是查找过程的入口点。它接受一个键（key）和一个指向Rid指针的指针（value）作为参数。函数的目标是在叶子节点中找到给定的键，并且如果找到了，就通过`value`参数返回对应的Rid。这个过程分为几个步骤：首先，使用lower_bound函数找到键的位置；然后，检查这个位置的键是否确实与给定的键匹配；如果匹配，就通过get_rid函数获取对应的Rid，并通过`value`参数返回。

- `page_id_t internal_lookup(const char *key);`

​		用于内部结点根据key来查找该key所在的孩子结点（子树）。

​		值value为Rid类型，对于内部结点，其Rid中的page_no表示指向的孩子结点的页面编号。而内部结点每个key右边的value指向的孩子结点中的键均大于等于该key，每个key左边的value指向的孩子结点中的键均小于该key。


#### B+树的查找

```cpp
class IxIndexHandle {
    // B+树的查找
    std::pair<IxNodeHandle *, bool> find_leaf_page(const char *key, Operation operation, Transaction *transaction,bool find_first = false);
    bool get_value(const char *key, std::vector<Rid> *result, Transaction *transaction);
}
```

- `std::pair<IxNodeHandle *, bool> find_leaf_page(const char *key, Operation operation, Transaction *transaction, bool find_first = false);`

​		用于查找指定键所在的叶子结点。
	函数的执行流程如下：

1. **获取根节点**：首先，通过`fetch_node`函数和根页面的编号（`file_hdr_->root_page_`）获取索引的根节点。
2. **向下查找**：然后，函数进入一个循环，不断向下遍历树，直到找到一个叶子节点。在每一步中，它使用当前节点的`internal_lookup`方法来找到下一个要遍历的子节点，并通过`fetch_node`获取该子节点。同时，它使用`buffer_pool_manager_`)来解除对当前节点页面的锁定（`unpin_page`），以便其他操作可以使用这些页面。
3. **返回叶子节点**：一旦找到叶子节点，循环结束，函数返回一个包含叶子节点指针和布尔值[`true`](vscode-file://vscode-app/d:/vs%E9%85%8D%E7%BD%AE/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "deps/googletest/googletest/include/gtest/internal/gtest-internal.h")的`std::pair`。

- `bool get_value(const char *key, std::vector<Rid> *result, Transaction *transaction);`

  用于查找指定键在叶子结点中的对应的值`result`。

	首先，`IxIndexHandle::get_value`函数通过调用`find_leaf_page`函数来定位目标键所在的叶子节点。`find_leaf_page`函数从根节点开始，逐层向下遍历B+树，直到找到包含目标键的叶子节点。这个过程中，如果当前节点不是叶子节点，则会根据目标键值查找下一层的节点，直到找到叶子节点为止。在查找过程中，使用了`buffer_pool_manager_`)来管理页面的引用，确保不再需要的页面被及时释放。

	找到叶子节点后，`get_value`函数接着调用`leaf->leaf_lookup`方法来在叶子节点中查找目标键的位置。`leaf_lookup`方法首先使用`lower_bound`函数确定目标键可能存在的位置，然后通过比较确定目标键是否真的存在于该位置。如果目标键存在，`leaf_lookup`会通过`get_rid`函数获取对应的Rid，并通过参数返回。

	最后，如果目标键存在，`get_value`函数会将找到的Rid添加到结果向量中，并返回`true`表示查找成功。如果目标键不存在，则直接返回`false`。在整个过程中，`std::scoped_lock`用于确保线程安全，防止在并发访问时出现数据竞争。

#### 实验发现

通过索引可以进行定向查找，沿着叶子节点可以顺序查找。
### 任务2 B+树的插入

#### 结点内的插入

```cpp
class IxNodeHandle {
    // 结点内的插入
    void insert_pairs(int pos, const char *key, const Rid *rid, int n);
    int insert(const char *key, const Rid &value);
}
```

- `void insert_pairs(int pos, const char *key, const Rid *rid, int n);`

​		用于在结点中的指定位置插入多个键值对。

​		该函数插入指定`n`个单位长度的键值对数组`(key,rid)`到结点中的指定位置`pos`。其中`key`为键数组的首地址，其每个单位长度为`file_hdr_->col_lens_[i]`。`rid`为值数组的首地址，其每个单位长度为`sizeof(Rid)`。这里内部存储结构是键数组和值数组连续存储，即键数组的后面存储了值数组。

首先，方法接收四个参数：`pos`表示插入的起始位置，`key`是指向键的指针，`rid`是指向记录标识符（Record Identifier）的指针，`n`表示要插入的键值对数量。方法的开始部分首先通过调用`get_size`函数获取当前节点中键的数量，以此来判断`pos`是否在合法范围内（即`pos`应该在0到当前键数量之间，包含两端）。如果`pos`不合法，方法直接返回，不执行任何插入操作。

如果`pos`合法，接下来的步骤是腾出空间来插入新的键值对。这通过从节点的末尾开始，将现有的键值对向后移动`n`个位置来实现，为新的键值对腾出空间。这一过程通过循环实现，循环中使用`set_key`和`set_rid`方法来设置移动后的键和记录标识符。

之后，方法进入实际的插入过程。通过另一个循环，将新的键值对插入到之前腾出的空间中。键的插入通过计算偏移量（`key + file_hdr->col_tot_len_ * i`）来实现，这里假设每个键的长度是固定的，并且通过文件头信息中的`col_tot_len_`字段来获取。记录标识符的插入则直接通过索引访问`rid`数组实现。

最后，更新节点中键的数量，即在原有的基础上加上新插入的键值对数量`n`。

- `int insert(const char *key, const Rid &value);`

   1. 查找要插入的键值对应该插入到当前节点的哪个位置

   2. 如果key重复则不插入

   3. 如果key不重复则插入键值对

   4. 返回完成插入操作之后的键值对数量

#### B+树的插入

```cpp
class IxIndexHandle {
    // B+树的插入
    page_id_t insert_entry(const char *key, const Rid &value, Transaction *transaction);
    IxNodeHandle *split(IxNodeHandle *node);
    void insert_into_parent(IxNodeHandle *old_node, const char *key, IxNodeHandle *new_node, Transaction *transaction);
}
```

- `page_id_t insert_entry(const char *key, const Rid &value, Transaction *transaction);`

​		用于将指定键值对插入到B+树。

​`insert_entry`函数是插入操作的入口点。它首先通过`find_leaf_page`函数找到应该插入新键值对的叶子节点。然后，它调用叶子节点的`insert`方法尝试插入键值对。如果插入成功，并且叶子节点因此变得已满，就需要分裂这个节点，并将新节点的一些信息插入到父节点中。这个过程可能会递归地向上延伸，直到根节点。最后，函数确保所有涉及的页面都被正确地“unpin”，以释放资源。

- `IxNodeHandle *split(IxNodeHandle *node);`

​		用于分裂结点。函数返回分裂产生的新结点。

​	首先，`split`函数接收一个需要分裂的节点`node`作为参数。函数的目标是将`node`中的键值对平均分配到`node`和一个新创建的节点`new_node`中。如果`node`是叶子节点，还需要更新叶子节点链表中的指针，确保叶子节点之间的顺序关系正确。

在分裂过程中，首先通过`create_node`函数创建一个新的节点`new_node`。然后，根据`node`是否为叶子节点执行不同的操作。如果`node`是叶子节点，将`new_node`也标记为叶子节点，并更新相关的指针，包括`prev_leaf`和`next_leaf`，以及文件头中的`last_leaf_`指针（如果需要的话）。这些操作确保了叶子节点之间的双向链表关系得到维护。

接下来，无论`node`是否为叶子节点，都会将`node`中的一半键值对移动到`new_node`中。这通过`insert_pairs`函数实现，该函数将指定数量的键值对插入到目标节点的指定位置。`insert_pairs`函数首先检查插入位置的合法性，然后为新插入的键值对腾出空间，并更新节点的键数量。

最后，如果`node`不是叶子节点，还需要更新`new_node`中所有子节点的父节点信息，以保持树结构的正确性。这通过maintain_child函数实现，该函数接收一个节点和子节点的索引，然后加载相应的子节点并更新其父节点信息。

- `void insert_into_parent(IxNodeHandle *old_node, const char *key, IxNodeHandle *new_node, Transaction *transaction);`

​		用于结点分裂后，更新父结点中的键值对。

​		这段代码是一个索引结构中插入新键值对的过程，特别是在B树或B+树这样的平衡树结构中处理节点分裂后的情况。整个过程主要关注在将一个新的键（`key`）和对应的记录标识符（`Rid`）插入到适当的父节点中，并且处理可能因为插入而导致的父节点分裂。这个过程是递归的，因为父节点分裂可能会一直传播到根节点。

1. **判断是否为根节点**：首先，代码检查当前要处理的旧节点（`old_node`）是否是根节点。如果是，那么会创建一个新的根节点（`new_root_node`），并更新根节点的页面编号。新根节点的父页面编号设置为无效（`INVALID_PAGE_ID`），表示它是树的最顶层。然后，将旧根节点作为新根节点的子节点插入。这一步骤确保了树的高度可以在必要时增加。
    
2. **获取父节点**：如果旧节点不是根节点，那么通过旧节点的父页面编号来获取其父节点（`parent`）。这一步是为了找到应该插入新键值对的节点。
    
3. **插入键值对**：无论是新创建的根节点还是通过父页面编号找到的父节点，都会在适当的位置插入新的键值对。插入位置是基于旧节点在父节点中的位置（`insert_pos`）来确定的，新键值对插入到这个位置之后。
    
4. **处理父节点分裂**：插入新键值对后，如果父节点的大小超过了其最大容量，就需要分裂父节点。分裂操作会产生一个新的节点（`parent_new_node`），并且需要将分裂产生的新键值对插入到父节点的父节点中，这可能会导致递归地分裂更高层的节点。每次分裂操作后，都需要对涉及的页面进行“unpin”操作，以表明这些页面的数据已经被修改并可以被写回磁盘。
    
5. **资源管理**：最后，无论是否发生了分裂，都会对父节点进行“unpin”操作，确保资源得到正确管理。

#### 实验发现


![[Pasted image 20240706231657.png]]

### 任务3 B+树的删除


#### 结点内的删除

```cpp
class IxNodeHandle {
    // 结点内的删除
    void erase_pair(int pos);
    int remove(const char *key);
}
```

学生需要实现以下函数：

- `void erase_pair(int pos);`

这个过程涉及到三个主要步骤：删除键、删除RID、更新节点中键值对的数量。

1. **删除键**：使用`memmove`函数来删除指定位置`pos`上的键。`memmove`函数的作用是将内存内容从一个位置复制到另一个位置，这里它被用来覆盖掉要删除的键。具体操作是将`pos + 1`位置开始的所有键向前移动一个键的位置，覆盖掉`pos`位置上的键。键的大小由`file_hdr->col_tot_len_`确定，表示每个键占用的字节长度。这样，从`pos`位置开始的所有键都向前移动了一个位置，实现了删除操作。
    
2. **删除RID**：注释掉的`memmove`行是原本用于删除RID的代码，但它被替换为了一个循环。这个循环通过`set_rid`函数将`pos + 1`位置及之后的所有RID向前移动一个位置，覆盖掉`pos`位置的RID。`get_rid`函数用于获取指定位置的RID，而`set_rid`则用于设置指定位置的RID。这种方法相比使用`memmove`可能更清晰地表达了删除RID的逻辑，尽管在性能上可能略有不同。
    
3. **更新节点的键值对数量**：通过`set_size(get_size() - 1)`更新节点中键值对的数量，即在原有数量的基础上减一。`get_size`函数返回当前节点中键值对的数量，`set_size`则用于更新这个数量。这一步骤是必要的，因为删除一个键值对后，节点中的总键值对数量自然应该减少。


#### B+树的删除

```cpp
class IxIndexHandle {
    // B+树的删除
    bool delete_entry(const char *key, Transaction *transaction);
    bool coalesce_or_redistribute(IxNodeHandle *node, Transaction *transaction = nullptr,bool *root_is_latched = nullptr);
    bool coalesce(IxNodeHandle **neighbor_node, IxNodeHandle **node, IxNodeHandle **parent, int index,Transaction *transaction, bool *root_is_latched);
    void redistribute(IxNodeHandle *neighbor_node, IxNodeHandle *node, IxNodeHandle *parent, int index);
    bool adjust_root(IxNodeHandle *old_root_node);
}
```

学生需要实现以下函数：

- `bool delete_entry(const char *key, Transaction *transaction);`

​		用于删除B+树中含有指定`key`的键值对。

​		首先，`delete_entry`函数是删除操作的入口点。它首先通过`find_leaf_page`函数找到包含要删除键值对的叶子节点。然后，它尝试在该叶子节点中通过调用`IxNodeHandle::remove`法来删除键值对。如果删除成功（即叶子节点的大小发生了变化），它会调用`coalesce_or_redistribute`函数来检查是否需要对树进行重组，以保持树的平衡。如果需要，它还会处理与事务相关的逻辑，比如在事务的`delete_page_set`中记录删除的页面。

`IxNodeHandle::remove`函数负责在特定的节点中删除键值对。它首先找到要删除的键值对的位置，如果找到了，就将其删除，并返回节点中剩余键值对的数量。

`find_leaf_page`函数用于从树的根节点开始，逐层向下查找，直到找到包含目标键的叶子节点。这个过程涉及到不断地获取子节点并解除对父节点的锁定，直到到达叶子节点。

`coalesce_or_redistribute`函数处理节点删除后的树重组逻辑。如果删除操作导致节点的键值对数量低于最小值，这个函数会尝试通过合并或重分配键值对来保持树的平衡。这包括找到节点的兄弟节点，并根据两个节点的键值对总数决定是合并节点还是重新分配键值对。

最后，`maintain_parent`函数确保在进行节点修改后，父节点中的键值对仍然正确地反映了子节点的最小键值。这是通过在父节点中查找子节点的位置，然后根据需要更新父节点中的键值对来实现的。

- `bool coalesce_or_redistribute(IxNodeHandle *node, Transaction *transaction = nullptr,bool *root_is_latched = nullptr);`

  用于处理合并和重分配的逻辑。函数返回是否有结点被删除（无论是`node`还是它的兄弟结点被删除）。传出参数`root_is_latched`记录根结点是否被上锁，该参数将在任务3使用，在本任务2中不使用。

​	包含了几个关键的操作：调整根节点（`adjust_root`、重新分配（`redistribute`）、合并（`coalesce`）、获取记录标识符（`get_rid`）和维护父节点（`maintain_parent`)
 调整根节点（`adjust_root`）

这个函数处理根节点的调整。如果根节点是内部节点且只有一个孩子，它会将这个孩子提升为新的根节点。如果根节点是叶子节点且为空，它会更新根页面编号为无效值。这两种情况都是在树的高度减小时发生的。

 重新分配

当一个节点的键值对数量低于最小值时，可以从它的兄弟节点借一个键值对。这个函数根据兄弟节点是前驱还是后继来决定如何移动键值对，并更新父节点中的相关信息。

 合并

如果一个节点和它的兄弟节点的键值对总数仍然低于两个节点的最小键值对数，这两个节点可以合并。`coalesce`函数将一个节点的所有键值对移动到它的兄弟节点中，并更新父节点中的信息。如果合并后的节点是叶子节点且是最右边的叶子节点，还需要更新文件头中的最后一个叶子节点的信息。

 获取记录标识符

这个函数根据给定的索引项（`Iid`）获取相应的记录标识符（`Rid`)

 维护父节点

当节点中的键值对发生变化时，可能需要更新父节点中的键值。这个函数遍历从当前节点开始向上的所有父节点，更新它们中的键值以保持树的正确性。

整体来看，这些函数共同工作以维护B+树的结构和平衡。通过调整根节点、重新分配键值对、合并节点以及维护父节点的键值，`IxIndexHandle`类确保了B+树在插入和删除操作后仍然保持有效和高效的结构。

- `bool coalesce(IxNodeHandle **neighbor_node, IxNodeHandle **node, IxNodeHandle **parent, int index,Transaction *transaction, bool *root_is_latched);`

​		将`node`向前合并到其前驱`neighbor_node`。函数返回`node`的父结点`parent`否需要被删除。

​		1. **合并操作的主要逻辑** ：首先，通过索引判断`neighbor_node`是否为node的前驱节点，如果不是，则交换两者，确保`neighbor_node`作为左节点，node作为右节点。然后，将node节点的键值对移动到`neighbor_node`中，并更新`node`节点孩子节点的父节点信息（通过调用`maintain_child`函数）。接着，释放和删除`node`节点，并从父节点中删除`node`节点的信息。最后，如果操作的是叶子节点且为最右叶子节点，需要更新`file_hdr_`的`last_leaf`属性。
    
2. **键值对的插入和删除**：`insert_pairs` 函数用于在指定位置插入键值对，首先检查位置的合法性，然后腾出空间并插入新的键值对，最后更新节点的键数量。`erase_pair` 函数用于删除指定位置的键值对，通过移动内存来覆盖要删除的键值对，并更新节点的键数量。
    
3. **合并或重分配的决策逻辑** ：首先判断节点是否为根节点，如果是，则调用`adjust_root`函数处理。如果不是根节点且节点的键值对数量满足最小要求，则不需要合并或重分配。如果需要合并或重分配，则获取节点的父节点和兄弟节点，根据键值对数量决定是进行重分配还是合并操作。
    
4. **叶子节点的删除** ：当删除叶子节点时，需要更新前一个叶子节点的`next_leaf`属性和后一个叶子节点的`prev_leaf`属性，以保持叶子节点链表的正确性。
    
5. **节点句柄的释放**：在删除节点后，需要更新文件头中的页面数量。
    
6. **维护子节点的父节点信息** ：在节点合并或键值对移动后，需要更新子节点的父节点信息，以保持树结构的正确性。

- `void redistribute(IxNodeHandle *neighbor_node, IxNodeHandle *node, IxNodeHandle *parent, int index);`

​		重新分配`node`和兄弟结点`neighbor_node`的键值对。参数`index`表示`node`在parent中的rid_idx，其决定`neighbor_node`是否为`node`的前驱结点。

​		首先，`redistribute`函数的目的是在`node`和其邻居节点`neighbor_node`间重新分配键值对，以及更新父节点partent中的相关信息。这个函数首先通过Index判断`neighbor_node`是node的前驱节点还是后继节点。如果neighbor_node前驱节点，那么它会从`neighbor_node`最后一个键值对开始移动到node的第一个位置；反之，如果`neighbor_node`是后继节点，它会从`neighbor_node`的第一个键值对开始移动到`node`的最后一个位置。移动键值对后，会调用`maintain_child`函数来更新孩子节点的父节点信息。

接下来，`erase_pair`函数用于从一个节点中删除指定位置的键值对。它首先通过memmove函数移动键数组(keys)和记录标识符数组（`rids`），以覆盖要删除的键值对。然后，它会更新节点中键值对的数量。这个函数是`redistribute`过程中用于从`neighbor_node`中删除已经移动到node的键值对的关键步骤。

最后，`maintain_child`函数用于更新节点的孩子节点信息。当一个节点不是叶子节点时，这个函数会加载其指定孩子节点，并将孩子节点的父节点设置为当前节点。这是在键值对移动过程中，确保树结构正确性的重要步骤。

- `bool adjust_root(IxNodeHandle *old_root_node);`

​		用于根结点被删除了一个键值对之后的处理。函数返回根结点是否需要被删除。

​		函数的主体分为两大部分，分别处理旧根节点是内部节点和叶节点的情况。

1. **内部节点的处理**：如果`old_root_node`是一个内部节点（非叶节点），函数会检查这个节点的大小（即它包含的键值对数量）。如果大小为1，说明这个内部节点只有一个孩子，那么这个孩子节点将被提升为新的根节点。为了实现这一点，函数首先获取这个孩子节点的页码（`new_root_page_no`），然后调用`update_root_page_no`函数更新根节点的页码。接着，通过`fetch_node`函数获取新根节点的句柄，并将其父页面编号设置为无效（`INVALID_PAGE_ID`），表示这是一个根节点。
    
2. **叶节点的处理**：如果`old_root_node`是一个叶节点，函数会检查这个节点的大小。如果大小为0，说明这个叶节点没有包含任何数据，此时会通过调用`release_node_handle`函数释放这个节点，并通过`update_root_page_no`函数将根页面编号设置为无效。
    

在处理完上述两种情况后，如果旧根节点既不是大小为1的内部节点，也不是大小为0的叶节点，函数不会进行任何操作。

#### 实验发现

![[Pasted image 20240706231713.png]]


### 任务4 B+树索引并发控制

对整个树加锁，即让查找、插入、删除三者操作互斥