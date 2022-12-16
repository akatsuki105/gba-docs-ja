# GPIO

ROMチップの中には4bitのGPIOが入っています。

[ボクらの太陽](https://ja.wikipedia.org/wiki/%E3%83%9C%E3%82%AF%E3%82%89%E3%81%AE%E5%A4%AA%E9%99%BD)ではRTCと太陽センサーにGPIOを使っています。

[まわるメイドインワリオ](https://www.nintendo.co.jp/n08/rzwj/index.html)では振動パックとジャイロセンサーにGPIOを使っています。

他のゲームでも、上記以外のセンサーやSRAMバンクの切り替えなど、他の目的で使用されるかもしれません。

GPIOのI/Oレジスタは(`0x04xx_xxxx`ではなく)ROMエリアの`0x0800_00C4`にある6バイトの領域にマッピングされており、この6バイトの領域はROMイメージではゼロ埋めされているはずです。ボクらの太陽では、このゼロ埋め領域のサイズは0E0hバイトとなっていますが、これは定義が間違っているためです。

この領域には追加のポートはありません。ミラー領域になっている場所もありません。

また、ROMバスへの書き込みは16bit/32bitのアクセスに限定されています。

## 0x0800_00C4 - I/Oポートデータ (W または R/W)

パーミッションは後述の[制御レジスタ](#0x0800_00c8---ioポート制御レジスタ-w-または-rw)で 書き込みのみ と、読み書き可能 のどちらかから選択可能です。

bit | 内容
-- | -- 
0 | データビット0
1 | データビット1
2 | データビット2
3 | データビット3
4-15 | 不使用(常に0)

## 0x0800_00C6 - I/Oポートディレクション (W または R/W)

bit | 内容
-- | -- 
0 | ポートディレクション0(0=In, 1=Out)
1 | ポートディレクション1
2 | ポートディレクション2
3 | ポートディレクション3
4-15 | 不使用(常に0)

## 0x0800_00C8 - I/Oポート制御レジスタ (W または R/W)

上のI/Oポートデータ、I/Oポートディレクション(、そしてこのレジスタ自身)のパーミッションを制御します。

bit | 内容
-- | -- 
0 | パーミッション(0=書き込みのみ, 1=読み書き可能)
1-15 | 不使用(常に0)

## 接続の例

```
  GPIO       | Boktai  | Wario
  Bit Pin    | RTC SOL | GYR RBL
  -----------+---------+---------
  0   ROM.1  | SCK CLK | RES -
  1   ROM.2  | SIO RST | CLK -
  2   ROM.21 | CS  -   | DTA -
  3   ROM.22 | -   FLG | -   MOT
  -----------+---------+---------
  IRQ ROM.43 | IRQ -   | -   -
```

Aside from the I/O Port, the ROM-chip also includes an inverter which is used for inverting the RTC /IRQ signal, and some sort of an (unused) address decoder output which appears to be equal or related to A23 signal. That is, reacting on ROM A23, or SRAM D7, which share the same pin on GBA slot.

