# バックアップメディア

セーブ機能のことです。

## バックアップID文字列

バックアップメディアの種類を検出するためのIDです。

GBAはROMヘッダにバックアップタイプのエントリを入れていません。しかしバックアップタイプはROM内にある後述のID文字列から検出できます。

任天堂のツールでは、これらの文字列を（ライブラリ・ヘッダの一部として）自動的に挿入しています。

**ID文字列**

ID文字列を配置するアドレスは4バイトアラインメントされている必要があります。また文字列の長さは4バイトの倍数でなければなりません。（余った分はゼロでパディングされます）

ID文字列 | 内容
-- | --
`EEPROM_Vnnn` | EEPROM 512B or 8KB
`SRAM_Vnnn` | SRAM 32KB or FRAM 32KB
`FLASH_Vnnn` or `FLASH512_Vnnn` | FLASH 64KB (後者は新しめのゲーム)
`FLASH1M_Vnnn` | FLASH 128KB

任天堂のツールでID文字列を挿入する場合、`nnn`は3桁のライブラリバージョン番号です。サードパーティツールで挿入する場合は、数字を入れずに`nnn`のままにしておくとよいでしょう。

## 🔋 SRAM/FRAM

>**Note**  
FRAM は `Ferroelectric RAM` の略で、FLASHとは別物です。

[こちら](./sram.md)を参照してください。

## ⚡️ EEPROM

[こちら](./eeprom.md)を参照してください。

## 📸 FLASH

[こちら](./flash.md)を参照してください。

## 🧰 DACS

```
  1Mbit DACS: 128 KB
    セーブ機能の寿命: 100,000 writes.
  8Mbit DACS: 1024 KB
    セーブ機能の寿命: 100,000 writes.
```

DACS (Debugging And Communication System) is used in Nintendo's hardware debugger only, DACS is NOT used in normal game cartridges.

Parts of DACS memory is used to store the debugging exception handlers (entry point/size defined in cartridge header), the remaining memory could be used to store game positions or other data. The address space is the upper end of the 32MB ROM area, the memory can be read directly by the CPU, including for ability to execute program code in this area.
