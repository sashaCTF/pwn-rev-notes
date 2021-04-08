# Dealing with data

In this we will go over how assembly handles data, through the use of the instructions: `mov`, `lea`, `push`, `pop`, `xchg`

## mov and lea

`mov` and `lea` are both quite similar, however it's important to know the differences.

### mov

`mov` is used to copy data to an address or register. The syntax is as follows:

```text
mov dest, src
```

For example, if I wanted to copy the value 5 to the `eax` register:

```text
 mov eax, 5
```

Or if I wanted to copy the contents of `eax` to `ebx`

```text
 mov ebx, eax
```

And here's an example with addresses: Lets say that our memory looks like this:

```text
 ...
 0x4000 | 0x1111
 0x4004 | 0x2222
 0x4008 | 0x3333
 ...
```

And we wanted to move the value at `0x4000` to `eax`, we would do:

```text
 mov eax, DWORD PTR [0x4000]
```

After the `mov`, our `eax` register would contain the value `0x1111`

### lea

`lea` is similar to `mov`, except it can be much more powerful. Syntax is as follows:

```text
 lea dest, [src]
```

`lea` is a useful instruction because it can help to reduce the lines of assembly by doing arithmetic in the `src` parameter before copying the value to `dest` 

For example:

```text
add eax, 5
mov ebx, eax
```

Can be translated to:

```text
lea ebx, [eax+5]
```

This can be extended to:

```text
lea ecx, [ebx*5 + ebx - 3]
```

Due to it's versatility, it can be a very useful instruction

### push and pop

These two instructions are usually used together, but do very opposite jobs. \(Note: It's important to remember here that `esp` points to the top of the stack frame\).

### push

`push` is used to copy values to the top of the stack frame. Syntax is as follows:

```text
 push value
```

For example, if you wanted to copy 5 to the top of the stack:

```text
 push 5
```

Or the contents of the register `eax`:

```text
 push eax
```

If you're using `push` to write the value of a register to the top, it keeps the value in the register, as it is only copying. Another important note, when a `push` is executed, it first _decrements_ `esp` by 4/8 bytes \(32bit/64bit\), as the stack grows downwards, then the value is copied to the stack frame, where `esp` points to.

So, push basically acts as:

```text
 sub esp, 4/8
 mov DWORD/QWORD PTR [esp], value
```

For example, take our memory diagram below:

```text
 ...
 0xffff000c | 0x11111111
 0xffff0008 | 0x22222222    <=== esp points here
 0xffff0004 | 0x00000000
 0xffff0000 | 0x00000000
 ...
```

Then we execute:

```text
 push 0x3333
```

Firstly, `esp` decrements by 4 bytes, so it points to the next available place \(`0xffff0004`\):

```text
 ...
 0xffff000c | 0x1111
 0xffff0008 | 0x2222
 0xffff0004 | 0x0000    <=== esp points here
 0xffff0000 | 0x0000
 ...
```

Then we copy the value to where `esp` now points to:

```text
 ...
 0xffff000c | 0x1111
 0xffff0008 | 0x2222
 0xffff0004 | 0x3333    <=== esp points here
 0xffff0000 | 0x0000
 ...
```

It's going down as that's the direction the stack grows

### pop

`pop` is used to read values from the top of the stack frame, and put them into a register. Syntax is as follows:

```text
 pop register
```

For example, if you wanted to read the value at the top of the stack frame, and put it into the `eax` register:

```text
 pop eax
```

`pop` doesn't remove the values from the stack frame, however it does copy them. Another important note, similar to push, is that when a `pop` instruction is executed, it will read the value that `esp` points to, then will increment `esp` by 4/8 \(32bit/64bit\).

So, `pop` basically acts as:

```text
mov register, esp
add esp, 4/8
```

Take our memory diagram from earlier:

```text
 ...
 0xffff000c | 0x11111111
 0xffff0008 | 0x22222222    <=== esp points here
 0xffff0004 | 0x00000000
 0xffff0000 | 0x00000000
 ...
```

When we execute:

```text
 pop eax
```

First, it reads at `0xffff0008`, and places the value into `eax`, so `eax` will now be: `0x22222222`.

Then, `esp` is incremented by 4 bytes:

```text
 ...
 0xffff000c | 0x11111111    <=== esp points here
 0xffff0008 | 0x22222222
 0xffff0004 | 0x00000000
 0xffff0000 | 0x00000000
 ...
```

This is commonly used alongside push. For example, the following code would write 3 to the top of the stack, and then remove it and put it into the `eax` register:

```text
 push 3
 pop eax
```

This combo is used a lot in calling higher level c functions, with arguments. This is covered more in another article.

#### popal

`popal` is an instruction that is exclusively on 32-bit architectures.  It is used to pop every general purpose register, however you won't see it used very often. It doesn't take any operands, so syntax is just:

```text
popal
```

It will pop the registers in this order:

```text
EDI
ESI
EBP
EBX
EDX
ECX
EAX
```

And has an opcode of `\x61`

## xchg

`xchg` will basically swap the contents of twp register around. Syntax is as follows:

```text
 xchg register1, register2
```

For example, if `eax` contained `3`, and `ebx` contained `5`, you could use `xchg` to swap \(or _exchange_\) them.

```text
 xchg eax, ebx
```

Then the registers would look like this:

```text
 eax: 5
 ebx: 3
```

