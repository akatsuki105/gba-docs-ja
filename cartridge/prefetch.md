# [カートリッジのプリフェッチ](https://problemkaputt.de/gbatek.htm#gbagamepakprefetch)

カートリッジのプリフェッチは [WAITCNT](../system.md#0x0400_0204---waitcnt---waitstate制御レジスタ-rw)のbit14で有効にすることができます。

プリフェッチバッファが有効になっている場合、GBAはCPUがバスを使用していないタイミングにカートリッジROMから（もしあれば）オペコードを読み出そうとします。

CPUが既にバッファにプリフェッチされているデータを要求した場合、メモリアクセスの際のWaitStateは0になり1サイクルでアクセスできます。

プリフェッチバッファには、最大8つの16bit値が格納できます。

## カートリッジROMのオペコード

プリフェッチ機能はカートリッジROMのオペコードに対してのみ有効です。

RAMやBIOSに配置されたオペコードに対してはプリフェッチ機能を使うことはできません。

## 有効な場面

プリフェッチは次のようなオペコードで発生します。

- Iサイクルで実行されるオペコードでR15を変更しないもの (shift/rotate register-by-register, load opcodes (ldr,ldm,pop,swp), multiply opcodes)
- ロードストア命令 (ldr,str,ldm,stm,etc.)

## プリフェッチ無効バグ

プリフェッチを無効にすると、R15を変更しない内部サイクルを持つカートリッジ内のオペコードに対してプリフェッチ無効バグが発生し、オペコードのフェッチ時間が1Sから1Nに変更されます。

影響を受けるオペコード: Shift/rotate register-by-register opcodes, multiply opcodes, and load opcodes (ldr,ldm,pop,swp).
