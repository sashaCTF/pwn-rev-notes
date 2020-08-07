# Calling functions

Assembly can call higher level functions, especially when compiled from C into machine code. This is done with the `call` instruction. Syntax is as follows:

```asm
call function
```

This can be used to call functions you made, for example, say we had this function:
```c
int myFunc() {
    puts("Hello World!");
    return 0;
}
```

And in assembly, we wanted to call this, we would do:

```asm
call myFunc
```

What this will do, is jump into `myFunc`, execute the machine code that makes up that function, then jump back to the `call` instruction with a `ret` instruction, and move to the next instruction. 

In basic terms, `call` is essentially a `jmp` instruction

## Calling functions with arguments

Often, functions are called with arguments passed in, such as C functions like `puts`and `gets`, or custom functions you made. How does assembly handle that?

### 32-bit vs 64-bit

Calling functions with arguments is different in 32-bit and 64-bit, and I'll go over how both work, as it is important to know the difference, especially when writing ROP exploits:

#### 32-bit

In 32-bit, arguments for functions are taken from the top of the stack frame. Often what happens is that the agrumnets are pushed onto the stack with the `push` instrcution, right before calling (more info on `push` can be found [here](dealing-with-data.md#push))

For example, say that we wanted to print the string `Hello World` with the C function `puts`

Here, we store the address of the string `Hello World` in `eax`:

```
eax: 0x11111111   ===> 'Hello World'
```

And, right before we call `puts`, we push `eax` onto the stack like so:

```asm
push eax
```

So, all together, it would look like this:

```asm
push eax
call puts
```

And we'd print `Hello World`

But what about *multiple* arguments?

Well, we can call multiple `push` instructions before the `call`, however we have to do it in reverse order. So if we wanted to call a funcion, say `myFunc` for example, and wanted to pass the args `1 , 2 , 3`, in that order, instead of doing:
```asm
push 1
push 2
push 3
call myFunc
```

We would do:
```asm
push 3
push 2
push 1
call myFunc
```

This is because `push` decrements `esp`, aka the top of the stack frame, before it writes to the stack. So, if we did what we did in our first example, this is what would happen:

Let's say our `esp` starts at `0xffff0010`. This is what our memory looks like right now:
```
...
0xffff0010 | 0xdeadbeef     <=== esp points here
0xffff000c | 0x0
0xffff0008 | 0x0
0xffff0004 | 0x0
...
```
When we call our first `push` (`push 1`), it would write to the next available space, which is 4 bytes below `esp`. So, when we call that, memory now looks like this:
```
...
0xffff0010 | 0xdeadbeef
0xffff000c | 0x1            <=== esp points here
0xffff0008 | 0x0
0xffff0004 | 0x0
...
```
And after executing the other two, it now looks like this:
```
...
0xffff0010 | 0xdeadbeef
0xffff000c | 0x1
0xffff0008 | 0x2
0xffff0004 | 0x3            <=== esp points here
...
```
Now, when the function executes, it will take the args going up in memory, so it would first take `0x3`, then `0x2`, then `0x1`. So, if we reverse the order, memory looks like this:
```
...
0xffff0010 | 0xdeadbeef
0xffff000c | 0x3
0xffff0008 | 0x2
0xffff0004 | 0x1            <=== esp points here
...
```
And would start at `0x1`, then `0x2`, then `0x3`

#### 64-bit

In 64-bit, instead of reading arguments from the top of stack frame, it reads them from registers. The register used, in order, are:
```
rdi
rsi
rdx
rcx
r8
r9
```
So for example, if you wanted to call our previous function with 1 arg, you would do:

```asm
mov rdi, 1
call myFunc
```
Or if you wanted to call with 3:
```asm
mov rdi, 1
mov rsi, 2
mov rdx, 3
call myFunc
```
However, if there are more than 6 arguments, the next ones are put onto the stack, and used in a similiar way to 32 bit, however you won't have to worry about this very often as it's usually rare to encounter functions that have 6 args

