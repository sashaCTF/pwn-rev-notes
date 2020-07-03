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
add 0x40000000, 6
```
```
0x40000000 | 0x00000009
```
