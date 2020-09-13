# Dealing with data

In this we will go over how assembly handles data, through the use of the instructions: `mov`, `lea`, `push`, `pop`, `xchg`

## mov and lea

`mov` and `lea` are both quite similiar, however it's important to know the differences

### mov

`mov` is used to copy data to an address or register. The syntax is as follows:

```text
mov destination, source
```

For example, if I wanted to copy the value 5 to the eax register:

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
 0x40000000 | 0x11111111
 0x40000004 | 0x22222222
 0x40000008 | 0x33333333
 ...
```

And we wanted to move the value at `0x40000000` to `eax`, we would do:

```text
 mov eax, 0x40000000
```

After the `mov`, our `eax` register would contain the value `0x11111111`

### lea

`lea` is similiar to `mov`, except it loads addresses. Syntax is as follows:

```text
 lea destination, source
```

Let's take our previous memory example:

```text
 ...
 0x40000000 | 0x11111111
 0x40000004 | 0x22222222
 0x40000008 | 0x33333333
 ...
```

And do the same instruction, except with a `lea`:

```text
 lea eax, 0x40000000
```

Instead of `eax` being `0x11111111`, it'll be `0x40000000`. This is because `lea` doesn't copy what's at the address over, rather it copies the address itself

Summed up, `mov` copies the _value_ at the address, while `lea` copies the _address_ itself

## push and pop

These two instructions are usually used together, but do very oppsoite jobs. \(Note: It's important to remember here that `esp` points to the top of the stack frame\)

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

If you're using `push` to write the value of a register to the top, it maintains the value in the register, as it is copying. Another important note, when a `push` is executed, it first _decrements_ `esp` by 4/8 bytes \(32bit/64bit\), as the stack grows downwards, then the value is copied to the stack frame, where `esp` points to.

So, push basically acts as:

```text
 sub esp, 4/8
 mov DWORD/QWORD PTR esp, register
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
 push 0x33333333
```

Firstly, `esp` decrements by 4 bytes, so it points to the next available place \(`0xffff0004`\):

```text
 ...
 0xffff000c | 0x11111111
 0xffff0008 | 0x22222222
 0xffff0004 | 0x00000000    <=== esp points here
 0xffff0000 | 0x00000000
 ...
```

Then we copy the value to where `esp` now points to:

```text
 ...
 0xffff000c | 0x11111111
 0xffff0008 | 0x22222222
 0xffff0004 | 0x33333333    <=== esp points here
 0xffff0000 | 0x00000000
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

`pop` doesn't remove the values from the stack frame, however it does copy them. Another important note, similiar to push, is that when a `pop` instruction is executed, it will read the value that `esp` points to, then will increment `esp` by 4/8 \(32bit/64bit\).

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

This combo is used a lot in calling higher level c functions, with arguments. This is covered more in another article

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

For example, if `eax` contained `3`, and `ebx` contained `5`, you could use `xchg` to swap \(or _exchange_\) them

```text
 xchg eax, ebx
```

Then the registers would look like this:

```text
 eax: 5
 ebx: 3
```

