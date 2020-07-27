# System Calls

A system call is where the code makes a call to the kernel to execute certain 'functions'. This is used with multiple instructions such as:
```
syscall
int [0x80/80h]
sysenter
```

The function that's executed from a system call depends on what number is provided, aka the system call number.

This system call number is stored in the `eax` register, and the arguments are stored in registers. The registers are different in 32-bit vs 64-bit, so here's both:

#### 32-bit
```
Call number:
eax

Arguments:
ebx
ecx
edx
esi
edi
ebp

Return value:
eax
```

#### 64-bit
```
Call number:
rax

Arguments:
rdi
rsi
rdx
rcx
r10
r8
r9

Return value:
rax
```

## int 80 vs syscall

These are the two you're likely to come across are `int 80` and `syscall`, and I'll go over both here.

#### int 80

`int 80` tends to be used on **32-bit**, however it can be used on 64-bit (*not* recommended though). `sysenter` is also more advisable to use on 32-bit rather than `int 80`

#### syscall

`syscall` is used on **64-bit**

## system call numbers

These are stored in `eax`/`rax` at the time of the system call. For example, if you wanted to make an `execve` system call on 64-bit, your registers would look like this:
```
rax: 59 (or 0x3b)
rdi: [command to execute]
rsi: 0
rdx: 0
```

You can also find tables online with this info, and I've linked some tables below:

https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md
