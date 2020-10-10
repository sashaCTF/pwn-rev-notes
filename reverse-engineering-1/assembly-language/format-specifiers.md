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

Sometimes you'll also see `DWORD PTR` instead of just `DWORD`, but these mean the same thing, so don't get confused by it. The `PTR` means pointer, so in this case it would move the 4 bytes \(`DWORD`\) that is at the address in `eax`, into ebx.

Say that `eax` contains an address `0x1000`:

```text
...
0x1000 : 0x11111111
0x2000 : 0x22222222
...
```

And we executed the previous instruction, `ebx` would then contain `0x11111111`

