# conditional vs unconditional jumps

This section covers the use of the `jmp` instruction, plus it's variants, and the `cmp` instruction

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
```

## conditional jumps

Conditional jumps are essentially how assembly implements `if` statements. There are two parts that make up a conditional jump:
```
1: Some form of logic operation or comparison
2: The jump
```

The first part is often filled out with a `cmp` instruction, however it can also be a [logic operation](logic.md)

### cmp

The `cmp` instruction essentially takes two operands, and subtracts one from the other, but *doesn't* change the values of them. Syntax is as follows:
```asm
cmp destination, source
```

For example, if we wanted to compare the value of `eax` to `10`, we would do:
```asm
cmp eax, 10
```

### jumps

We'll come back to that in a moment, as we also need to discuss jumps to be able to use that `cmp`

A conditional jump instruction has the basic structure of:
```
j[condition]
```
For example:
```
je  = jump equal
jne = jump not equal
```
I won't cover them all here, if you want to see more, [here](https://www.tutorialspoint.com/assembly_programming/assembly_conditions.htm) is a good site

Using `jne`:
```asm
cmp eax, 10
jne [somewhere else]
```
This will jump if `eax` isn't equal to `10`

Here's also a common example for `test`:
```asm
test eax, eax
je [somehwere else]
```
This essentially jumps if `eax` is equal to `0`
