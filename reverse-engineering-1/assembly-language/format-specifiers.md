# Format Specifiers

Format specifiers are used to specify how much data will be handled in assembly instructions, such as `mov` or `lea`. The specifiers are shown in the table below:

| Type Specifier | No. of bytes |
| :--- | :--- |
| BYTE | 1 |
| WORD | 2 |
| DWORD | 4 |
| QWORD | 8 |
| TBYTE | 10 |

An example of it's usage is:

```text
mov ebx, DWORD [eax]
```

Sometimes you'll also see `DWORD PTR` instead of just `DWORD`, but these mean the same thing, so don't get confused by it. The `PTR` means pointer, so in this case it would move the 4 bytes \(`DWORD`\) that is at the address in `eax`, into `ebx`.

Say that `eax` contains an address `0x1000`:

```text
...
0x1000 : 0x11112222
0x2000 : 0x33334444
...
```

And we executed the previous instruction, `ebx` would then contain `0x11112222`

However, if we executed the following instruction:

```text
mov bx, WORD [0x1000]
```

The result in `bx` would be `0x2222` , as it would only move 2 bytes. We have to use `bx` because the sizes of the operands have to match, and `bx` is the lower 16-bits of `ebx`. For example, the following instruction would be invalid.

```text
mov ebx, BYTE [0x1000]
```

