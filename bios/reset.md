# リセット

SoftReset, RegisterRamReset, HardReset

## SoftReset: swi 0x00

スタック、BIOSのIRQベクタやフラグなどRAM領域を200バイトクリアし、システムスタック、SVCスタック、IRQスタックのポインタを初期化します。最後に、R0-R12, LR_svc, SPSR_svc, LR_irq, SPSR_irqを0クリアしてシステムモードに切り替わります。

NDS9のスタックレジスタSPはハードコードされていることに注意してください。NDS9では`SoftReset`はさらに、キャッシュとライトバッファをクリアし、CP15コントロールレジスタを`0x12078`に設定します。

Host | sp_svc | sp_irq | sp_sys | zerofilled area | return address
-- | -- | -- | -- | -- | --  
GBA  | 0x3007FE0 | 0x3007FA0 | 0x3007F00 | [0x3007E00..0x3007FFF] | Flag[0x3007FFA]
NDS7 | 0x380FFDC | 0x380FFB0 | 0x380FF00 | [0x380FE00..0x380FFFF] | Addr[0x27FFE34]
NDS9 | 0x0803FC0 | 0x0803FA0 | 0x0803EC0 | [DTCM+0x3E00..0x3FFF]  | Addr[0x27FFE24]

(NDSのみ)リセット処理はSWIを実行したCPUでのみ行われることに注意してください。

```
引数、返り値: なし
```

`SoftReset`からリターンするときは呼び出した関数には戻らず、上記のリターンアドレスをR14にロードし、`BX R14`オペコードでそのアドレスにジャンプします。

## RegisterRamReset: swi 0x01

ResetFlagsで指定されたI/OレジスタやRAMをリセットします。ただし、`0x3007E00-0x3007FFF`のCPU内蔵RAM領域はクリアされません。

```
引数: r0(ResetFlags)
    Bit   Expl.
    0     Clear 256K on-board WRAM  ; -don't use when returning to WRAM
    1     Clear 32K on-chip WRAM    ; -excluding last 200h bytes
    2     Clear Palette
    3     Clear VRAM
    4     Clear OAM                 ; -zerofilled! does NOT disable OBJs!
    5     Reset SIO registers       ; -switches to general purpose mode!
    6     Reset Sound registers
    7     Reset all other registers (except SIO, Sound)

返り値: なし
```

R0のBit5をクリアしてもSIODATA32のLSBが常にクリアされてしまうバグがあります。

`DISPCNT=0x0080`にすると、常に強制的に画面を白に切り替える機能があります（R0の入力に関わらず、画面が白になります）。

## HardReset: swi 0x26(GBA, Undocumented)

この機能は、GBAを再起動します（時間のかかるnintendoの起動画面を無視するための機能も含まれていますが、この機能は特に役に立たず、煩わしいものとなっています）。

```
引数、返り値: なし
```

実行時間: 2秒

