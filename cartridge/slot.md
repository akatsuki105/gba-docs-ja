# スロット

## ピン配置

カートリッジバスは、GB/GBAカートリッジの両方で使用可能です。GBAモードでは、次のように使用されます：

```
  Pin     Name       Dir   Expl.
  1       VDD        O     電源(3.3V)
  2       PHI        O     カートリッジの動作クロック (後述)
  3       /WR        O     Write Select  ;\latched address to be incremented on
  4       /RD        O     Read Select   ;/rising edges of /RD or /WR signals
  5       /CS        O     ROMチップセレクト
  6-21    AD0..AD15  I/O   アドレスバス/ROMデータバス共用
  22-29   A16..A23   I/O   アドレスバス/SRAMデータバス共用
  30      /CS2       O     SRAMチップセレクト
  31      /REQ       I     /IREQ or /DREQ
  32      GND        O     Ground(0V)
```

バス(6~29ピン)の内容は、ROMアクセスとSRAMアクセスで次のようになっています。
- ROMアクセス:  `AD0..AD15` と`A16..A23` を通じて24ビットのアドレスが出力され、その後 `AD0..AD15` を通じて16ビットのデータが転送されます。
- SRAMアクセス: `AD0..AD15` を通じて16bitのアドレスが出力され、その後 `A16..A23` を通じて8ビットのデータが転送されます。

ROMアクセスの場合は2バイト単位のアクセスとなります。(`A0..A23` に2を掛けたのが実際のアドレス) よって最大で32MB(`(1 << 25)`)までアドレス指定できます。SRAMアクセスの場合はバイト単位の指定になるため最大で64KB(`(1 << 16)`)までアドレス指定できます。

GBA本体のスロット内部には小さなスイッチがあり、GB/GBCカートリッジ(8ビットカートリッジ)を挿入するとこのスイッチは押し下げられますが、GBAカートリッジを挿入した時や何も挿さっていない場合はスイッチは元に戻ります。
このスイッチで電源電圧を 5V と 3.3V のどちらにするか切り替えています。つまり、GBAではカートリッジスロットと通信ポートに3.3V、GB/GBCでは5Vが使用されます。

The switch additionally drags IN35 to 3V when an 8bit cart is inserted, the current state of IN35 can be determined in GBA mode via Port 4000204h (WAITCNT), if the switch is pushed, then CGB mode can be activated via Port 4000000h (DISPCNT.3), this bit can be set ONLY by opcodes in BIOS region (eg. via CpuSet SWI function).

In 8bit mode, the cartridge bus works much like for GBA SRAM, however, the 8bit /CS signal is expected at Pin 5, while GBA SRAM /CS2 at Pin 30 is interpreted as /RESET signal by the 8bit MBC chip (if any). In practice, this appears to result in 00h being received as data when attempting to read-out 8bit cartridges from inside of GBA mode.
