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

In 32-bit, arguments for functions are taken from the top of the stack frame. Often what happens is that the agrumnets are pushed onto the stack with the `push` instrcution, right before calling (more info on `push` can be found [here](https://github.com/sashaCTF/pwn-rev-notes/blob/master/reverse-engineering/assembly-language/dealing-with-data.md#push))
