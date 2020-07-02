# Dealing with data

In this we wil go over how assembly handles data, through the use of instructions such as `mov`, `lea`, `push`, `pop`, `xchg`

## mov and lea

`mov` and `lea` are both quite simliar, however it's important to know the differences

### mov
`mov` is used to copy data to an address or register. The syntax is as follows:

```asm
mov destination, source
```

 For example, if I wanted to copy the value 5 to the eax register:
 ```asm
 mov eax, 5
 ```
 
 Or if I wanted to copy the contents of `eax` to `ebx`
 ```asm
 mov ebx, eax
 ```
 And here's an example with addresses:
 Lets say that our memory looks like this:
 ```
 ...
 0x40000000 | 0x11111111
 0x40000004 | 0x22222222
 0x40000008 | 0x33333333
 ...
 ```
 And we wanted to move the value at `0x40000000` to `eax`, we would do:
 ```asm
 mov eax, 0x40000000
 ```
 After the `mov`, our `eax` register would contain the value `0x11111111`
 
 ### lea
 
 `lea` is similiar to `mov`, except it loads addresses. Syntax is as follows:
 ```asm
 lea destination, source
 ```
 Let's take our previous memory example:
 ```
 ...
 0x40000000 | 0x11111111
 0x40000004 | 0x22222222
 0x40000008 | 0x33333333
 ...
 ```
 And do the same instruction, except with a `lea`:
 ```asm
 lea eax, 0x40000000
 ```
 Instead of `eax` being `0x11111111`, it'll be `0x40000000`. This is because `lea` doesn't copy what's at the address over, rather it copies the address itself
 
 
 Summed up, `mov` copies *values*, while `lea` copies *addresses*
 
 ## push and pop
 
 These two instructions are usually used together, but do very oppsoite jobs. 
 (Note: It's important to remember here that `esp` points to the top of the stack frame)
 
 ### push
 `push` is used to copy values to the top of the stack frame. Syntax is as follows:
 
 ```asm
 push value
 ```
 
 For example, if you wanted to copy 5 to the top of the stack:
 ```asm
 push 5
 ```
 
 Or the contents of the register `eax`:
 ```asm
 push eax
 ```
 
 If you're using `push` to write the value of a register to the top, it maintains the value in the register, as it copies. Another important note, when a `push` is executed, it first *decrements* `esp` by 4/8 bytes (32bit/64bit), as the stack grows downwards, then the value is copied to the stack frame, where `esp` points to. For example, take our memory diagram below:
 ```
 ...
 0xffff000c | 0x11111111
 0xffff0008 | 0x22222222    <=== esp points here
 0xffff0004 | 0x00000000
 0xffff0000 | 0x00000000
 ...
 ```
 
 Then we execute:
 ```asm
 push 0x33333333
 ```
 Firstly, `esp` decrements by 4 bytes, so it points to the next available place (`0xffff0004`):
 ```
 ...
 0xffff000c | 0x11111111
 0xffff0008 | 0x22222222
 0xffff0004 | 0x00000000    <=== esp points here
 0xffff0000 | 0x00000000
 ...
 ```
 
 Then we copy the value to where `esp` now points to:
 ```
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
 
 ```asm
 pop register
 ```
 
 For example, if you wanted to read the value at the top of the stack frame, and put it into the `eax` register:
 ```asm
 pop eax
 ```
 
 
 This is commonly used alongside push. For example, the following code would write 3 to the top of the stack, and then remove it and put it into the `eax` register:
 ```asm
 push 3
 pop eax
 ```
 
 This combo is used a lot in calling higher level c functions, with arguments. This is covered more in another article
 
 ## xchg
 
 `xchg` will basically swap the contents of twp register around. Syntax is as follows:
 
 ```asm
 xchg register1, register2
 ```
 For example, if `eax` contained `3`, and `ebx` contained `5`, you could use `xchg` to swap (or *exchange*) them
 
 ```asm
 xchg eax, ebx
 ```
 Then the registers would look like this:
 ```
 eax: 5
 ebx: 3
 ```
 
