# conditional vs unconditional jumps

This section covers the use of the `jmp` instruction, and it's variants

## unconditional jumps

We'll start with unconditional jumps first, as they are the easiest. They basically redirect execution by jumping to a different place in the code. This is done with the `jmp` instruction, whose syntax is as follows:
```asm
jmp destination
```

Destination can hold either a static address, or a register

Here's an example of it's usage

```asm
0x100000:   jmp 0x200000
...
0x200000:   [code]
```
Where it just skips over the code between `0x100000` and `0x200000`

They can also be used to make a `while true` loop:

```asm
0x100000:   [code]
...
0x200000:   jmp 0x100000
