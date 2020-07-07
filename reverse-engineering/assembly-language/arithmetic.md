# Arithmetic

In this we will go over arithmetic in assembly language, which will cover the instructions: `add`, `sub`, `mul`, `div`, `inc`, `dec`.

## add

`add` is the integer addition instruction. The syntax is as follows:

```asm
add destination, source
```

Here, source is added to destination, then is stored in destination. For example:
```
eax: 10
```
```asm
add eax, 5
```
Then `eax` will be `15`
This instruction can be used to add registers together, for example:
```
eax: 5
eax: 10
```
```asm
add ebx, eax
```
```
eax: 5
ebx: 15
```
You can also add to memory addresses, for example:
```
0x40000000 | 0x00000003
```
```asm
add BYTE PTR 0x40000000, 6
```
This uses a format specifier to point to the byte at memory address `0x40000000`. You can read more about format specifiers here.
```
0x40000000 | 0x00000009
```

## sub

`sub` is the integer subtraction instruction. Syntax is as follows:

```asm
sub destination, source
```
Here, source is subtracted from destination, and stored in destination. For example:
```
eax: 5
```
```asm
sub eax, 3
```
```
eax: 2
```
Same concept to `add`, except with subtraction.

## mul

## div

## inc and dec

`inc` and `dec` are very simple, `inc` is `increment`, and `dec` is `decrement`, which will add 1 or subtract 1 from the supplied operand. Syntax is as follows:

```asm
inc/dec register
```

For example:
```
eax: 2
```
```asm
inc eax
```
```
eax: 3
```

Same for `dec`
