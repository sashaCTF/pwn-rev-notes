# Logical Operations

Assembly uses logic gates as instructions, and we'll go over how it uses them with the instructions: `and`, `or`, `xor`, `test`, `not`.

The syntax for the next 4 instructions is just:

```text
[operation] dest, src
```

## and

`and` is a logic gate where both operands have to be `true` \(or `1`\) to return `true` \(or `1`\). It will do the operation, and store the result in the first operand.

| A | B | Result |
| :--- | :--- | :--- |
| 0 | 0 | 0 |
| 1 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 1 | 1 |

For example, if `eax` contained `17`, and you did the `and` operation with `3`:

```text
and eax, 3
```

```text
00010001  =  17
00000011  =  3
---------------
00000001  =  1
```

`eax` would then hold `1`

## or

`or` is a logic gate where only one operand needs to be `true` \(or `1`\). Works in a similar way to `and`

| A | B | Result |
| :--- | :--- | :--- |
| 0 | 0 | 0 |
| 1 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 1 | 1 |

For example if `eax` stored `6`, and was or'ed against `1`:

```text
or eax, 1
```

```text
00000110  =  6
00000001  =  1
--------------
00000111  =  7
```

`eax` would then hold `7`.

## xor

`xor` is a logic gate where exactly one operand should be `true` \(or `1`\) to result in `true`

| A | B | Result |
| :--- | :--- | :--- |
| 0 | 0 | 0 |
| 1 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 1 | 0 |

For example, if `eax` held `7` and you xor'ed it against `18`:

```text
xor eax, 18
```

```text
00000111  =  7
00010010  =  18
---------------
00010101  =  21
```

`eax` would then hold `21`

## test

`test` is just an `and` instruction, however, unlike `and`, `test` _doesn't_ change any operands.

## not

`not` is a simple gate, it just inverts it's operand.

| Operand | Result |
| :--- | :--- |
| 1 | 0 |
| 0 | 1 |

For example, if `eax` contained `26`, and you did:

```text
not eax
```

```text
00011010  =  26
--------
11100101  =  229
```

`eax` would then hold `229` \(`11100101`\).

