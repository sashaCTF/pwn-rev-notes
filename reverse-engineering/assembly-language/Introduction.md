# What is assembly language?

Assembly language is the lowest level of programming language you can get. A low level programming is one whose instruction set closer resembles the language that computers understand, which can make it harder for humans to understand. This is the opposite of a high level programmng language, such as python, which is easier for humans to read, but machines can't read it (so we translate to machine code).

Assembly code and machine code are the two lowest programming languages, and are essentially the same, except that machine code is the raw bytes, while assembly is the visual representation of those instructions

Because of this, more instructions are required to do the task compared to a higher level language.

Here we see the same program written in 3 different programming languages, which is a classic "Hello World!" program, to demonstrate this point.

Python (high)
```python
print("Hello World!")
```

C (medium)
```c
#include <stdio.h>
int main() {
   puts("Hello, World!");
   return 0;
}
```

Assembly (low)
```asm
section     .text
global      _start                              ;must be declared for linker (ld)

_start:                                         ;tell linker entry point

    mov     edx,len                             ;message length
    mov     ecx,msg                             ;message to write
    mov     ebx,1                               ;file descriptor (stdout)
    mov     eax,4                               ;system call number (sys_write)
    int     0x80                                ;call kernel

    mov     eax,1                               ;system call number (sys_exit)
    int     0x80                                ;call kernel

section     .data

msg     db  'Hello, world!',0xa                 ;our dear string
len     equ $ - msg                             ;length of our dear string
```
