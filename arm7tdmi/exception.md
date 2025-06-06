# 例外

次のテーブルはメモリ上にある例外ベクタです。例外が生じた時、CPUはARMモードになりPCがそれぞれのアドレスによってロードされます。

 アドレス | 優先度 | 例外 | エントリモード | 割り込みフラグ
---- | ---- | ---- | ---- | ----
BASE + 0x00 | 1 | リセット | _svc | I=1, F=1
BASE + 0x04 | 7 | 未定義命令 | _und | I=1, F=不変
BASE + 0x08 | 6 | ソフトウェア割り込み(SWI) | _svc | I=1, F=不変
BASE + 0x0c | 5 | プリフェッチアボート | _abt | I=1, F=不変
BASE + 0x10 | 2 | データアボート | _abt | I=1, F=不変
BASE + 0x14 | ?? | Address Exceeds 26bit | _svc | I=1, F=不変
BASE + 0x18 | 4 | 割り込み(IRQ) | _irq | I=1, F=不変
BASE + 0x1c | 4 | 高速割り込み(FIQ) | _irq | I=1, F=1

BASEは`0x0000_0000`か`0xffff_0000`のどちらかです。

優先度は1~7まであり、1が最高つまり最優先です。

上記の各アドレスにはARMオペコード1つ分のスペースしかないので、通常は各ベクタにBranchオペコードを入れておいて、実際に例外処理を行うハンドラのアドレスにリダイレクトするのが普通です。

## 例外発生時にCPUが行う処理

- `R14_${new mode} = $+nn`: 古いPC、つまりリターンアドレスを保存します。
- `SPSR_${new mode} = CPSR`: 古いPSRを保存します。
- `CPSRのT,Mを更新`: T=0にしてARMモードにし、M4-0を新しいモードに更新
- `CPSRのIを更新`: I=1にして割り込みを無効にします。
- `CPSRのFを更新`: F=1にして高速割り込みを無効にします。リセットとFIQによる例外時にのみ行われます
- `PC=割りこみベクタのアドレス`

上記の`$+nn`は例外の種類によって異なります。基本的に、ARMモードでは`nn`はパイプラインによってもたらされるものです。

THUMBモードもハーフワードでアラインメントされている可能性があるという点では違いますが、同じARM形式でアドレスを保存することに注意してください。

## 例外からリターンするときに

例外ハンドラによって変更された可能性のある汎用レジスタ(R0-R14)を元に戻します。

以下のそれぞれの説明に記載されているように、リターン命令を使用して、PCとCPSRの両方をリストアします。 これはCPUのモード（THUMBまたはARM）とFIQ/IRQ無効化フラグの状態も含まれます。

上述したように（例外発生時の処理を参照）、リターンアドレスは常にARM形式で保存されるので、例外ハンドラは、例外がARMモードの内部で生じたか、THUMBモードの内部で生じたかに関わらず、同じリターン命令を使用することができます。

## FIQ (高速割り込み)

この割り込みはnFIQ入力のLOWレベルで発生します。これは、タイミングが重要な割り込みを優先して、できるだけ早く処理することを目的としています。

FIQモードでは、他のモードと違いR13とR14バンクレジスタ(R13_fiq,R14_fiq)に加えて、5つの余分なバンクレジスタ(R8_fiq-R12_fiq)が利用可能です。おかげで例外ハンドラは、メインプログラムのR8-R12レジスタを変更することなく(スタックに保存することなく)、これらのレジスタに自由にアクセスすることができます。

特権（非ユーザ）モードでは、CPSRのFビットをセットすることによって、FIQは手動で無効にすることもできます。

## IRQ(割り込み)

この割り込みはnIRQ入力のLOWレベルで発生します。FIQと違ってFIQモード用のR8-R12レジスタは用意されていません。

IRQはFIQより優先度が低く、FIQ割り込み中はIRQ割り込みは無効化されています。

特権（非ユーザ）モードでは、CPSRのIビットを設定することによって、IRQは手動で無効にすることもできます。

IRQモードから復帰する際には次の命令を実行します。 

```asm
  subs pc,lr,4   ; PC=R14_irq-4 して CPSR=SPSR_irq
```

## ソフトウェア割り込み

ソフトウェア割り込み命令(`SWI`)によって生成されます。例外ハンドラではBIOSの機能を

スーパバイザ（オペレーティングシステム）機能を要求することを推奨します。SWI命令には、オペコードのコメントフィールド部分にパラメータが含まれている場合もあります。

メインプログラムがTHUMBとARMの両方の状態の内側からSWIを発行する場合、例外ハンドラはARMの24bitのコメントフィールドとTHUMBの8bitのコメントフィールドを分離する必要があります（必要に応じてSPSR_svcのTビットを調べてもとのCPUモードを判断してください）。

ただし、リトルエンディアンモードでは、24bitのARMコメントフィールドのうち、最も重要な8bitのみを使用することができます（GBAなどではそう行われています）。

SVCモードから復帰するには次の命令を実行します。

`MOVS PC,R14   ;both PC=R14_svc, and CPSR=SPSR_svc`

注意: SWI命令はTHUMBモードで生じたとしても、常にARMモードで実行されます。

## 未定義命令割り込み

この例外は、CPU が処理できない命令に遭遇したときに発生します。

ほとんどの場合、プログラムがロックアップしたことを示し、エラーメッセージを表示すべきであることを示しています。

しかし、これはカスタム関数をエミュレートするためにも使用されるかもしれません。例えば、追加の 'SWI' 命令としてするなどです。(これは R14_und と SPSR_und を使用しますが、R14_svc と SPSR_svc を保存することなく、SVCモードの内部から未定義命令ハンドラを実行することができます)

UNDモードから復帰するには次の命令を実行します。

`MOVS PC,R14   ; both PC=R14_und, and CPSR=SPSR_und`

## アボート

アボート(ページフォールト)は大抵は仮想メモリでサポートされていますが、それ以外にもエラーメッセージを表示するためにも使われることがあります。

この場合はアボートには次の2つのパターンがあります。

- プリフェッチアボート(命令のプリフェッチ中に生じる)
- データアボート(データアクセス中に生じる)

仮想メモリシステムのアボートハンドラは、フォールトアドレスを決定します。プリフェッチアボートの場合は`R14_abt - 4`です。データアボートでは、メモリ内のアドレスを決定するために、`R14_abt - 8`のTHUMB命令またはARM命令を分解する必要があります。

復帰時にハンドラは次の命令を実行することで(アボートを起こした場所に)復帰します。

```
  prefetch abort: SUBS PC,R14,#4   ; PC = R14_abt - 4, and CPSR = SPSR_abt
  data abort:     SUBS PC,R14,#8   ; PC = R14_abt - 8, and CPSR = SPSR_abt
```

## Address Exceeds 26bit

この例外は、26bitのアドレススキームを持つ古いARM CPU（または26bitの下位互換モード）でのみ発生します。

## リセット

強制的に次のことをおこないます

```
PC = BASE
CPSR.T = 0            ; ARMステート
CPSR.F = 1            ; FIQ無効
CPSR.I = 1            ; IRQ無効
CPSR.M[4-0] = 0b10011 ; SVCモード 
```
