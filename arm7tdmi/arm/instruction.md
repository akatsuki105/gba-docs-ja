# ARMモードの命令一覧

CPSRのフラグの更新は全ての{S}命令で省略可能です

サイクルのS, Nなどについては[サイクル](./cycle.md)を参照してください。

## 論理演算

 命令  |  サイクル | フラグ | 処理内容
---- | ---- | ---- | ----
MOV{cond}{S} Rd,Op2    | 1S+x+y | NZc-  | Rd = Op2
MVN{cond}{S} Rd,Op2    | 1S+x+y | NZc-  | Rd = NOT Op2
ORR{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZc-  | Rd = Rn OR Op2
EOR{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZc-  | Rd = Rn XOR Op2
AND{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZc-  | Rd = Rn AND Op2
BIC{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZc-  | Rd = Rn AND NOT Op2
TST{cond}{P}    Rn,Op2 | 1S+x   | NZc-  | Void = Rn AND Op2
TEQ{cond}{P}    Rn,Op2 | 1S+x   | NZc-  | Void = Rn XOR Op2

- Op2がレジスタによってシフトされる場合、`x=1I`となります。
- `Rd=R15`の場合、`y=1S+1N`となります。

キャリーフラグはOp2のシフト量が1以上のときに更新されます。

## 算術演算

 命令  |  サイクル | フラグ | 処理内容
---- | ---- | ---- | ----
ADD{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZCV | Rd = Rn+Op2
ADC{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZCV | Rd = Rn+Op2+Cy
SUB{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZCV | Rd = Rn-Op2
SBC{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZCV | Rd = Rn-Op2+Cy-1
RSB{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZCV | Rd = Op2-Rn
RSC{cond}{S} Rd,Rn,Op2 | 1S+x+y | NZCV | Rd = Op2-Rn+Cy-1
CMP{cond}{P}    Rn,Op2 | 1S+x   | NZCV | Void = Rn-Op2
CMN{cond}{P}    Rn,Op2 | 1S+x   | NZCV | Void = Rn+Op2

- Op2がレジスタによってシフトされる場合、`x=1I`となります。
- `Rd=R15`の場合、`y=1S+1N`となります。

## 乗算

 命令  |  サイクル | フラグ | 処理内容
---- | ---- | ---- | ----
MUL{cond}{S} Rd,Rm,Rs          | 1S+mI    | NZx- | Rd = Rm×Rs
MLA{cond}{S} Rd,Rm,Rs,Rn       | 1S+mI+1I | NZx- | Rd = Rm×Rs+Rn
UMULL{cond}{S} RdLo,RdHi,Rm,Rs | 1S+mI+1I | NZx- | RdHiLo = Rm×Rs
UMLAL{cond}{S} RdLo,RdHi,Rm,Rs | 1S+mI+2I | NZx- | RdHiLo = Rm×Rs+RdHiLo
SMULL{cond}{S} RdLo,RdHi,Rm,Rs | 1S+mI+1I | NZx- | RdHiLo = Rm×Rs
SMLAL{cond}{S} RdLo,RdHi,Rm,Rs | 1S+mI+2I | NZx- | RdHiLo = Rm×Rs+RdHiLo

## ロード/ストア

 命令  |  サイクル | フラグ | 処理内容
---- | ---- | ---- | ----
LDR{cond}{B}{T} Rd,\${Addr}     | 1S+1N+1I+y | ---- | Rd=\[Rn+/-${offset}\]
LDR{cond}H      Rd,${Addr}     | 1S+1N+1I+y | ---- | Load Unsigned halfword
LDR{cond}SB     Rd,${Addr}     | 1S+1N+1I+y | ---- | Load Signed byte
LDR{cond}SH     Rd,${Addr}     | 1S+1N+1I+y | ---- | Load Signed halfword
LDM{cond}{amod} Rn{!},${Rlist}{^} | nS+1N+1I+y | ---- | Load Multiple
STR{cond}{B}{T} Rd,\${Addr}     | 2N         | ---- | \[Rn+/-${offset}]=Rd
STR{cond}H      Rd,${Addr}     | 2N         | ---- | Store halfword
STM{cond}{amod} Rn{!},${Rlist}{^} | (n-1)S+2N  | ---- | Store Multiple
SWP{cond}{B}    Rd,Rm,\[Rn]       | 1S+2N+1I   | ---- | Rd=\[Rn], \[Rn]=Rm

LDR/LDMでは、`Rd=R15` または `R15 in Rlist` のとき、`y=1S+1N` となります。

## ジャンプ、コール、CPSRモード、その他

 命令  |  サイクル | フラグ | 処理内容
---- | ---- | ---- | ----
B{cond}   label           | 2S+1N    | ---- | PC=$+8+/-32M
BL{cond}  label           | 2S+1N    | ---- | PC=\$+8+/-32M, LR=$+4
BX{cond}  Rn              | 2S+1N    | ---- | PC=Rn, T=Rn.0 (THUMB/ARM)
MRS{cond} Rd,Psr          | 1S       | ---- | Rd=Psr
MSR{cond} Psr{_field},Op  | 1S       | (psr) | Psr\[field]=Op
SWI{cond} Imm24bit        | 2S+1N    | ---- | PC=8, ARM Svc mode, LR=$+4
The Undefined Instruction | 2S+1I+1N | ---- | PC=4, ARM Und mode, LR=$+4
cond=false                | 1S       | ---- | Any opcode with condition=false
NOP                       | 1S       | ---- | R0=R0
