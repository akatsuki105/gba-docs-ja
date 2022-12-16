# Halt

## Halt: swi 0x02(GBA) & swi 0x06(NDS)

割り込みリクエストが発生するまでCPUを停止します。CPUは低消費電力モード(Halt状態)に切り替わり、その他の回路（ビデオ、サウンド、タイマー、シリアル、キーパッド、システムクロック）は動作し続けます。

Halt状態は，有効な割り込みが要求されると終了します。つまり，`IE & IF`が0でない場合です。その条件が成立しないとGBAはHaltし続けます。

ただし，CPUのCPSRレジスタのIRQ無効化ビットとIMEレジスタの状態は関係なく，どちらかが割り込みを無効にしていてもHaltは終了します。

GBAおよびNDS7/DSi7では、`0x4000301`の`HALTCNT`に書き込むことでHalt状態に入ります。

NDS9/DSi9では、システム制御コプロセッサへの書き込み(mov p15,0,c7,c0,4,r0 opcode)でHalt状態に入りますが、このopcodeはIME=0の場合にハングアップします。

```
引数、返り値: なし
```

GBA/NDS7/DSi7ではレジスタは全て不変、NDS9/DSi9のみR0が変更されます

## IntrWait: swi 0x04

DSi7/DSi9、ではバグがある模様？

引数で指定した割り込みが起きるまでHaltし続けます。

このシステムコール関数は強制的に、IMEを1にセットします。

複数の割り込みを同時に使用する場合、この関数はHalt関数を繰り返し呼び出すよりもオーバーヘッドが少なくなります。

```
引数:
  r0
    0: 古いフラグでセットされていた場合、すぐにリターンする
    1: 古いフラグは無視して新たにフラグがセットされるまで待つ
  r1: 対象の割り込みフラグ(IE/IFと同じフォーマット)
  r2: 追加のフラグ(DSi7のみ)
```

Caution: When using IntrWait or VBlankIntrWait, the user interrupt handler MUST update the BIOS Interrupt Flags value in RAM; when acknowleding processed interrupt(s) by writing a value to the IF register, the same value should be also ORed to the BIOS Interrupt Flags value, at following memory location:

```
  Host     GBA (16bit)  NDS7 (32bit)  NDS9 (32bit)  DSi7-IF2 (32bit)
  Address  [3007FF8h]   [380FFF8h]    [DTCM+3FF8h]  [380FFC0h]
```

NDS9: BUG: No Discard (r0=0) doesn't work. The function always waits for at least one IRQ to occur (no matter which, including IRQs that are not selected in r1), even if the desired flag was already set. NB. the same bug is also found in the GBA/NDS7 functions, but it's compensated by a second bug, ie. the GBA/NDS7 functions are working okay because their "bug doesn't work".
Return: No return value, the selected flag(s) are automatically reset in BIOS Interrupt Flags value in RAM upon return.

DSi9: BUG: The function tries to enter Halt state via Port 4000301h (which would be okay on ARM7, but it's probably ignored on ARM9, which should normally use CP15 to enter Halt state; if Port 4000301h is really ignored, then the function will "successfully" wait for interrupts, but without actually entering any kind of low power mode).

DSi7: BUG: The function tries to wait for IF and IF2 interrupts, but it does accidently ignore the old IF interrupts, and works only with new IF2 ones.

## VBlankIntrWait: swi 0x05

次のVBlank割り込みが起きるまでHaltし続けます。

内部では`r0=1`, `r1=1`, `r2=0(DSi7 only)`をして`IntrWait`を実行しているだけなので、詳細は`IntrWait`を参照してください。

```
引数、返り値: なし
```

## Stop: swi 0x03(GBA)

GBAを超低電力モード(Stop状態)に移行します。CPU、システムクロック、サウンド、ビデオ、DMAやタイマーなども停止します。

Stop state can be terminated by the following interrupts only (as far as enabled in IE register): Joypad, Game Pak, or General-Purpose-SIO.

Stop状態は、次の割り込みが起きた時に終了します。Joypad, Game Pak, General-Purpose-SIO

システムクロックも停止しているので、IFフラグがセットされないことに注意してください。

Stop状態に入る前に:

Stop状態に入る前に画面表示を無効にしましょう。そうしないとStop状態に入った時に画面はフリーズし、電力を消費し続けます。

サウンドも無効にしておくのが望ましいです。当然ですが、外部のハードウェアも無効にできるならそうした方が望ましいです。

```
引数、返り値: なし
```

## Sleep: swi 0x07(NDS)

No info, probably similar as GBA SWI 03h (Stop). 

Sleep is implemented for ARM7 only, not for ARM9. 

But maybe the ARM7 function does stop **both** ARM7 and ARM9 (?)
