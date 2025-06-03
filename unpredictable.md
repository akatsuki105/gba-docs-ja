# 特殊な挙動

[GBA Unpredictable Things](https://problemkaputt.de/gbatek.htm#gbaunpredictablethings)より

## BIOS領域からの読み込み(0x0000_0000..0000_3FFF)

BIOSは読み出し処理に対して保護されています。

GBAは、プログラムカウンタがBIOSエリア内にある場合に限り、オペコードやデータを読み出すことができます。

プログラムカウンタがBIOSエリアにない場合に読み出しを行うと、正常にフェッチされた最新のBIOSオペコードが返ってきます。

例えば、

- 電源ON直後やソフトリセット直後は`[00DCh+8]`のオペコード
- IRQハンドラの処理中は`[0134h+8]`のオペコード
- IRQ終了後は`[013Ch+8]`のオペコード
- SWIの処理中は`[0188h+8]`のオペコード

が帰ってきます。

## 不使用メモリ領域からの読み込み (0x0000_4000..01FF_FFFF, 0x1000_0000..FFFF_FFFF)

Accessing unused memory at 00004000h-01FFFFFFh, and 10000000h-FFFFFFFFh (and 02000000h-03FFFFFFh when RAM is disabled via Port 4000800h) returns the recently pre-fetched opcode. 

For ARM code this is simply:

```
  WORD = [$+8]
```

For THUMB code the result consists of two 16bit fragments and depends on the address area and alignment where the opcode was stored.

For THUMB code in Main RAM, Palette Memory, VRAM, and Cartridge ROM this is:

```
  LSW = [$+4], MSW = [$+4]
```

For THUMB code in BIOS or OAM (and in 32K-WRAM on Original-NDS (in GBA mode)):

```
  LSW = [$+4], MSW = [$+6]   ; for opcodes at 4-byte aligned locations
  LSW = [$+2], MSW = [$+4]   ; for opcodes at non-4-byte aligned locations
```

For THUMB code in 32K-WRAM on GBA, GBA SP, GBA Micro, NDS-Lite (but not NDS):

```
  LSW = [$+4], MSW = OldHI   ; for opcodes at 4-byte aligned locations
  LSW = OldLO, MSW = [$+4]   ; for opcodes at non-4-byte aligned locations
```

Whereas OldLO/OldHI are usually:

```
  OldLO=[$+2], OldHI=[$+2]
```

Unless the previous opcode's prefetch was overwritten; that can happen if the previous opcode was itself an LDR opcode, ie. if it was itself reading data:

```
  OldLO=LSW(data), OldHI=MSW(data)
```

Theoretically, this might also change if a DMA transfer occurs.

Note: Additionally, as usually, the 32bit data value will be rotated if the data address wasn't 4-byte aligned, and the upper bits of the 32bit value will be masked in case of LDRB/LDRH reads.

Note: The opcode prefetch is caused by the prefetch pipeline in the CPU itself, not by the external gamepak prefetch, ie. it works for code in ROM and RAM as well.

## 8bitデータをビデオメモリに書き込んだときの挙動

Video Memory (BG, OBJ, OAM, Palette) can be written to in 16bit and 32bit units only. Attempts to write 8bit data (by STRB opcode) won't work: 

Writes to OBJ (6010000h-6017FFFh) (or 6014000h-6017FFFh in Bitmap mode) and to OAM (7000000h-70003FFh) are ignored, the memory content remains unchanged.

Writes to BG (6000000h-600FFFFh) (or 6000000h-6013FFFh in Bitmap mode) and to Palette (5000000h-50003FFh) are writing the new 8bit value to BOTH upper and lower 8bits of the addressed halfword, ie. "[addr AND NOT 1]=data*101h".

## Using Invalid Tile Numbers

In Text mode, large tile numbers (combined with a non-zero character base setting in BGnCNT register) may exceed the available 64K of BG VRAM.

On GBA and GBA SP, such invalid tiles are displayed as if the character data is filled by the 16bit BG Map entry value (ie. as vertically striped tiles). Above applies only if there is only one BG layer enabled, with two or more layers, things are getting much more complicated: tile-data is then somehow derived from the other layers, depending on their priority order and scrolling offsets.

On NDS (in GBA mode), such invalid tiles are displayed as if the character data is zero-filled (ie. as invisible/transparent tiles).

## SRAM領域への16/32bitのアクセス

**Read**

アドレスから8bitの値を取得しますが、その値には `0x0101`(LDRH),`0x01010101`(LDR) が掛け算されています。

**Write**

対象のアドレスに対してのみ(つまり1バイト分)書き込みを行います。

書き込みたい値を`val`(16 or 32bit)、対象のアドレスを4で割った余りを`x`とすると`val ROR (x*8)`が対象のアドレスに書き込まれます。

