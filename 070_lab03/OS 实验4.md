# 005-2022112479-曲本磊

## 4.1. 实验目的

- 建立对系统调用接口的深入认识
- 掌握系统调用的基本过程
- 能完成系统调用的全面控制
- 为后续实验做准备

## 4.2 实验过程

1. 添加新增系统调用的编号（挂载后也应修改）
修改/include/unistd.h，如图
![[Pasted image 20241223195332.png]]

2. 修改系统调用表
在sys.h中修改，table数组中添加两个中断调用函数，函数在 sys_call_table 数组中的位置必须和__NR_xxxxxx 的值对应上，根据（1）的序号，先添加whoami。同时在上面仿照其他函数加上声明。
![[Pasted image 20241223195802.png]]
由于系统调用总数变成了74，还需要修改kernel/system_call.s

![[Pasted image 20241223195914.png]]

3. 实现sys_whoami和sys_iam
在kernal/who.c中实现，代码如下：
```c
#include <errno.h>
#include <string.h>
#include <linux/head.h>
#include <linux/sched.h>
#include <linux/kernel.h>
#include <asm/system.h>
#include <asm/segment.h> /* get_fs_byte put_fs_byte */
#include <asm/io.h>

char msg[24]; /* 23+'\0' */

int sys_iam(const char* name){  
    int i,flag = 0;
    char tmp[24];
    for(i = 0; i < 24; i++){
        tmp[i] = get_fs_byte(name+i); /*逐字符取*/  
        if(tmp[i] == '\0'){
            flag = 1;
            break;
        }
    }

    if(flag == 0) /* 0-23中无'\0'，字符数超过23*/
        return -EINVAL;
    strcpy(msg,tmp); /*拷贝到内核中*/
    return i;
}  

int sys_whoami(char* name, unsigned int size){
    int i,num = 0;
    while(num < 24 && msg[num] != '\0') /* 计数 */
        num += 1;
    if(num > size){
        errno = EINVAL;
        return -1;
    }
    for(i = 0; i < size; i++){ /* 取字符 */
        put_fs_byte(msg[i], name + i);
        if(msg[i] == '\0')
           break;
    }
    return i;
}
```


4. 修改Makefile

修改makefile将添加的系统调用文件一起加入到编译、链接即可。
此处修改linux-0.11/kernel/Makefile两处进行修改：
![[Pasted image 20241223200041.png]]

![[Pasted image 20241223200104.png]]

make编译

5. 编写iam.c和whoami.c放入到Linux0.11环境中，编译测试
whoami.c:
```c
#define __LIBRARY__                    /* 有它，_syscall1等才有效。详见unistd.h */
#include <unistd.h>                /* 有它，编译器才能获知自定义的系统调用的编号 */
#include <stdio.h> 
#include <string.h> 
_syscall2(int, whoami,char*,name,unsigned int,size);    /* whoami()在用户空间的接口函数 */

int main()
{
	char meg[24];  
	int length=0;
	length = whoami(meg,24);
	printf("%s\n",meg);
	/* printf("%d\n",length); */
	return 0;
}

```

iam.c:
```c
#define __LIBRARY__                    /* 有它，_syscall1等才有效。详见unistd.h */
#include <unistd.h>                /* 有它，编译器才能获知自定义的系统调用的编号 */
#include <errno.h> 
#include <string.h> 
#include <stdio.h> 

_syscall1(int, iam, const char*, name);        /* iam()在用户空间的接口函数 */

int main(int argc,char ** argv)
{
	iam(argv[1]);
	return 0;
}
```

编译指令
`gcc -o iam iam.c -Wall`

## 4.3实验结果

1. 手动结果
![[Pasted image 20241223200909.png]]

2. 编译testlab2.c结果
![[Pasted image 20241223200928.png]]

3. 运行testlab2.sh结果
![[Pasted image 20241223200947.png]]

## 4.4 回答问题

（1）问题1
从 Linux 0.11 现在的机制看，它的系统调用最多能传递几个参数？
      最多可以传递5个，通过int 0x80：eax,ebx,ecx,edx,edi,esi

（2）问题2
你能想出办法来扩大这个限制吗？
将额外的参数放在栈中，从栈中取出参数

（3）问题3
用文字简要描述向 Linux 0.11 添加一个系统调用 foo() 的步骤。
1. 修改include/unistd.h，添加新增系统调用的编号
2. 修改include/linux/sys.h中的sys_call_table，添加foo
3. 实现foo函数
4. 修改MakeFile，加入foo函数的实现
5. 编译