# デコード

<pre>
Note:

この記事は

- <a href="http://emucode.blogspot.com/2010/09/decoding-arm-instruction-set.html">Decoding the ARM instruction set</a>
- <a href="http://emucode.blogspot.com/2010/08/gba-emulator.html">Decoding the THUMB instruction set</a>

を翻訳したもので、主にGBAエミュレータ開発者向けです。
</pre>

## ARMモード

まず、ARM命令のバイナリオペコードフォーマットを見てみましょう。

![arm_bit_pattern](/images/decode/arm_bit_pattern.png)

しかし、この参考文献は完全に正確なものではなく、最初の命令をデコードするために必要な重要な情報を見逃しているので、これから説明します。

まず最初に、THUMB命令セットをデコードするとき、私たちは単純に命令の上位8ビットを使用していたので、命令を明確にデコードすることができました。

例えば、ビット27～20を使って、どの命令なのかを判断しようとすると、ビット27～20をすべてアンセット（0）にする命令が複数種類あるため、曖昧になってしまいます。

そこで、上位8ビット（ビット27-20）とベース4ビット（ビット7-4）を端と端で足し合わせることにします。この原理をもう少しわかりやすく説明するための絵があります。

![arm_bit_pattern_1](/images/decode/arm_bit_pattern_1.png)

C/C++では、この値を実際に取得するには、ビット単位での微調整が必要です。例えば、以下のようになります。

```cpp
u32 instruction = fetch();
u16 _12Bits = (((instruction >> 16) & 0xFF0) | ((instruction >> 4) & 0x0F));
```

これで、命令をデコードするのに使う12ビットの数字ができました。では、最初から考えてみましょう。

この12ビットの数字が0、つまりすべてのビットがアンセットされていたらどうなるでしょうか。

オペコードフォーマットを見ると、これを可能にする命令フォーマットは1つだけで、1枚目の写真の一番上の`Data Processing/PSR`命令であることがわかります。

さて、この命令がどのようなタイプの命令であるかはすでにわかっていますが、どのような命令であるかを正確に知るためには、さらに情報が必要です。

`Data Processing/PSR`命令では、`Operand2`というフィールドがあります。

このフィールドは、上記のバイナリオペコードフォーマットには示されていないフォーマットを持っており、以下に`Operand2`フィールドの仕様を示します。

![arm_bit_pattern_2](/images/decode/arm_bit_pattern_2.png)

今回の状況では、すべてのビットは0なので、当然`immediate`ビット（bit25）がセットされていないので、シフトされたレジスタを使用していることになります。

バレルシフトについては今のところ説明しませんが、これはこの記事の範囲を超えています。ここでは、より正式な言い方として、マニュアルから直接引用します。

> 第2オペランドがシフトレジスタと指定されている場合、バレルシフタの動作は命令の`shift`フィールドによって制御されます。

`shift`フィールドは次のようになっています。

![arm_bit_pattern_3](/images/decode/arm_bit_pattern_3.png)

これは非常に複雑なことだと思いますが、マニュアルを読んでいただくのが一番です。

とにかく、私についてきてください。この命令について、これまでに集めた情報を確認してみましょう。12ビットすべてが0であると仮定すると...。

1. `Data Processing`命令です。
2. シフトされたレジスタのバレルシフト演算を使用します。
3. bit24-21からAND命令です。
4. bit4は0なので、シフトしたレジスタを即値でシフトするシフト操作であることがわかり、直前の図の左側のフォーマットを使用しています。
5. bit6-5は0なので、シフト内容は`LSL`です。

よって

`AND Rd, Rn, Rm, LSL #imm5`

となります！

## THUMBモード

まず、THUMB命令のバイナリオペコードフォーマットを見てみましょう。

![thumb_bit_pattern](/images/decode/thumb_bit_pattern.png)

各命令は2バイト（ハーフワード）で構成されており、各命令には、CPUがデコードする独自のフォーマットがあります。

ご覧のように、フォーマット1(`Move shifted register`)のようないくつかの命令フォーマットには、これが特定の命令であることを示すオペコードフィールドがあります。`Move shifted register`の場合には、12-11bitに2ビットのオペコードフィールドがあります。

さて、私が命令をデコードする方法は、命令の上位8ビットを取り、その値をチェックすることです。

例えば、上位8ビットの値が「0〜7」であれば、その命令が`LSL Rd, Rs, Offset`とわかります。なぜでしょうか？

これは、上位8ビットが「0〜7」になっている間は、`Move shifted register`のオペコードフィールドが0になり、この形式の他のビットはすべて正しいからです。

![thumb_bit_pattern_1](/images/decode/thumb_bit_pattern_1.png)

上の図では上位8bitは`00000101`となっています。

命令フォーマットを確認すると、`Move shifted register`フォーマットでは、上位8bitがこの値になる場合がありえます。

`Move shifted register`フォーマットでは、先の8bit値の下位3bitが、`Offset5`フィールドの上位3bit（bit8,9,10）であり、このフィールドは任意の値がありうるからです。

また、任意の値をとりうるということは、次の8つのバイナリはすべて`LSL Rd, Rs, Offset5`となります。

```
00000000    (0)
00000001    (1)
00000010    (2)

00000011    (3)
00000100    (4)
00000101    (5)
00000110    (6)
00000111    (7)
```

次に、仮に8という値が出た場合、オペコードフィールドが1つ増えたことになるので、命令は`LSR Rd, Rs, Offset5`となります。

残りの命令セットのデコードも、この論理パターンに従うだけです。

ここでは、最初の数個の命令をどのようにデコードするか、コード例を紹介します

条件分岐を使う場合は、

```cpp
u16 instruction = fetch();

switch (instruction >> 8)
{
    case 0:
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
    case 6:
    case 7:
    {
        // LSL Rd, Rs, Offset5
    } break;
    case 8:
    case 9:
    case 0xA:
    case 0xB:
    case 0xC:
    case 0xD:
    case 0xE:
    case 0xF:
    {
        // LSR Rd, Rs, Offset5
    } break;
}
```

関数ポインタの配列を使う場合は、

```cpp

void (*CPU_Thumb_Instruction[0x100]) (u16 instruction) =
{
    Thumb_LSL_Rd_Rs_Offset,        // 00h
    Thumb_LSL_Rd_Rs_Offset,        // 01h
    Thumb_LSL_Rd_Rs_Offset,        // 02h
    Thumb_LSL_Rd_Rs_Offset,        // 03h
    Thumb_LSL_Rd_Rs_Offset,        // 04h
    Thumb_LSL_Rd_Rs_Offset,        // 05h
    Thumb_LSL_Rd_Rs_Offset,        // 06h
    Thumb_LSL_Rd_Rs_Offset,        // 07h
    /*-------------------*/
    Thumb_LSR_Rd_Rs_Offset,        // 08h
    Thumb_LSR_Rd_Rs_Offset,        // 09h
    Thumb_LSR_Rd_Rs_Offset,        // 0Ah
    Thumb_LSR_Rd_Rs_Offset,        // 0Bh
    Thumb_LSR_Rd_Rs_Offset,        // 0Ch
    Thumb_LSR_Rd_Rs_Offset,        // 0Dh
    Thumb_LSR_Rd_Rs_Offset,        // 0Eh
    Thumb_LSR_Rd_Rs_Offset,        // 0Fh
};
```

