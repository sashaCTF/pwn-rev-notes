# Registers

Registers are an area of memory that is quickly accessible by the processor compared to everyhwere else. They are often used to store small amounts of data for short periods of time, such as small numbers, addresses, or any other 4/8 byte value.

## An overview

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

## Their purposes

(note: i will refer to registers by their 64 bit counterpart, even though they're basically the same)

While these are general purpose registers, some usually have a purpose that they're used for

* `rax`: this is typically used as a return value register. For example, if a function returns the value `10`, it will be stored in `rax`
* `rbx`: preserved register
* `rcx`: scratch register, can be used as a counter sometimes
* `rdx`: scratch register
* `rsi`: argument register, holds argument #2
* `rdi`: argument register, holds argument #1
* `r8-11`: scratch registers
* `r12-r15`: preserved registers
