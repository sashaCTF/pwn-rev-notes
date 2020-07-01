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

Assembly code is what represents machine code, as I mentioned earlier. The following is a hello world program in machine code:

```
\x31\xc0\x31\xdb\x31\xc9\x31\xd2\xeb\x11\xb0\x04\xb3\x01\xb2\x0b\x59\xcd\x80\x31\xc0\xb0\x01\x30\xdb\xcd\x80\xe8\xea\xff\xff\xff\x48\x65\x6c\x6c\x6f\x20\x57\x6f\x72\x6c\x64
```

The above is machine code represented in hex, except, machines don't interpret code as hex, they interpret it as binary. The following is in binary, and is how a computer would interpet it:

```
00110001 11000000 00110001 11011011 00110001 11001001 00110001 11010010 11101011 00010001 10110000 00000100 10110011 00000001 10110010 00001011 01011001 11001101 10000000 00110001 11000000 10110000 00000001 00110000 11011011 11001101 10000000 11101000 11101010 11111111 11111111 11111111 01001000 01100101 01101100 01101100 01101111 00100000 01010111 01101111 01110010 01101100 01100100
```

As horrible as assembly code can be, we're gonna stick to it, because machine code is just horrible to read, unless you're a computer

Don't worry if the above looks daunting, assembly code still isn't a very nice language to learn, and besides, you're here to learn it, and I'll try make it as clear as I can :)

By the end of these, I'm not expecting you to be able to write fluently in assembly or machine code, that isn't quite what I'm trying to teach, however, the aim of this is so that you can **understand** it when decompiling programs
