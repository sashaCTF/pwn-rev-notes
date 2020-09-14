# Returns

Returing is when the function reaches the end of the code it had to execute, so it returns a value, and goes back to where it was called, or, if it was main, it will terminate. In assembly, this is done with the `ret` instruction, and it is important to know how this works

We'll also cover the `leave` instruction

## What does `ret` do?

`ret` is used to jump back to where it was called, and continue execution there instead, as it has finished executing code in that function.

It does this by changing the `eip` register, which contains the address of the current instruction. It will change it to what is at the top of the stack frame, aka what `esp` points to. A `ret` instrcution, is basically:

```text
pop eip
```

As it takes what `esp` points to, sets `eip` to it, and increments `esp`

Another note is that when a function is called, the `call` instruction writes the address of the next insruction to the top of the stack frame, so `ret` can use it to call back to where it was called from

## What does `leave` do

You may sometimes see a `leave` instruction called before a `ret` as well. What `leave` does, is copy `ebp` to `esp`, then pop a value from `esp` to `ebp`. `leave` is basically:

```text
mov esp, ebp
pop ebp
```

This is often used to get `esp` pointing to the required return address

