# 仕様

## 概要

- CPU: 32bit CPU, クロックは16.78MHz
- RAM: 288KB
- グラフィック: 240x160pxで最大256色
- サウンド: 6チャンネル, 16bit音源

## 本体

- 発売日: 2001/3/21
- 定価: 9800円(税別)
- 大きさ(幅x奥行きx高さ): 144×82×24.5mm
- 重量: 約140g

## CPU

モード | 概要
---- | ----
ARMモード |   ARM7TDMI 32bit RISC CPU, 16.78MHz, 32bit opcodes
THUMBモード |   ARM7TDMI 32bit RISC CPU, 16.78MHz, 16bit opcodes
CGBモード  |   GBZ80 8bit CPU, 4.2MHz or 8.4MHz
DMGモード  |   GBZ80 8bit CPU, 4.2MHz

## 内部メモリ

名称 | 容量 | 備考
---- | ---- | ----
BIOS ROM    | 16KB | 
Work RAM    | 288KB | チップ上の32KBは高速で、オンボードの256KBは低速
VRAM        | 96KB |
OAM         | 1KB | 128 OBJs 3x16bit, 32 OBJ-Rotation/Scalings 4x16bit
Palette RAM | 1KB | BG:256色, OBJ:256色

## 画面表示

名称 | 概要
---- | ----
Display     | 240x160px (2.9インチ TFT液晶)
BG layers   | 4つの背景レイヤ
BG types    | タイルモードとビットマップモード
BG colors   | 256色 or 16色/16パレット or 32768色
OBJ colors  | 256色 or 16色/16パレット
OBJ size    | 12種類 (ピクセル単位で 8x8 から 64x64まで)
OBJs/Screen | max. 128 OBJs of any size (up to 64x64 dots each)
OBJs/Line   | max. 128 OBJs of 8x8 dots size (under best circumstances)
Priorities  | OBJ/OBJ: 0-127, OBJ/BG: 0-3, BG/BG: 0-3
Effects     | 画面に適用できるエフェクト。伸縮回転, ブレンド, モザイク, ウィンドウ
Backlight   | GBASPのみ搭載

## サウンド

名称 | 概要
---- | ----
アナログ | CGB互換のための4チャンネル (3x square wave, 1x noise)
デジタル  | DMAサウンドチャンネルが2つ
出力   | スピーカー(モノラル) または ヘッドホン端子(ステレオ)

## 操作

上下左右4つの方向キーと、A,B,Start,Select,L,Rの6つのボタン

## 通信機能
  
Serial Port  Various transfer modes, 4-Player Link, Single Game Pak play

## 外部メモリ

- GBA Game Pak max. 32MB ROM or flash ROM + max 64K SRAM
- CGB Game Pak max. 32KB ROM + 8KB SRAM (more memory requires banking)

## サイズ (mm)   

- GBA: 145x81x25
- GBASP: 82x82x24 (閉), 155x82x24 (開)

## 電源

- Battery GBA  GBA: 2x1.5V DC (AA), Life-time approx. 15 hours
- Battery SP   GBA SP: Built-in rechargeable Lithium ion battery, 3.7V 600mAh
- External     GBA: 3.3V DC 350mA - GBA SP: 5.2V DC 320mA
