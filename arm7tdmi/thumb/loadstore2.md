# ロード/ストア命令 その2

## THUMB.14: push/pop registers

 bit  |  内容
---- | ----
15-12 | 0b1011
11 | オペコード
10-9 | 0b10
8 | PC/LR Bit (0: No, 1: `PUSH LR (R14)`, or `POP PC (R15)`)
7-0 | Rlist - レジスタのリスト (R7..R0)

オペコード(bit11):

```
0: PUSH {Rlist}{LR}   ; store in memory, decrements SP (R13)
1: POP  {Rlist}{PC}   ; load from memory, increments SP (R13)
```

例:

```
 PUSH {R0-R3}     ; push R0,R1,R2,R3
 PUSH {R0,R2,LR}  ; push R0,R2,LR
 POP  {R4,R7}     ; pop R4,R7
 POP  {R2-R4,PC}  ; pop R2,R3,R4,PC
```

### サブルーチン

サブルーチンを呼び出す際には、LRレジスタにリターンアドレスを格納する必要があります。

ネストして呼び出す場合は`PUSH {LR}`をしてLRレジスタの値をまずスタックに退避する必要があります。逆にサブルーチンから戻る時は`POP {PC}`をしてください。

POP {PC} ignores the least significant bit of the return address (processor remains in thumb state even if bit0 was cleared), when intending to return with optional mode switch, use a POP/BX combination (eg. POP {R3} / BX R3).
ARM9: POP {PC} copies the LSB to thumb bit (switches to ARM if bit0=0).

フラグ: 不変

実行時間:

- PUSH: (n-1)S+2N
- POP: nS+1N+1I
- POP PC: (n+1)S+2N+1I

## THUMB.15: multiple load/store

 bit  |  内容
---- | ----
15-12 | 0b1100
11 | オペコード          
10-8 | Rb - ベースレジスタ (R0-R7)
7-0  | Rlist - レジスタのリスト (R7..R0)

オペコード(bit11):

```
0: STMIA Rb!,{Rlist}   ; store in memory, increments Rb
1: LDMIA Rb!,{Rlist}   ; load from memory, increments Rb
```

例:

```
 STMIA R7!,{R0-R2}  ; [R7]=R0, [R7+4]=R1, [R7+8]=R2
 LDMIA R0!,{R1,R5}  ; R1=[R0], R5=[R0+4]
```

フラグ: 不変

実行時間:

- LDM: nS+1N+1I
- STM: (n-1)S+2N

## 順番

[ARMモードの場合](../arm/loadstore3.md#順番)と同じです。
