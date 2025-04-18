# カートリッジのプリフェッチ

カートリッジのプリフェッチは`WAITCNT.14`で有効にすることができます。

プリフェッチが有効になっている場合、GBAはCPUがバスを使用していない間、カートリッジROMからオペコードを読み出そうとします。

CPUが既にバッファにプリフェッチされているデータを要求した場合、メモリアクセスの際のウェイトステートは`0`になり1サイクルでアクセスできます。

プリフェッチバッファには、最大8つの16bit値が格納できます。

> [!NOTE]
> プリフェッチはカートリッジのオペコードに対してのみ有効です。RAMやBIOSに配置されたオペコードに対してはプリフェッチは行われません。

## 有効な場面

プリフェッチは次のようなオペコードで発生します。

```
  実行時にIサイクルが発生し、かつ R15を変更しない 命令
    shift/rotate register-by-register
    load opcodes (ldr,ldm,pop,swp)
    multiply opcodes
  ロードストア命令
    ldr,str,ldm,stm,etc.
```

## プリフェッチ無効バグ

When Prefetch is disabled, the Prefetch Disable Bug will occur for all

```
  "Opcodes in GamePak ROM with Internal Cycles which do not change R15"
```

for those opcodes, the bug changes the opcode fetch time from 1S to 1N.

Note: Affected opcodes (with I cycles) are: Shift/rotate register-by-register opcodes, multiply opcodes, and load opcodes (ldr,ldm,pop,swp).
