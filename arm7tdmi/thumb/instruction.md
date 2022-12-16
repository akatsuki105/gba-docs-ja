# THUMBモードの命令一覧

THUMBモードはARMモードと違って基本的に汎用レジスタをR0-R7までしか利用できません。

サイクルのS, Nなどについては[サイクル](../cycle.md)を参照してください。

## 論理演算

 命令  |  サイクル | フラグ | フォーマット | 処理内容
---- | ---- | ---- | ---- | ----
MOV Rd,Imm8bit    | 1S    | NZ-- | 3 | Rd=nn
MOV Rd,Rs         | 1S    | NZ00 | 2 | Rd=Rs+0
MOV R0..14,R8..15 | 1S    | ---- | 5 | Rd=Rs
MOV R8..14,R0..15 | 1S    | ---- | 5 | Rd=Rs
MOV R15,R0..15    | 2S+1N | ---- | 5 | PC=Rs
MVN Rd,Rs         | 1S    | NZ-- | 4 | Rd=NOT Rs
AND Rd,Rs         | 1S    | NZ-- | 4 | Rd=Rd AND Rs
TST Rd,Rs         | 1S    | NZ-- | 4 | Void=Rd AND Rs
BIC Rd,Rs         | 1S    | NZ-- | 4 | Rd=Rd AND NOT Rs
ORR Rd,Rs         | 1S    | NZ-- | 4 | Rd=Rd OR Rs
EOR Rd,Rs         | 1S    | NZ-- | 4 | Rd=Rd XOR Rs
LSL Rd,Rs,Imm5bit | 1S    | NZc- | 1 | Rd=Rs SHL nn
LSL Rd,Rs         | 1S+1I | NZc- | 4 | Rd=Rd SHL (Rs AND 0FFh)
LSR Rd,Rs,Imm5bit | 1S    | NZc- | 1 | Rd=Rs SHR nn
LSR Rd,Rs         | 1S+1I | NZc- | 4 | Rd=Rd SHR (Rs AND 0FFh)
ASR Rd,Rs,Imm5bit | 1S    | NZc- | 1 | Rd=Rs SAR nn
ASR Rd,Rs         | 1S+1I | NZc- | 4 | Rd=Rd SAR (Rs AND 0FFh)
ROR Rd,Rs         | 1S+1I | NZc- | 4 | Rd=Rd ROR (Rs AND 0FFh)
NOP               | 1S    | ---- | 5 | R8=R8

## 算術演算+乗算

 命令  |  サイクル | フラグ | フォーマット | 処理内容
---- | ---- | ---- | ---- | ----
ADD Rd,Rs,Imm3bit   | 1S    | NZCV |  2 | Rd=Rs+nn
ADD Rd,Imm8bit      | 1S    | NZCV |  3 | Rd=Rd+nn
ADD Rd,Rs,Rn        | 1S    | NZCV |  2 | Rd=Rs+Rn
ADD R0..14,R8..15   | 1S    | ---- |  5 | Rd=Rd+Rs
ADD R8..14,R0..15   | 1S    | ---- |  5 | Rd=Rd+Rs
ADD R15,R0..15      | 2S+1N | ---- |  5 | PC=Rd+Rs
ADD Rd,PC,Imm8bit*4 | 1S    | ---- | 12 | Rd=(($+4) AND NOT 2)+nn
ADD Rd,SP,Imm8bit*4 | 1S    | ---- | 12 | Rd=SP+nn
ADD SP,Imm7bit*4    | 1S    | ---- | 13 | SP=SP+nn
ADD SP,-Imm7bit*4   | 1S    | ---- | 13 | SP=SP-nn
ADC Rd,Rs           | 1S    | NZCV |  4 | Rd=Rd+Rs+Cy
SUB Rd,Rs,Imm3Bit   | 1S    | NZCV |  2 | Rd=Rs-nn
SUB Rd,Imm8bit      | 1S    | NZCV |  3 | Rd=Rd-nn
SUB Rd,Rs,Rn        | 1S    | NZCV |  2 | Rd=Rs-Rn
SBC Rd,Rs           | 1S    | NZCV |  4 | Rd=Rd-Rs-NOT Cy
NEG Rd,Rs           | 1S    | NZCV |  4 | Rd=0-Rs
CMP Rd,Imm8bit      | 1S    | NZCV |  3 | Void=Rd-nn
CMP Rd,Rs           | 1S    | NZCV |  4 | Void=Rd-Rs
CMP R0-15,R8-15     | 1S    | NZCV |  5 | Void=Rd-Rs
CMP R8-15,R0-15     | 1S    | NZCV |  5 | Void=Rd-Rs
CMN Rd,Rs           | 1S    | NZCV |  4 | Void=Rd+Rs
MUL Rd,Rs           | 1S+mI | NZx- |  4 | Rd=Rd*Rs

## ロード/ストア

 命令 | サイクル | フラグ | フォーマット | 処理内容
---- | ---- | ---- | ---- | ----
LDR  Rd,\[Rb,5bit*4\] | 1S+1N+1I  | ---- |  9 | Rd = WORD\[Rb+nn\]
LDR  Rd,\[PC,8bit*4\] | 1S+1N+1I  | ---- |  6 | Rd = WORD\[PC+nn\]
LDR  Rd,\[SP,8bit*4\] | 1S+1N+1I  | ---- | 11 | Rd = WORD\[SP+nn\]
LDR  Rd,\[Rb,Ro\]     | 1S+1N+1I  | ---- |  7 | Rd = WORD\[Rb+Ro\]
LDRB Rd,\[Rb,5bit*1\] | 1S+1N+1I  | ---- |  9 | Rd = BYTE\[Rb+nn\]
LDRB Rd,\[Rb,Ro\]     | 1S+1N+1I  | ---- |  7 | Rd = BYTE\[Rb+Ro\]
LDRH Rd,\[Rb,5bit*2\] | 1S+1N+1I  | ---- | 10 | Rd = HALFWORD\[Rb+nn\]
LDRH Rd,\[Rb,Ro\]     | 1S+1N+1I  | ---- |  8 | Rd = HALFWORD\[Rb+Ro\]
LDSB Rd,\[Rb,Ro\]     | 1S+1N+1I  | ---- |  8 | Rd = SIGNED_BYTE\[Rb+Ro\]
LDSH Rd,\[Rb,Ro\]     | 1S+1N+1I  | ---- |  8 | Rd = SIGNED_HALFWORD\[Rb+Ro\]
STR  Rd,\[Rb,5bit*4\] | 2N        | ---- |  9 | WORD\[Rb+nn\] = Rd
STR  Rd,\[SP,8bit*4\] | 2N        | ---- | 11 | WORD\[SP+nn\] = Rd
STR  Rd,\[Rb,Ro\]     | 2N        | ---- |  7 | WORD\[Rb+Ro\] = Rd
STRB Rd,\[Rb,5bit*1\] | 2N        | ---- |  9 | BYTE\[Rb+nn\] = Rd
STRB Rd,\[Rb,Ro\]     | 2N        | ---- |  7 | BYTE\[Rb+Ro\] = Rd
STRH Rd,\[Rb,5bit*2\] | 2N        | ---- | 10 | HALFWORD\[Rb+nn\] = Rd
STRH Rd,\[Rb,Ro\]     | 2N        | ---- |  8 | HALFWORD\[Rb+Ro\]=Rd
PUSH {Rlist}{LR}    | (n-1)S+2N | ---- | 14 |
POP  {Rlist}{PC}    |           | ---- | 14 | (ARM9: with mode switch)
STMIA Rb!,{Rlist}   | (n-1)S+2N | ---- | 15 |
LDMIA Rb!,{Rlist}   | nS+1N+1I  | ---- | 15 |

## ジャンプ、コール

 命令  |  サイクル | フラグ | フォーマット | 処理内容($=PC)
---- | ---- | ---- | ---- | ----
B disp             | 2S+1N        | ---- | 18  | PC=$+/-2048
BL disp            | 3S+1N        | ---- | 19  | PC=\$+/-4M, LR=\$+5
B{cond=true} disp  | 2S+1N        | ---- | 16  | PC=$+/-0..256
B{cond=false} disp | 1S           | ---- | 16  | N/A
BX R0..15          | 2S+1N        | ---- |  5  | PC=Rs, ARM/THUMB (Rs bit0)
SWI Imm8bit        | 2S+1N        | ---- | 17  | PC=8, ARM SVC mode, LR=$+2
POP {Rlist,}PC     | (n+1)S+2N+1I | ---- | 14  | 
MOV R15,R0..15     | 2S+1N        | ---- |  5  | PC=Rs
ADD R15,R0..15     | 2S+1N        | ---- |  5  | PC=Rd+Rs

BL命令は全体で32bit長であることに注意してください。
