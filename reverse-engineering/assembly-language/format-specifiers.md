# Format Specifiers

Format specifiers are used to specify how much data will be handled in assembly instructions, such as `mov` or `lea`. The specifiers are shown in the table below:

| Type Specifier | No. of bytes |
| :--- | :--- |
| BYTE | 1 |
| WORD | 2 |
| DWORD | 4 |
| QWORD | 8 |
| TBYTE | 10 |

For example:

```text
mov ebx, DWORD eax
```

Would move the 4 bytes at eax into `ebx`. So if `eax` was `0x11223344`, `ebx` would hold `0x11223344`

`PTR` is also used quite a lot, for example:

```text
BYTE PTR
DWORD PTR
QWORD PTR
```

`PTR` means pointer, and represents what an address in a register points to. So:

If there was a memory address `0x11111111`:

```text
0x11111111 | 0x22222222
```

Which held `0x22222222`, and if `eax` held that address:

```text
eax: 0x11111111
```

Then the contents of `eax` is `0x11111111`. So, if we did:

```text
DWORD eax
```

The value this represents is `0x11111111`. However, if we did:

```text
DWORD PTR eax
```

This value would be `0x22222222`, as the `PTR` means it takes the value at the address that `eax` stores

Summed up, `DWORD` would take the _value_, while `DWORD PTR` takes the value _at the address_

