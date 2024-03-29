# interwork

GBAのソフトを解析していると次のようにして関数からリターンしている場合が多いです。

```
pop { r1 }
bx r1
```

ですが、

```
pop { pc }
```

と書く方が素直に思えます。

以後、前者の書き方を `popbx`, 後者の書き方を `poppc` と呼ぶことにします。

実行速度も、

```
; 合計: 3S+2N+1
pop { r1 } ; 1S+1N+1
bx r1      ; 2S+1N
```

```
pop { pc } ; 2S+2N+1
```

なので、実行時間は `1S` だけ`poppc`のほうが速いです。

スタックの使い方によっては、もっと差が出る場合もあります。

例えば、

```
; 合計: 4S+3N+2
pop { r4 }  ; 1S+1N+1
pop { r1 }  ; 1S+1N+1
bx r1       ; 2S+1N
```

```
pop { r4, pc } ; 3S+2N+1
```

の場合、実行時間は `1S+1N+1` だけ`poppc`のほうが速くなります。

一般的なROMでは `(N, S) = (4, 2)` なので、最初の例では 2サイクル、2番目の例では、 7サイクル の差がつくことになります。

## `popbx` と `poppc` の違い

両方とも、スタックからアドレスを取り出してそこにジャンプする処理ですが、GBAの場合は挙動が異なります。

`popbx`では、ジャンプ先のアドレスを偶数にすることでジャンプ先で`ARM`モードに、奇数にすることでジャンプ先で`THUMB`モードにスイッチします。

`poppc`の場合は、ジャンプしても`ARM/THUMB`モードが切り替わることはありません。

つまり、`poppc`で正常にリターンできるのは、呼び出し元とサブルーチンのCPUモードが同じ場合です。そのため、呼び出し元とサブルーチンのCPUモードが同じであることを保証できない場合は、`popbx`でリターンする必要があります。

`popbx`のような、`ARM/THUMB`モードのスイッチをケアするプログラムは `interwork` と呼ばれています。

## agbcc

`agbcc` は NintendoがC言語で書かれたソースコードをGBA ROM向けにビルドするときに使っていたコンパイラです。`GCC 2.95.1`が元になっています。<sup>[1](#agbcc)</sup>

`agbcc`(というか元になったGCC ARM)では、コンパイルの際に、`-mthumb-interwork`オプションを指定することで、リターン時のコードとして、`poppc`ではなく、`popbx`を使ったプログラムを吐き出すようになります。

## 使い所

冒頭でも述べましたが、`interwork`しない(`poppc`)ほうが、実行速度も速く、プログラムのサイズも小さくなります。

そのため、呼び出し元とサブルーチンのCPUモードが同じであることを保証できる場合は、`interwork`しない方が望ましいです。

実際のところ、(筆者の観測範囲で)多くのゲームは、とりあえず`-mthumb-interwork`オプションを指定して`interwork`したプログラムを吐き出させていました。

ただし、最適化に力を入れているゲーム(例: 黄金の太陽 失われし時代)ではなるべく`poppc`でリターンするようにしていたようです。<sup>[2](#gstla)</sup>

## 注釈

<sup id="agbcc">1: 他にも`GCC 2.95.0`を元にした初期版(`old agbcc`と呼ばれる) や `GCC 2.95.3`を元にした後期版(`modern agbcc`,`new agbcc`と呼ばれる)がある模様 </sup>

<sup id="gstla">2: https://discord.com/channels/768759024270704641/833314924766953482/931632076992684093 </sup>

