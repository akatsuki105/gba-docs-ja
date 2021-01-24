# ARMの命令一覧

CPSRのフラグの更新は全ての{S}命令で省略可能です

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

Add x=1I cycles if Op2 shifted-by-register. Add y=1S+1N cycles if Rd=R15.

Op2がレジスタによってシフトされた即値だったときは x(1I)サイクルを加えます。RdがR15だったときはy(1S+1N)サイクルを加えます。

キャリーフラグはOp2のシフト量が1以上のときに変更される可能性があります。

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

Add x=1I cycles if Op2 shifted-by-register. Add y=1S+1N cycles if Rd=R15.

## 乗算

 命令  |  サイクル | フラグ | 処理内容
---- | ---- | ---- | ----
LDR{cond}{B}{T} Rd,\${Addr}     | 1S+1N+1I+y | ---- | Rd=\[Rn+/-${offset}\]
LDR{cond}H      Rd,${Addr}     | 1S+1N+1I+y | ---- | Load Unsigned halfword
LDR{cond}D      Rd,${Addr}     |            | ---- | Load Dword ARMv5TE
LDR{cond}SB     Rd,${Addr}     | 1S+1N+1I+y | ---- | Load Signed byte
LDR{cond}SH     Rd,${Addr}     | 1S+1N+1I+y | ---- | Load Signed halfword
LDM{cond}{amod} Rn{!},${Rlist}{^} | nS+1N+1I+y | ---- | Load Multiple
STR{cond}{B}{T} Rd,\${Addr}     | 2N         | ---- | \[Rn+/-${offset}]=Rd
STR{cond}H      Rd,${Addr}     | 2N         | ---- | Store halfword
STR{cond}D      Rd,${Addr}     |            | ---- | Store Dword ARMv5TE
STM{cond}{amod} Rn{!},${Rlist}{^} | (n-1)S+2N  | ---- | Store Multiple
SWP{cond}{B}    Rd,Rm,\[Rn]       | 1S+2N+1I   | ---- | Rd=\[Rn], \[Rn]=Rm
PLD             ${Addr}        | 1S         | ---- | Prepare Cache ARMv5TE

For LDR/LDM, add y=1S+1N if Rd=R15, or if R15 in Rlist.

## ジャンプ、コール、CPSRモード、その他

 命令  |  サイクル | フラグ | 処理内容
---- | ---- | ---- | ----
B{cond}   label           | 2S+1N    | ---- | PC=$+8+/-32M
BL{cond}  label           | 2S+1N    | ---- | PC=\$+8+/-32M, LR=$+4
BX{cond}  Rn              | 2S+1N    | ---- | PC=Rn, T=Rn.0 (THUMB/ARM)
BLX{cond} Rn              | 2S+1N    | ---- | PC=Rn, T=Rn.0, LR=PC+4, ARM9
BLX       label           | 2S+1N    | ---- | PC=PC+\$+/-32M, LR=$+4, T=1, ARM9
MRS{cond} Rd,Psr          | 1S       | ---- | Rd=Psr
MSR{cond} Psr{_field},Op  | 1S       | (psr) | Psr\[field]=Op
SWI{cond} Imm24bit        | 2S+1N    | ---- | PC=8, ARM Svc mode, LR=$+4
BKPT      Imm16bit        | ???      | ---- | PC=C, ARM Abt mode, LR=$+4 ARM9
The Undefined Instruction | 2S+1I+1N | ---- | PC=4, ARM Und mode, LR=$+4
cond=false                | 1S       | ---- | Any opcode with condition=false
NOP                       | 1S       | ---- | R0=R0
CLZ{cond} Rd,Rm           | ???      | ----  | Count Leading Zeros ARMv5
QADD{cond} Rd,Rm,Rn       |          | ----q | Rd=Rm+Rn       ARMv5TE(xP)
QSUB{cond} Rd,Rm,Rn       |          | ----q | Rd=Rm-Rn       ARMv5TE(xP)
QDADD{cond} Rd,Rm,Rn      |          | ----q | Rd=Rm+Rn*2     ARMv5TE(xP)
QDSUB{cond} Rd,Rm,Rn      |          | ----q | Rd=Rm-Rn*2     ARMv5TE(xP)
