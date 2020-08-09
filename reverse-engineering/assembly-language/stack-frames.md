# The Stack Frame

The stack is an area of memory where data is typically stored (unless it's larger data, as that's usually stored in the heap). It is located near the top of the stack, just below the kernel, and grows downwards. The stack is made up of *stack frames*, which are areas of the stack that different fucntions use. When a function is called, it is assigned it's own stack frame, and the size of it depends on the size of what it stores. For example:

```c
int func() {
        char buffer[64];
        char variable[8];
}
```

When this function is called, it is assigned a stack frame, where it will have 72 bytes of memory allocated, as that's how much it requests for the variables it needs. However, this isn't all that a stack frame will store. When a called function has finished its execution, it needs to know where to return back to, but to also know where the previous stack frame was. This is why, a stack frame also stores a `base pointer` and a `return address`. These are placed after all the assigned variables. Let's talk about these in a bit more detail:

### Base Pointer

Right after the variables (and canary if there is one), is the *saved* base pointer. This is essentially the `rbp` value in the *previous* stack frame, and it's saved to the new stack frame so that when you return back, the program resets `rbp` back to its previous value. This is also why, at the start of functions, you'll usually see:
```asm
push rbp
```

### Return Pointer

The return pointer comes after the saved `rbp`, and dictates where the program should return back to once the called function has ended. This is saved to the stack frame by the `call` instruction. I mentioned this [here](calling-functions.md) as well, but I'll go over it again. When a function is called, it jumps to the code, but also pushes the address of the next instruction to the stack. However, to make sure it doesn't write this to the current stack frame, `rsp` will be altered before it happens. So, `call` is basically:
```asm
jmp func
push [rip + 5]
```
(it's 5 because that's usually how many bytes a `call` instruction is)
