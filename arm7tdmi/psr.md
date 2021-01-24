# PSR Transfer

MRS, MSRが該当します。

## 命令のフォーマット

### MRS

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | 0b00
25 | 0b00
24-23 | 0b10
22 | Psr - Source/Destination PSR(0=CPSR, 1=SPSR_${mode})
21 | オペコード(0b0)
20 | 0b0 (otherwise TST,TEQ,CMP,CMN)
19-16 | 0b1111 (otherwise SWP)
15-12 | Rd - 命令の結果を格納するレジスタ(R0-R14)
11-0 | 0b0000_0000_0000

### MSR

#### I=0 つまり MSR Psr, Rmのとき

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | 0b00
25 | 0b0 - I=0で第２オペランドがレジスタ
24-23 | 0b10
22 | Psr - Source/Destination PSR(0=CPSR, 1=SPSR_${mode})
21 | オペコード(0b1)
20 | 0b0 (otherwise TST,TEQ,CMP,CMN)
19 | f  write to flags field     Bit 31-24 (aka _flg)
18 | s  write to status field    Bit 23-16 (reserved, don't change) 
17 | x  write to extension field Bit 15-8  (reserved, don't change)
16 | c  write to control field   Bit 7-0   (aka _ctl) 
15-12 | Rd - 命令の結果を格納するレジスタ(R0-R14)
11-4 | 0b0000_0000 (otherwise BX)
3-0 | Rm - ソースレジスタ(R0-R14)

#### I=1 つまり MSR Psr, Immのとき

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | 0b00
25 | 0b1 - I=1で第２オペランドが即値
24-23 | 0b10
22 | Psr - Source/Destination PSR(0=CPSR, 1=SPSR_${mode})
21 | オペコード(0b1)
20 | 0b0 (otherwise TST,TEQ,CMP,CMN)
19 | f  write to flags field     Bit 31-24 (aka _flg)
18 | s  write to status field    Bit 23-16 (reserved, don't change) 
17 | x  write to extension field Bit 15-8  (reserved, don't change)
16 | c  write to control field   Bit 7-0   (aka _ctl) 
15-12 | Rd - 命令の結果を格納するレジスタ(R0-R14)
11-8 | ImmをRORシフトする量 (シフト量は1ごとに2増える つまり0b1111で30)
7-0 | Imm - 符号なし8bitの即値

Immについての注意: ソースコードでは、オペランドとして32ビットのイミディエイトを指定する必要があります。 その後、アセンブラはそれをシフトされた8ビットの値に変換しなければなりません。

## オペコード

- 0b0: `MRS{cond} Rd,Psr          ; Rd = Psr`
- 0b1: `MSR{cond} Psr{_field},Op  ; Psr[field] = Op`

## 説明

- これらのビットのうち1つ以上を設定する必要があります。例えば、CPSR_fsxc（別名CPSR、CPSR_all）はすべてのビットのロックを解除します（以下のユーザモードの制限を参照してください）。
- 非特権モード(ユーザモード)では、CPSRの条件コードビットのみ変更可能で、制御ビットは変更できません。
- ユーザモードおよびシステムモードでは、SPSR は存在しません。
- Tビットは変更できません。ARM/THUMBステートの切り替えは BX命令を使用してください。
- CPSRの未使用ビットは将来のために予約されており、決して変更してはいけません（フラグフィールドの未使用ビットを除く）。

実行時間: 1S
