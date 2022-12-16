# レジスタ操作

算術、分岐命令など

## THUMB.1: move shifted register

 bit  |  内容
---- | ----
15-13 | 0b000
12-11 | オペコード
10-6 | オフセット (0-31)
5-3 | Rs - ソースレジスタ (R0..R7)
2-0 | Rd - ターゲットレジスタ (R0..R7)

オペコード(bit12-11):

```
0: LSL{S} Rd,Rs,#Offset     ; 左シフト  Rd = Rs << nn
1: LSR{S} Rd,Rs,#Offset     ; 論理右シフト Rd = Rs >> nn
2: ASR{S} Rd,Rs,#Offset     ; 算術右シフト Rd = Rs >> nn
3: 未使用
```

- 論理右シフト: 右に溢れたbitは捨てられ、左に空いたbitは0埋めされる
- 算術右シフト: 右に溢れたbitは捨てられ、左に空いたbitは元の符号(0 or 1)で埋められる

ARMモードと同様、シフト量0のときは特殊な処理になります。

- LSL#0: シフトは行われません。キャリーの値は不変です。
- LSR/ASR#0 は LSR/ASR#32 として解釈されます。 
- ソースコードで LSR/ASR#0 を指定しようとすると、自動的に LSL#0 としてリダイレクトされます。同様に LSR/ASR#32 は LSR/ASR#0 としてリダイレクトされます。

フラグ: NZC (LSL#0のときはCは不変)

実行時間: 1S

## THUMB.2: add/subtract

 bit  |  内容
---- | ----
15-11 | 0b00011
10-9 | オペコード
8-6 | 足し引きする値(0..7 オペコードによってRnかnnかどうか変わる)
5-3 | Rs - ソースレジスタ (R0..R7)
2-0 | Rd - ターゲットレジスタ (R0..R7)

オペコード(bit10-9):

```
0: ADD{S} Rd,Rs,Rn   ; Rd=Rs+Rn
1: SUB{S} Rd,Rs,Rn   ; Rd=Rs-Rn
2: ADD{S} Rd,Rs,#nn  ; Rd=Rs+nn
3: SUB{S} Rd,Rs,#nn  ; Rd=Rs-nn
```

フラグ: NZCV (LSL#0のときはCは不変)

実行時間: 1S

## THUMB.3: move/compare/add/subtract immediate

 bit  |  内容
---- | ----
15-13 | 0b001
12-11 | オペコード        
10-8 | Rd - ターゲットレジスタ (R0..R7)
7-0  | nn - 符号無しオフセット (0-255)

オペコード(bit12-11):

```
0: MOV{S} Rd,#nn      ; Rd   = #nn
1: CMP{S} Rd,#nn      ; Void = Rd - #nn
2: ADD{S} Rd,#nn      ; Rd   = Rd + #nn
3: SUB{S} Rd,#nn      ; Rd   = Rd - #nn
```

フラグ:

- NZ: MOV
- NZCV: それ以外

実行時間: 1S

## THUMB.4: ALU operations

 bit  |  内容
---- | ----
15-10 | 010000
9-6 | オペコード
5-3 | Rs - ソースレジスタ (R0..R7)
2-0 | Rd - ターゲットレジスタ (R0..R7)

オペコード(bit9-6):

 値  |  命令 | 処理内容 | 式
---- | ---- | ---- | ---- 
0 | AND{S} Rd,Rs | AND | Rd = Rd AND Rs
1 | EOR{S} Rd,Rs | XOR | Rd = Rd XOR Rs
2 | LSL{S} Rd,Rs | 論理左シフト | Rd = Rd << (Rs AND 0FFh)
3 | LSR{S} Rd,Rs | 論理右シフト |  Rd = Rd >> (Rs AND 0FFh)
4 | ASR{S} Rd,Rs | 算術右シフト |  Rd = Rd SAR (Rs AND 0FFh)
5 | ADC{S} Rd,Rs | 足し算(+キャリー)   |  Rd = Rd + Rs + Carry
6 | SBC{S} Rd,Rs | 引き算(+キャリー)   |  Rd = Rd - Rs - NOT Carry
7 | ROR{S} Rd,Rs | ROR    |  Rd = Rd ROR (Rs AND 0FFh)
8 | TST    Rd,Rs | bitチェック             | Void = Rd AND Rs
9 | NEG{S} Rd,Rs | 反転           |   Rd = 0 - Rs
A | CMP    Rd,Rs | 比較          | Void = Rd - Rs
B | CMN    Rd,Rs | 比較(反転)      | Void = Rd + Rs
C | ORR{S} Rd,Rs | OR       |   Rd = Rd OR Rs
D | MUL{S} Rd,Rs | 乗算         |   Rd = Rd * Rs
E | BIC{S} Rd,Rs | bitクリア        |   Rd = Rd AND NOT Rs
F | MVN{S} Rd,Rs | 否定              |   Rd = NOT Rs

フラグ:

- NZCV: ADC,SBC,NEG,CMP,CMN
- NZC: LSL,LSR,ASR,ROR (シフト量が0のときはCは不変)
- NZC: MUL (Cは必ずクリア)
- NZ: AND,EOR,TST,ORR,BIC,MVN

実行時間:

- AND,EOR,ADC,SBC,TST,NEG,CMP,CMN,ORR,BIC,MVN: 1S
- LSL,LSR,ASR,ROR: 1S+1I
- MUL: 1S+mI (m=1..4; 入力Rd値の最上位bitに依存)

## THUMB.5: Hi register operations/branch exchange

THUMBモードで通常操作できないR8-R15を操作する命令です。

 bit  |  内容
---- | ----
15-10 | 0b010001
9-8 | オペコード          
7   | MSBd - ターゲットレジスタのMSB
6   | MSBs - ソースレジスタのMSB
5-3 | Rs - ソースレジスタ (MSBsと合わせて R0..R15)
2-0 | Rd - ターゲットレジスタ (MSBdと合わせて R0..R15)

オペコード(bit9-8):

```
0: ADD Rd,Rs   ; add        Rd = Rd+Rs
1: CMP Rd,Rs   ; compare    Void = Rd-Rs  ; CPSR affected
2: MOV Rd,Rs   ; move       Rd = Rs
2: NOP         ; nop        R8 = R8
3: BX  Rs      ; jump       PC = Rs
```

ADD, CMP, MOVの場合、MSBsかMSBdのどちらかがセットされている、つまりソースレジスタかターゲットレジスタのどちらか(または両方)がR8-R15の必要があります。

R15がオペランドのとき、R15に入っているのはこの命令のあるアドレスに4を加えたもの(\$+4)であることに注意してください。

### BX

BXにはいくつか注意点があります

まずMSBdは必ずクリアされている必要があります。Rdは使用されません。

またRsレジスタに格納されている値(つまりジャンプ先)のbit0が0の場合、ARMモードに切り替わり、(アラインメントのため)Rsのbit1はクリアされます。そのため`BX R15`は必ずワードでアラインメントされたアドレスで行う必要があり、ジャンプ先は上述の通り`PC+4`となります。

### その他

THUMBモードにおいて`MOV R8,R8`は`NOP`として扱われます。

フラグ: 

- NZCV: CMP
- 不変: それ以外

実行時間:

- ADD,MOV,CMP: 1S
- ADD(`Rd=R15`), MOV(`Rd=R15`), BX: 2S+1N

