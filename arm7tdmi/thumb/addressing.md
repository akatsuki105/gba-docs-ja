# アドレッシング

## THUMB.12: get relative address

PCやSPからnnだけ離れたアドレスを取得する命令です。

 bit  |  内容
---- | ----
15-12 | 0b1010
11 | オペコード
10-8 | Rd - ターゲットレジスタ  (R0..R7)
7-0  | nn - 符号無しオフセット (0-1020で1ごとに4刻み つまり0-4080)

オペコード(bit11):

```
0: ADD  Rd,PC,#nn    ; Rd = (($+4) AND NOT 2) + nn
1: ADD  Rd,SP,#nn    ; Rd = SP + nn
```

フラグ: 不変

実行時間: 1S

## THUMB.13: add offset to stack pointer

SPをnnだけ動かす命令です。

 bit  |  内容
---- | ----
15-8 | 0b10110000
7   | オペコード
6-0 | nn - 符号無しオフセット (0-508で1ごとに4)

オペコード(bit7):

```
0: ADD  SP,#nn       ; SP = SP + nn
1: ADD  SP,#-nn      ; SP = SP - nn
```

フラグ: 不変

実行時間: 1S
