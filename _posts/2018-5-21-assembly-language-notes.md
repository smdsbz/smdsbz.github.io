---
layout: post
title: Assembly Language Notes
---

{:toc}

### MASM Code

```assembly
.386

DATA    SEGMENT USE16
    ...
DATA    ENDS

STACK   SEGMENT UES16 STACK
    DB xxx DUP(0)
STACK   ENDS

CODE    SEGMENT USE16
ASSUME  CS:CODE, DS:DATA, SS:STACK

START:
    MOV     AX, DATA
    MOV     DS, AX
    ...
    MOV     AH, 4CH
    INT     21H

CODE    ENDS
    END START

```

```assembly
IOSTR   MACRO A, N
    ...
    ENDM
```

### DOS System Calls

- Keyboard input
    ```assembly
    MOV     AH, 1
    INT     21H
    ; send ASCII to AL
    ```
- Character output
    ```assembly
    MOV     DL, CHAR    ; send CHAR to output to DL
    MOV     AH, 2
    INT     21H
    ```
- String output
    ```assembly
    LEA     DX, STR     ; pointer to a '$' terminated string
    MOV     AH, 9
    INT     21H
    ```
- String input
    ```assembly
    LEA     DX, BUF     ; BUF with structure (count)(input_length)(buffer)
    MOV     AH, 10
    INT     21H
    ```



### Physical Address

```assembly
(SS) = 1000H,
[EBP + ESI] = SS << 4 + EBP + ESI (real mode)
```

段首址左移 4 位后再加。


### Comparison

|                            | Signed  | Unsigned |
|:-------------------------- |:-------:|:--------:|
| Equal to                   | `JE`    | `JE`     |
| Not Equal to               | `JNE`   | `JNE`    |
| Greater than (or Equal to) | `JG(E)` | -        |
| Less than (or Equal to)    | `JL(E)` | -        |
| Above (or Even)            | -       | `JA(E)`  |
| Below (or Even)            | -       | `JB(E)`  |


### Addressing and Registers

- Register Indirect Addressing - 寄存器间接寻址
    - Format:
        - `[R]`
    - Register Availability:
        - All 32-bit registers
        - `BX`, `DI`, `SI`, `BP`
    - Default Segment:
        - `BP`, `EBP`, `ESP` goes to stack segment
- Indexed Addressing - 变址寻址
    - Format:
        - `[R*F+V]`, `[R*F]+V`, `V[R*F]`
    - Register Availability:
        - All 32-bit registers
    - Limitations:
        - `F` must be 1 when `R` is 16-bit register or `ESP`
    - Default Bit Width:
        - Depend on `V` if `V` is variable or label
- Based Indexed Addressing - 基址加变址寻址
    - **NOTE:** With two registers.
    - Format:
        - `[BR+IR*F+V]`, `V[BR+IR*F]`, `V[BR][IR*F]`
    - Register Availability:
        - All 32-bit registers (`IR` can't be `ESP`)
        - `BR`: `BX`, `BP`
        - `IR`: `SI`, `DI`
    - Default Segment:
        - When `BR` is `BX`, goes to data segment
        - When `BR` is `BP`, `ESP`, `EBP`, goes to stack segment


### Bit Length

- `PUSH AL` 非法：压/出栈操作数应为 16/32 位。


### Sub-Process

- `INVOKE` `STDCALL`：从右到左顺序进栈。

`INVOKE ITOA, 1234H, 10, OFFSET BUF`
```assembly
RADIX_S PROC NEAR STDCALL USES EBX EDX SI, LpResult, Radix:DWORD, NUM:DWORD
    LOCAL   COUNT:WORD
    ...
RADIX_S     ENDP
```

stack structure:

**Note:** Stack grows from **TOP**, goes **DOWN**.

| address | content  | comment        |
|:-------:|:--------:|:--------------:|
| *high*  | ...      |                |
|         | `00H`    |                |
| `+8`    | `1FH`    | `OFFSET BUF`   |
|         | `00H`    |                |
| `+6`    | `0AH`    | `10` as `WORD` |
|         | `12H`    |                |
| `+4`    | `34H`    | `1234H`        |
|         | `00H`    |                |
| `+2`    | `10H`    | `IP` to next line under `INVOKE` |
|         | `(BP-H)` |                |
| `0`     | `(BP-L)` | `(BP)=(SP)`    |
| *low*   | ...      |                |

| content |
|:-------:|
| *low*   |
| (SI)    |
|         |
| (EDX)   |
|         |
| (EBX)   |
| COUNT   |
| (BP)    |
| (IP)    |
| LpResult|
|         |
| Radix   |
|         |
| NUM     |
| *high*  |

`CALL` only pushes `IP` (for `FAR`, push `CS` then `IP`), finally pushes `BP`.

```assembly
STRCMP  PROC NEAR
    PUSH    BP
    MOV     BP, SP
    MOV     SI, [BP+4]      ; take first parameter to SI
                            ; +4 = sizeof(BP) + sizeof(IP)
    ...
```


### Interupt Service Process

Number `10H`: 4 bits, from `0:40H` to `0:43H` (`40H` = `10H` * `(4)_10`)

- Interupt is done by CPU.

#### What happened when an interupt process is invoked?

Under real mode, when executing `INT m`
1. `PUSHF` - save flag register status
2. push `CS` and `IP` successively
3. `4 * m` to `IP`, `4 * m + 2` to `CS`


### Sub-Process

- `NEAR`: `CS` is not pushed with `NEAR` jumps.
- `STDCALL`: parameters are pushed into stack from right to left.


### Win32 Window Programming

#### Tasks of `WinMain()`

1. Initialize window parameters, register the window class
2. Create window instance
3. Load resources, e.g. menus
4. Message looooooop
5. On recieving quit message, quit


### Clever Hacks

- `n << 1 == n * 2 == n + n`

