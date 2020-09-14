# Registers

Registers are an area of memory that is quickly accessible by the processor compared to everyhwere else. They are often used to store small amounts of data for short periods of time, such as small numbers, addresses, or any other 4/8 byte value.

## General Purpose Registers

Listed below are the general puprose registers, which are used for most regular tasks:

### 64 bit

* rax
* rbx
* rcx
* rdx
* rsi
* rdi
* r8
* r9
* r10
* r11
* r12
* r13
* r14
* r15

### 32 bit

* eax
* ebx
* ecx
* edx
* esi
* edi

### Their purposes

\(note: i will refer to registers by their 64 bit counterpart, even though they're basically the same\)

While these are general purpose registers, some usually have a purpose that they're used for

* `rax`: this is typically used as a return value register. For example, if a function returns the value `10`, it will be stored in `rax`
* `rbx`: preserved register
* `rcx`: scratch register, can be used as a counter sometimes
* `rdx`: scratch register
* `rsi`: argument register, holds argument \#2
* `rdi`: argument register, holds argument \#1
* `r8-11`: scratch registers
* `r12-r15`: preserved registers

## Special Purpose Registers

Special purpose registers are reserved registers that each perform an important and unqiue task. The 3 special purpose regsisters are as follows \(64 bit\):

* rbp
* rsp
* rip

### rbp

`rbp` is the base pointer, which points to the _bottom_ of the current stack frame. `rbp` is also where variables are calculated from

### rsp

`rsp` is the stack pointer, which points to the _top_ of the current stack frame. This is where `ret` gets it's address to return to, and is an important register as it controls execution

### rip

`rip` is the program counter, and holds the address of the current instruction. This is often controlled to also change execution, such as with `jmp`, `ret` and `call` instructions

## Register Segments

Most of the registers are broken up into segments, such as the general purpose registers. For example, the `rax` register is the full, 64-bit version of that register, while `eax` is the 32-bit version. Then there's a 16-bit version `ax`, which is then broken up into two 8-bit registers `ah` and `al`. These 8-bit registers are labelled as such, because the `h` means `higher` and `l` means `lower`, basically saying that `al` is the lower of the two bytes in `ax`. This might sound confusing, so I'll give a diagram below:

![rax](../../../.gitbook/assets/rax.png)

This structure is also applied to other general purpose registers such as `rbx`, `rcx` and `rdx`:

![registers](../../../.gitbook/assets/registers.png)

Other registers can be segmented as well, but these are the main ones you'll see.

Special purpose registers are _not_ segmented in this manner either, the only segments you'll see are the 64-bit and 32-bit versions

