# ロード/ストア命令

## THUMB.6: load PC-relative

PCからの相対アドレスで指定したメモリから32bitデータをロード/ストアする命令です。リテラルプールからのデータ読み込みで頻繁に利用します。

```
  LDR Rd,[PC,#nn]
```

 bit  |  内容
---- | ----
15-11 | 0b01001
10-8 | Rd - 読み取った値を格納するレジスタ  (R0..R7)
7-0  | nn - 符号無しオフセット (1増えるとオフセットが4増加。つまりnnの範囲は 0..1020)

PCは`($+4) AND NOT 2` として扱われます。(`$`: この命令のあるアドレス)

フラグ: 不変

実行時間: 1S+1N+1I

## THUMB.7: load/store with register offset

RbレジスタとRoレジスタに入っているオフセットで指定したメモリから32bitデータ(または8bitデータ)をロード/ストアする命令です。

 bit  |  内容
---- | ----
15-12 | 0b0101
11-10 | オペコード        
9   | 0b0
8-6 | Ro - オフセットレジスタ (R0..R7)
5-3 | Rb - ベースレジスタ (R0..R7)
2-0 | Rd - ターゲットレジスタ (R0..R7)

フラグ: 不変

実行時間: 

- LDR: 1S+1N+1I
- STR: 2N

オペコード(bit11-10):

```
0: STR  Rd,[Rb,Ro]   ; 32bit格納    WORD[Rb+Ro] = Rd
1: STRB Rd,[Rb,Ro]   ; 8bit格納     BYTE[Rb+Ro] = Rd
2: LDR  Rd,[Rb,Ro]   ; 32bitロード  Rd = WORD[Rb+Ro]
3: LDRB Rd,[Rb,Ro]   ; 8bitロード   Rd = BYTE[Rb+Ro]
```

## THUMB.8: load/store sign-extended byte/halfword

 bit  |  内容
---- | ----
15-12 | 0b0101
11-10 | オペコード        
9   | 0b1
8-6 | Ro - オフセットレジスタ (R0..R7)
5-3 | Rb - ベースレジスタ (R0..R7)
2-0 | Rd - ターゲットレジスタ (R0..R7)

フラグ: 不変

実行時間: 

- LDR: 1S+1N+1I
- STR: 2N

オペコード(bit11-10):

```
0: STRH Rd,[Rb,Ro] ; 16bit格納               HALFWORD[Rb+Ro] = Rd
1: LDSB Rd,[Rb,Ro] ; 符号付き8bitをロード      Rd = BYTE[Rb+Ro]
2: LDRH Rd,[Rb,Ro] ; 16bitをロード           Rd = HALFWORD[Rb+Ro]
3: LDSH Rd,[Rb,Ro] ; 符号付き16bitをロード     Rd = HALFWORD[Rb+Ro]
```

## THUMB.9: load/store with immediate offset

Rbレジスタと即値オフセットで指定したメモリから32bitデータ(または8bitデータ)をロード/ストアする命令です。

 bit  |  内容
---- | ----
15-13 | 0b011
12-11 | オペコード        
10-6 | nn - 符号無しオフセット (0-31 for BYTE, 0-124 for WORD)
5-3 | Rb - ベースレジスタ (R0..R7)
2-0 | Rd - ターゲットレジスタ (R0..R7)

オペコード(bit12-11):

```
0: STR  Rd,[Rb,#nn]  ; store 32bit data   WORD[Rb+nn] = Rd
1: LDR  Rd,[Rb,#nn]  ; load  32bit data   Rd = WORD[Rb+nn]
2: STRB Rd,[Rb,#nn]  ; store  8bit data   BYTE[Rb+nn] = Rd
3: LDRB Rd,[Rb,#nn]  ; load   8bit data   Rd = BYTE[Rb+nn]
```

フラグ: 不変

実行時間: 

- LDR: 1S+1N+1I
- STR: 2N

## THUMB.10: load/store halfword

Rbレジスタと即値オフセットで指定したメモリから16bitデータをロード/ストアする命令です。

 bit  |  内容
---- | ----
15-12 | 0b1000
11 | オペコード        
10-6 | nn - 符号無しオフセット (0-62、2刻み)
5-3 | Rb - ベースレジスタ (R0..R7)
2-0 | Rd - ターゲットレジスタ (R0..R7)

オペコード(bit11):

```
0: STRH Rd,[Rb,#nn]  ; store 16bit data   HALFWORD[Rb+nn] = Rd
1: LDRH Rd,[Rb,#nn]  ; load  16bit data   Rd = HALFWORD[Rb+nn]
```

フラグ: 不変

実行時間: 

- LDR: 1S+1N+1I
- STR: 2N

## THUMB.11: load/store SP-relative

SPからの相対アドレスで指定したメモリから32bitデータをロード/ストアする命令です。

 bit  |  内容
---- | ----
15-12 | 0b1001
11 | オペコード
10-8 | Rd - ターゲットレジスタ (R0..R7)
7-0  | nn - 符号無しオフセット (0-1020, 4刻み)

オペコード(bit11):

```
0: STR  Rd,[SP,#nn]  ; store 32bit data   WORD[SP+nn] = Rd
1: LDR  Rd,[SP,#nn]  ; load  32bit data   Rd = WORD[SP+nn]
```

フラグ: 不変

実行時間: 

- LDR: 1S+1N+1I
- STR: 2N
