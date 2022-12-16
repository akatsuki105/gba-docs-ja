# 解凍

## BitUnPack: swi 0x10(GBA/NDS7/NDS9/DSi7/DSi9)

ビットマップやタイルデータの色深度を上げるために使用します。例えば、1bitのモノクロフォントを4bitや8bitのGBAタイルに変換する場合などに使用します。

`Unpack Info`をr2で指定することで、同じソースデータを異なるフォーマットに変換することができます。

```
引数:
  r0: ソースアドレス(アラインメントはしなくてもよい)
  r1: ターゲットアドレス(必ず4byte単位でアラインメント)
  r2: Unpack Infoのポインタ
    16bit: ソースデータの長さ(バイト単位, 0x-0xFFFF)
    8bit: ソースデータの単位(bit単位, 1, 2, 4, 8)
    8bit: ターゲットデータの単位(bit単位, 1, 2, 4, 8, 16, 32)
    32bit: Data Offset (Bit 0-30), Zero Data Flag (Bit 31)

返り値: なし。ターゲットアドレスにデータが書き込まれます。
```


`Data Offset`は解凍対象のデータが0でないときに、解凍対象のデータに加えられる値です。ただし、`Zero Data Flag`がセットされている場合、解凍対象のデータが0であっても`Data Offset`が加えられます。

解凍したデータは32bit単位でWRAMかVRAMのターゲットアドレスで指定したメモリに書き込まれます。

解凍データのサイズは必ず4バイトの倍数である必要があります。ソースデータの単位はターゲットデータの単位を超えてはいけません。

## Diff8bitUnFilterWrite8bit (Wram): swi 0x16 (GBA/NDS9/DSi9)
## Diff8bitUnFilterWrite16bit (Vram): swi 0x17 (GBA)
## Diff16bitUnFilter: swi 0x18 (GBA/NDS9/DSi9)

これらは実際の解凍機能ではなく、転送先データは転送元データと全く同じサイズになります。

しかし、ビットマップや波形が`10...19`のように増加する数字のストリームを含む場合、フィルタリング/フィルタリング解除されたデータは次のようになります。

```
  生データ:                    10  11  12  13  14  15  16  17  18  19
  フィルタリングされたデータ:     10  +1  +1  +1  +1  +1  +1  +1  +1  +1
```

この場合、フィルタリングされたデータを使えば（実際の圧縮アルゴリズムと組み合わせれば）、より良い圧縮結果が得られることは明らかです。

データ単位は8bit, 16bitで、それぞれ`Diff8bitXXX`、`Diff16bitXXX`関数で使用します。

```
引数:
  r0: ソースアドレス (must be aligned by 4) pointing to data as follows:
    データヘッダ (32bit)
      Bit 0-3:  データサイズ (Diff8bit関数なら1, Diff16bit関数なら2)
      Bit 4-7:  圧縮タイプ (必ず8)
      Bit 8-31: 元データのサイズ
    データユニット(Diff8bitXXXなら8bit, Diff16bitXXXなら16bit)
      Data0          ; オリジナルデータ
      Data1-Data0    ; データの差分
      Data2-Data1    ; ...
      Data3-Data2
      ...
  r1: ターゲットアドレス

返り値: なし。ターゲットアドレスにデータが書き込まれます。
```

## HuffUnCompReadNormal: swi 0x13(GBA)
## HuffUnCompReadByCallback: swi 0x13(NDS)

The decoder starts in root node, the separate bits in the bitstream specify if the next node is node0 or node1, if that node is a data node, then the data is stored in memory, and the decoder is reset to the root node. 

The most often used data should be as close to the root node as possible. For example, the 4-byte string "Huff" could be compressed to 6 bits: 10-11-0-0, with root.0 pointing directly to data "f", and root.1 pointing to a child node, whose nodes point to data "H" and data "u".

Data is written in units of 32bits, if the size of the compressed data is not a multiple of 4, please adjust it as much as possible by padding with 0.
Align the source address to a 4Byte boundary.

```
引数:
  r0: ソースアドレスで、次のデータ構造を指すポインタ
    データヘッダ (32bit)
      Bit 0-3:  Data size in bit units (normally 4 or 8)
      Bit 4-7:  圧縮タイプ (必ず2)
      Bit 8-31: 元データのサイズ
    ツリーサイズ (8bit)
      Bit0-7: Size of Tree Table/2-1 (ie. Offset to Compressed Bitstream)
    ツリーテーブル (list of 8bit nodes, starting with the root node)
      Root Node and Non-Data-Child Nodes are:
        Bit0-5:  Offset to next child node,
          Next child node0 is at (CurrentAddr AND NOT 1)+Offset*2+2
          Next child node1 is at (CurrentAddr AND NOT 1)+Offset*2+2+1
        Bit6:    Node1 End Flag (1=Next child node is data)
        Bit7:    Node0 End Flag (1=Next child node is data)
      Data nodes are (when End Flag was set in parent node):
        Bit0-7: Data (upper bits should be zero if Data Size is less than 8)
    圧縮bit列 (32bit単位)
        Bit0-31: Node Bits (Bit31=First Bit)  (0=Node0, 1=Node1)
  r1: ターゲットアドレス
  r2: Callback temp buffer      ; \for NDS/DSi "ReadByCallback" variants only
  r3: Callback structure        ; /(see Callback notes below)

返り値: なし。ターゲットアドレスにデータが書き込まれます。
```

## LZ77UnCompReadNormalWrite8bit (Wram): swi 0x11 (GBA/NDS7/NDS9/DSi7/DSi9)
## LZ77UnCompReadNormalWrite16bit (Vram): swi 0x12 (GBA)
## LZ77UnCompReadByCallbackWrite8bit: swi 0x01 (DSi7/DSi9)
## LZ77UnCompReadByCallbackWrite16bit: swi 0x12 (NDS), swi 0x02 or swi 0x19 (DSi)

LZ77圧縮されたデータを展開します。

WRAM版の方が高速で、8bit単位で書き込みを行います。

VRAM版では、展開先が2バイトでアラインメントされている必要があり、データは16bit単位で書き込まれます。

[LZ Decompression Functions](https://problemkaputt.de/gbatek.htm#lzdecompressionfunctions)

CAUTION: Writing 16bit units to [dest-1] instead of 8bit units to [dest] means that reading from [dest-1] won't work, ie. the "Vram" function works only with disp=001h..FFFh, but not with disp=000h.

圧縮データのサイズが4の倍数でない場合は、0でパディングするなどして可能な限り調整してください。 またソースアドレスを4バイト単位でアラインメントしておいてください。

```
引数:
  r0: ソースアドレスで、次のデータ構造を指すポインタ
    データヘッダ (32bit)
      Bit 0-3:  予約領域 (0)
      Bit 4-7:  圧縮タイプ (必ず1)
      Bit 8-31: 元データのサイズ
    以下のデータを繰り返す。各フラグバイトの後に8つのブロックが続く。
    フラグデータ (8bit)
      Bit 0-7:  直後の8ブロックのブロックタイプ MSB(Bit7(直後) -> Bit6 -> .. -> Bit0)
    ブロックタイプ0 - 生データ - そのままソースからターゲットにコピーされる
      Bit 0-7:  ターゲットアドレスにコピーされる1バイトデータ
    ブロックタイプ1 - 圧縮データ - Dest-Disp-1 から Dest へ N+3バイト をコピーする。
      Bit 0-3:  Disp MSBs
      Bit 4-7:  N(コピーするバイト数-3)
      Bit 8-15: Disp LSBs
  r1: ターゲットアドレス
  r2: コールバック引数             ;\for NDS/DSi "ReadByCallback" variants only
  r3: Callback structure        ;/(see Callback notes below)

返り値: なし。
```

解凍処理の例を疑似コードで表すと次のようになっています。

```go
width := 1 // or 2バイト
source, dest := r0, r1
remaining := Load24(source+1)

blockheader := 0 // Some compilers warn if this isn't set, even though it's trivially provably always set
source += 4
blocksRemaining := 0
halfword := 0

for remaining > 0 {
  if blocksRemaining != 0 {
    if BitI(blockheader, 7) {
      // 圧縮データ
      block := Load16(source)
      source += 2
      disp := dest - (block&0x0fff) - 1
      bytes := (block >> 12) + 3
      for bytes > 0 {
        bytes--

        if remaining > 0 {
          remaining--
        } else {
          panic("")
        }

        switch width {
          case 2:
            bytedata := Load16(disp & ^1)
            if dest&1 == 1 {
              bytedata >>= (disp & 1) * 8
              halfword |= bytedata << 8
              Store16(dest^1, halfword)
            } else {
              bytedata >>= (disp & 1) * 8
              halfword = bytedata & 0xff
            }
          default:
            Store8(dest, Load8(disp))
        }

        disp++
        dest++
      }
      
    } else {
      // 生データ
      bytedata := Load8(source)
      source++
      switch width {
        case 2:
          if dest&1 == 1 {
            halfword |= bytedata << 8
            Store16(dest^1, halfword)
          } else {
            halfword = bytedata
          }
        default:
          Store8(dest, bytedata)
      }
      dest++
      remaining--
    }
  }

  // フラグデータの読み取り
  blockheader = Load8(source)
  source++
  blocksRemaining = 8
}
```

## RLUnCompReadNormalWrite8bit (Wram): swi 0x14 (GBA/NDS)
## RLUnCompReadNormalWrite16bit (Vram): swi 0x15 (GBA)
## RLUnCompReadByCallbackWrite16bit: swi 0x15 (NDS)

ランレングス圧縮(RLE)されたデータを展開します。

WRAM版の方が高速で、8bit単位で書き込みを行います。

VRAM版では、解凍先アドレスは2バイト単位でアラインメントされている必要があり、データは16bit単位で書き込まれます。

圧縮データのサイズが4の倍数でない場合は、0でパディングして可能な限り調整してください。 またソースアドレスを4バイト単位でアラインメントしておいてください。

```
引数:
  r0: ソースアドレスで、次のデータ構造を指すポインタ:
    データヘッダ (32bit)
      Bit 0-3:  予約領域
      Bit 4-7:  圧縮タイプ (必ず3)
      Bit 8-31: 元データのサイズ
    Repeat below. Each Flag Byte followed by one or more Data Bytes.
    フラグデータ (8bit)
      Bit 0-6   Expanded Data Length (uncompressed N-1, compressed N-3)
      Bit 7     Flag (0=uncompressed, 1=compressed)
    データ本体 - N uncompressed bytes, or 1 byte repeated N times
  r1: ターゲットアドレス
  r2: コールバック引数                ; DSの swi 0x15のみ
  r3: Callback structure           ; DSの swi 0x15のみ

返り値: なし。ターゲットアドレスにデータが書き込まれます。
```
