# コプロセッサ命令

MRC/MCR, LDC/STC, CDP, MCRR/MRRC が該当します

## Coprocessor Register Transfers (MRC, MCR) (with ARM Register read/write)

MRC命令は、コプロセッサレジスタからプロセッサレジスタに読み込みを行います。

MCR命令は、プロセッサレジスタからコプロセッサレジスタに書き込みを行います。

 ビット | 内容
---- | ---- 
31-28 | 条件 (オペコードがMRC2/MCR2のときは 0b1111)
27-24 | 必ず 0b1110
23-21 | cpopc - Coprocessor operation code (0-7)
20 | オペコード(後述)
19-16 | Cn - Coprocessor source/dest. Register  (C0-C15)
15-12 | Rd - 読み書き対象のARMレジスタ    (R0-R15)
11-8  | Pn - 対象のコプロセッサ (P0-P15, 例: cp15 -> 15)
7-5   | cp - Coprocessor information            (0-7)
4     | Reserved, must be one (1) (otherwise CDP opcode)
3-0   | Cm - Coprocessor operand Register       (C0-C15)

### オペコード(20bit)

 値  |  フォーマット | 処理内容
---- | ---- | ----
0b0 | `MCR{cond} Pn,<cpopc>,Rd,Cn,Cm{,<cp>}` | move from ARM to CoPro
0b0 | `MCR2      Pn,<cpopc>,Rd,Cn,Cm{,<cp>}` | move from ARM to CoPro
0b1 | `MRC{cond} Pn,<cpopc>,Rd,Cn,Cm{,<cp>}` | move from CoPro to ARM
0b1 | `MRC2      Pn,<cpopc>,Rd,Cn,Cm{,<cp>}` | move from CoPro to ARM

MCR/MRCはARMv2以降、MCR2/MRC2はARMv5以降でサポートされています。

A22i syntax allows to use MOV with Rd specified as first (dest), or last (source) operand. 

Native MCR/MRC syntax uses Rd as middle operand, \<cp\> can be ommited if \<cp\> is zero.

MCRでRdにR15を指定した場合: コプロセッサには、`PC+12`が書き込まれます。

MRCでRdにR15を指定した場合: Bit 31-28 of data are copied to Bit 31-28 of CPSR (ie. N,Z,C,V flags), other data bits are ignored, CPSR Bit 27-0 are not affected, R15 (PC) is not affected.

実行時間:

- MCR: 1S+bI+1C
- MRC: 1S+(b+1)I+1C

Return: For MRC only: Either R0-R14 modified, or flags affected (see above).

For details refer to original ARM docs. The opcodes are irrelevant for GBA/NDS7 because no coprocessor exists (except for a dummy CP14 unit). However, NDS9 includes a working CP15 unit. And, 3DS ARM11 uses CP10/CP11 as VFP floating point unit.

## Coprocessor Data Transfers (LDC, STC) (with Memory read/write)

TODO

## Coprocessor Data Operations (CDP) (without Memory or ARM Register operand)

TODO

## Coprocessor Double-Register Transfer (MCRR, MRRC) - ARMv5TE and up only

TODO


