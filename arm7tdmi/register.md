# レジスタ

## 概要

次のテーブルはARM7TDMIの各モードで利用可能なレジスタの一覧です。

全部で37個の32bitレジスタがあり、そのうち31個は汎用レジスタ(Rxx)で6個はステータスレジスタです(xPSR)。

いくつかのレジスタはバンク方式であることに注意してください。例をあげると、それぞれのモードはそれぞれ独自のR14レジスタを持っています。

もちろん、バンク方式でないレジスタもあります。例えばR0はどのモードも共有しています。

 System | User FIQ | Supervisor | Abort | IRQ | Undefined
---- | ---- | ---- | ---- | ---- | ----
R0 | R0 | R0 | R0 | R0 | R0 
R1 | R1 | R1 | R1 | R1 | R1 
R2 | R2 | R2 | R2 | R2 | R2 
R3 | R3 | R3 | R3 | R3 | R3 
R4 | R4 | R4 | R4 | R4 | R4 
R5 | R5 | R5 | R5 | R5 | R5 
R6 | R6 | R6 | R6 | R6 | R6 
R7 | R7 | R7 | R7 | R7 | R7 
R8 | R8_fiq | R8 | R8 | R8 | R8 
R9 | R9_fiq | R9 | R9 | R9 | R9 
R10 | R10_fiq | R10 | R10 | R10 | R10 
R11 | R11_fiq | R11 | R11 | R11 | R11 
R12 | R12_fiq | R12 | R12 | R12 | R12 
R13 | R13_fiq | R13_svc | R13_abt | R13_irq | R13_und 
R14 | R14_fiq | R14_svc | R14_abt | R14_irq | R14_und 
R15 | R15 | R15 | R15 | R15 | R15 
CPSR | CPSR | CPSR | CPSR | CPSR | CPSR 
-- | SPSR_fiq | SPSR_svc | SPSR_abt | SPSR_irq | SPSR_und 

## R0-R12 (汎用レジスタ)

これらの13個のレジスタは、さまざまな目的に使用可能です。

基本的には、それぞれのレジスタの機能や性能は同じで、演算用の「高速アキュムレータ」やメモリアドレス指定用の「ポインタレジスタ」のような特殊な用途に限定されたレジスタはありません。

ただし、THUMBモードではR0～R7(Loレジスタ)のみ自由にアクセスでき、R8～R12(Hiレジスタ)は一部の命令でしかアクセスできません。

## R13 (SP)

このレジスタはTHUMBモードではスタックポインタ(SP)として使用されます。

ARMモードでは、R13ではなく他のレジスタをスタックポインタとして使用したり、R13を汎用レジスタとして使用したりすることができます。

上のテーブルからわかるように、各モードには個別のR13レジスタがあり、（SPとして使用されている場合は）各モードの例外ハンドラは独自のスタックを使用することができます。

## R14 (LR)

このレジスタはリンクレジスタ(LR)として使用されます。リンク付き分岐命令(BL)でサブルーチンを呼び出すと、そのリターン先のアドレス(PCの古い値)がこのレジスタに保存されます。

サブルーチンをネストして呼び出す場合、LRレジスタは各モードに1つしかないので、リターンアドレスを手動でプッシュする必要があります。

また、例外が呼ばれた場合も同様で、PCは新しいモードのLRに保存されます。

ARMモードでは、上記のLRレジスタとしての使用が不要であれば、R14を汎用レジスタとして使用することも可能になっています。

## R15 (PC)

R15は常にプログラムカウンタ(PC)として利用されます。

R15から読み出しを行ったとき、パイプライン処理のため基本的に`$+nn`の値が返されることに注意してください。（つまり今実行している命令のアドレス(`$`)よりR15のほうが進んでいます）

`$+nn`の`nn`はARMモードかTHUMBモードかによって変わります。

## CPSR と SPSR

PSRは `Program Status Registers`の略です。 CPSRは `Current PSR`, SPSRは `Saved PSR` の略です。

例外が起こると、古いCPSRが例外のモードに応じたSPSRに退避されます。(PCがLRに退避されるのに似ています。)

<table>
    <thead>
        <tr>
            <th>31</th>
            <th>30</th>
            <th>29</th>
            <th>28</th>
            <th>27</th>
            <th>26-8</th>
            <th>7</th>
            <th>6</th>
            <th>5</th>
            <th>4-0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td colspan=1 class="td-colspan">N</td>
            <td colspan=1 class="td-colspan">Z</td>
            <td colspan=1 class="td-colspan">C</td>
            <td colspan=1 class="td-colspan">V</td>
            <td colspan=1 class="td-colspan">Q</td>
            <td colspan=1 class="td-colspan">Reserved</td>
            <td colspan=1 class="td-colspan">I</td>
            <td colspan=1 class="td-colspan">F</td>
            <td colspan=1 class="td-colspan">T</td>
            <td colspan=1 class="td-colspan">Mode</td>
        </tr>
    </tbody>
</table>
