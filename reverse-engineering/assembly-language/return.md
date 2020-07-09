# Returning

Returing is when the function reaches the end of the code it had to execute, so it returns a value, and goes back to where it was called, or, if it was main, it will terminate. In assembly, this is done with the `ret` instruction, and it is important to know how this works

## What does `ret` do?

`ret` is used to jump back to where it was called, and continue execution there instead, as it has finished executing code in that function. 

It does this by changing the `eip` register, which contains the address of the current instruction. It will change it to what is at the top of the stack frame, aka what `esp` points to. A `ret` instrcution, is basically a:
```asm
pop epi
```
As it takes what `esp` points to, sets `epi` to it, and increments `esp`
