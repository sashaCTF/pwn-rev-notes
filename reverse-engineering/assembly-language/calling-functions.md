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
