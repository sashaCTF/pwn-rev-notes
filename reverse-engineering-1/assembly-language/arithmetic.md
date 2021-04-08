# Arithmetic

In this we will go over arithmetic in assembly language, which will cover the instructions: `add`, `sub`, `mul`, `div`, `inc`, `dec`.

## add

`add` is the integer addition instruction.

The syntax is as follows:

```text
add dest, src
```

Here, `src` added to `dest`, then is stored in `dest`. For example:

```text
eax = 10
```

```text
add eax, 5
```

Then `eax` will be `15` This instruction can be used to add registers together, for example:

```text
eax = 5
ebx = 10
```

```text
add ebx, eax
```

```text
eax: 5
ebx: 15
```

You can also add to memory addresses, for example:

```text
0x4000 | 0x00000003
```

```text
add BYTE PTR [0x4000], 6
```

This uses a format specifier to point to the byte at memory address `0x40000000`. You can read more about format specifiers [here](format-specifiers.md).

```text
0x4000 | 0x00000009
```

## sub

`sub` is the integer subtraction instruction.

Syntax is as follows:

```text
sub dest, src
```

Here, `src` is subtracted from `dest`, and stored in destination. For example:

```text
eax = 5
```

```text
sub eax, 3
```

```text
eax = 2
```

Same concept as `add`, except with subtraction.

## mul

`mul` is used to multiply two numbers. The `eax` register is _always_ multiplied against the supplied operand, and the result is stored in two registers, `eax` and `edx`, where the _most significant_ bits are stored in `edx`, and _least significant_ bits are stored in `eax`.

Syntax is as follows:

```text
mul multiplier
```

For example:

```text
eax: 5
edx: 0
```

```text
mul 4
```

```text
eax: 5
edx: 20
```

## div

`div` is used to divide numbers. Similar concept to `mul`, where the `eax` register is always divided by the provided operand. The quotient is then stored in the `eax` register, while the remainder is stored in the `edx` register.

Syntax is as follows:

```text
div divisor
```

For example:

```text
eax: 10
edx: 0
```

```text
div 3
```

```text
eax: 3    <== quotient
edx: 1    <== remainder
```

## inc and dec

`inc` and `dec` are very simple, `inc` is `increment`, and `dec` is `decrement`, which will add 1 or subtract 1 from the supplied operand.

Syntax is as follows:

```text
inc/dec destination
```

For example:

```text
eax: 2
```

```text
inc eax
```

```text
eax: 3
```

Same for `dec`

