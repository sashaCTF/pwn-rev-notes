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
mov ebx, DWORD PTR [eax]
```

Often you'll also see `DWORD PTR` instead of just `DWORD`

