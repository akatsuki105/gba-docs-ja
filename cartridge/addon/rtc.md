# RTC(Real-Time Clock)

RTCは、ポケモンシリーズやボクらの太陽シリーズで使用されています。

## 概要

GBAのRTCには、ボクらの太陽の場合、S3511という、3線式のシリアルバスをもった8ピンのRTCが使われています。

このRTCチップはほとんど[NDSで使われているもの](https://github.com/akatsuki105/nds-docs-ja/blob/main/system/rtc.md)と同じように使用されます。

このRTCチップは4bit幅のIOポート(GPIO)を通じてアクセスします。RTCで使うのは3bitだけです。

## NDSのRTCレジスタとの比較

```
  NDS_________GBA_________GBA/Params___
  stat2       control     (1-byte)
  datetime    datetime    (7-byte)
  time        time        (3-byte)
  stat1       force reset (0-byte)
  clkadjust   force irq   (0-byte)
  alarm1/int1 always FFh  (boktai contains code for writing 1-byte to it)
  alarm2      always FFh  (unused)
  free        always FFh  (unused)
```

## 制御レジスタ

bit | R/W | 内容
-- | -- | --  
0 | -   | 不使用
1 | R/W | IRQ duty/hold related?
2 | -   | 不使用
3 | R/W | Per Minute IRQ (30s duty)        (0=Disable, 1=Enable)
4 | -   | 不使用
5 | R/W | 用途不明
6 | R/W | 12/24-hour Mode                  (0=12h, 1=24h) (usually 1)
7 | R   | Power-Off (auto cleared on read) (0=Normal, 1=Failure)

このレジスタは、`Battery-Shortcut`の後では`$82(0b1000_0010)`に、`Force-Reset`の後では`$00`がセットされます。

不使用のビットは常に0のように見えますが、読み取り可能(R)か書き込み可能(W)のどちらかであるようです。

## Dateレジスタ

NDSのそれと全く同じです。

```
  Year Register
    0-7 R/W Year     (BCD 00h..99h = 2000..2099)
  Month Register
    0-4 R/W Month    (BCD 01h..12h = January..December)
    5-7 -   Not used (always zero)
  Day Register
    0-5 R/W Day      (BCD 01h..28h,29h,30h,31h, range depending on month/year)
    6-7 -   Not used (always zero)
  Day of Week Register (septenary counter)
    0-2 R/W Day of Week (00h..06h, custom assignment, usually 0=Monday?)
    3-7 -   Not used (always zero)
```

## Timeレジスタ

AM/PMフラグのビットの位置が違うことを除けばNDSのそれと同じです。

- NDSのAM/PMフラグ: hour.bit6
- GBAのAM/PMフラグ: hour.bit7

```
  Hour Register
    0-5 R/W Hour     (BCD 00h..23h in 24h mode, or 00h..11h in 12h mode)
    6   -   Not used (always zero)
    7   *   AM/PM    (0=AM before noon, 1=PM after noon)
            * 24h mode: AM/PM flag is read only (PM=1 if hour = 12h..23h)
            * 12h mode: AM/PM flag is read/write-able
            * 12h mode: Observe that 12 o'clock is defined as 00h (not 12h)
  Minute Register
    0-6 R/W Minute   (BCD 00h..59h)
    7   -   Not used (always zero)
  Second Register
    0-6 R/W Minute   (BCD 00h..59h)
    7   -   Not used (always zero)
```

## Force Reset/Irq Registers

Used to reset all RTC registers (all used registers become 00h, except
day/month which become 01h), or to drag the IRQ output LOW for a short moment.
These registers are strobed by ANY access to them, ie. by both writing to, as
well as reading from these registers.

## Pin-Outs / IRQ Signal

パッケージのピン配置はNDSと同じですが、チップがDSのそれよりも若干大きくなっています。

For whatever reason, the RTC's /IRQ output is passed through an inverter
(contained in the ROM-chip), the inverted signal is then passed to the /IRQ pin
on the cartridge slot. So, IRQ's will be triggered on the "wrong" edge -
possible somehow in relation with detecting cartridge-removal IRQs?
