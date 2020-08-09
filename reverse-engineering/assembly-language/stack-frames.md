# The Stack Frame

The stack is an area of memory where data is typically stored (unless it's larger data, as that's usually stored in the heap). It is located near the top of the stack, just below the kernel, and grows downwards. The stack is made up of *stack frames*, which are areas of the stack that different fucntions use. When a function is called, it is assigned it's own stack frame, and the size of it depends on the size of what it stores. For example:

```c
int func() {
        char buffer[64];
        char variable[8];
}
```

When this function is called, it is assigned a stack frame, where it will have 72 bytes of memory allocated, as that's how much it requests for the variables it needs. However, this isn't all that a stack frame will store. When a called function has finished its execution, it needs to know where to return back to, but to also know where the previous stack frame was. This is why, a stack frame also stores a `base pointer` and a `return address`. These are placed after all the assigned variables. Let's talk about these in a bit more detail:

### base pointer

Right after the variables (and canary if there is one), is the *saved* base pointer. This is essentially the `rbp` value in the previous stack frame
