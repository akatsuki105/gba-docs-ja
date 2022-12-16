# agbasm

agbasm は gas<sup>[1](#gas)</sup> を改造したもので、GBAの逆アセンブラで使われているアセンブラです。

## コマンドオプション

> **Note**  
> https://pastebin.com/nFpfnVvH を翻訳したもの

<details>
  <summary>引数</summary>
  
  ```
  --agbasm          --agbasm-debug 以外の全ての agbasm のオプションを有効にします

  --agbasm-debug FILE
                    agbasm のデバッグ情報を有効にします。
                    Outputs miscellaneous debugging print statements to the specified file. ("printf debugging")

  --agbasm-colonless-labels
                    agbasm のコロン無しラベルを有効にします。
                    これは、ラベルが0列目で、かつ（オプションの空白文字の後の）改行で終わる場合、
                    最後にコロンを付けずにラベルを定義することを可能にします。
                    ラベルが改行で終わっていない場合、エラーが投げられ、ラベルは文であるとみなされます。

  --agbasm-colon-defined-global-labels
                    このオプションを有効にすると、ラベルの定義時に、ラベル名の後に1つではなく2つのコロンを付けることで、
                    ラベルをグローバルとして設定することができます。

  --agbasm-local-labels
                    agbasm のローカルラベルを有効にします。
                    $ラベル(例: $2)と同じものですが、ラベル名は数字に限定されません。
                    agbasmのローカルラベルは、接頭辞として "." を付けます。
                    内部的には、agbasmのローカルラベルは、実際には最も最近定義された非ローカルラベルとローカルラベルを連結したものに過ぎません。
                    これにより、ローカルラベル名を正規化する安全な方法が得られ、デバッグ情報のためにエクスポートすることができます。
                    また、正規化されたラベル名を使用することにより、ローカルラベルをそのスコープの外側から参照することができます。
                    agbasmのローカルラベルは、デフォルトではローカルシンボルではないことに注意してください。

  --agbasm-multiline-macros
                    enable agbasm multiline macros. This allows the use of
                    a macro to span across multiple lines, by placing a `['
                    after the macro name, and then placing a `]' once all
                    macro arguments have been defined, e.g.
                    
                    my_macro [
                        arg_1=FOO,
                        arg_2=BAR
                    ]
                    
                    In a multiline macro, the equal sign used in assigning
                    keyword arguments can substituted with a colon (`:'). Note
                    that there cannot be whitespace before the colon, but
                    there can be whitespace after the colon (this behavior also
                    exists in unmodified gas with the equal sign).
                    
                    The opening character (`[') must be defined before any
                    macro arguments are specified. Arguments can be defined
                    on the same line as the opening character with optional
                    whitespace in-between the opening character and the starting
                    argument, e.g.:
                    
                    my_macro [arg_1=FOO,
                      arg_2=BAR
                    ]
                    
                    The closing character (`]') can be defined in one of
                    two ways:
                    - After the last argument, a comma is placed (to indicate
                      the end of the argument), followed by optional whitespace
                      and then the closing character, e.g.:
                      
                      my_macro [
                          FOO, ]
                      
                    - On a single line by itself (supposedly after the last
                      argument has been defined) with no non-whitespace characters
                      before or after it, e.g.:
                      
                      my_macro [
                          FOO
                      ]
                      
                    Note that the first method **requires** a comma before the
                    closing character, while the second method does not require
                    the closing character. This is due to the inherent design
                    of how macro arguments are parsed, which may be explained
                    here in the future.
                    
                    A comma should be inserted after the last argument for each
                    line (except as mentioned above in the second closing character
                    method), otherwise a warning is generated. It is
                    recommended to not ignore these warnings as they can
                    be an indicator of a missing closing character, as most
                    directives do not end with a comma.

  --agbasm-charmap
                    agbasm charmapを有効にします。
                    これにより、文字列中の文字を、使用するエンコーディング(例: UTF-8)の値ではなく、カスタム値にマッピングするように指定することができます。
                    マッピングの指定には、 次のように .charmap マクロを用います。
                    
                        .charmap "A", 0x20
                        .string "A"
                    
                    この場合、0x20の値が出力されます。 .charmap のマッピングは1バイトに制限されず、マッピングは可能な限り長くすることができます。
                    マッピングのマッチ方法は、文字列の現在のポイントで定義された最長のパターンを見つけようとすることです。
                    例えば：
                    
                        .charmap "d", 1
                        .charmap "o", 2
                        .charmap "n", 3
                        .charmap "'", 4
                        .charmap "t", 5
                        .charmap "'t", 6
                        .string "don't"
                    
                    は `1, 2, 3, 4, 5' ではなく `1, 2, 3, 6' となります。(`'t`のほうが、`'`より長いので)
                    もっと複雑な例を挙げるなら、
                    
                        .charmap "B", 1
                        .charmap "A", 2
                        .charmap "N", 3
                        .charmap "BAN", 4
                        .charmap "BANANA", 5
                        .charmap "ANA", 6
                        .string "BANAN"
                    
                    は `4, 2, 3' となります。
                    
                    .charmapの出力値のサイズは、1バイトより長くすることができます。これには2つの方法があります。
                    1つ目は、出力値としてバイトのリストを指定する方法です。
                    
                        .charmap "C", 0x20, 0x21, 0x22
                        .string "C"
                    
                    これは `0x20, 0x21, 0x22' となります。
                    2つ目の方法は，最大4バイトの値を1つだけ指定する方法です。
                    この値は可変幅として解釈され、先頭のゼロは無視されます。
                    例えば、値が 0x100 未満なら1バイトとなり、0x10000 未満なら2バイトとなります。
                    出力されるバイトはシステムのエンディアンに関係なくビッグエンディアンとなることに注意してください。つまり

                        .charmap "X", 0x0000E051
                        .string "X"
                    
                    は `0xE0, 0x51` となります。
                    
                    .string が charmap に使用されるようになったため、元の .stringN の動作を再現するために agbasmでは新たに .ascizN が存在します。
                    charmapが有効でない場合、.ascizN は有効ではありません。
                    
                    内部的には、agbasm の charmap はツリー構造で表現されます。
                    文字列のパースは O(n) になるように設計されており、メモリ消費もできるだけ抑えられています。
                    n文字の .charmap エントリは、64ビットマシンでは最悪の場合、256*n + 8 バイト を消費します。
                    上記の例 "BANAN" のパースのフローは、おおよそ次のようになります。
                        - Recognize B as a potential match and save it as the last match
                        - Parse BA, has no match
                        - Recognize BAN as a potential match and save it
                        - Parse BANA, has no match
                        - Parse BANAN, has no match
                        - Reached end of string, output the value of the last match
                          which is `BAN', and set the input pointer to the end of the
                          last match, which is after 'BAN`
                        - Recognize A as a potential patch and save it
                        - Parse AN, has no match
                        - Reached end of string, output the value of `A' and set the
                          input pointer to the end of `A'
                        - Recognize N as a potential match and save it
                        - Reached end of string, output the value of `N' and stop
                          parsing

  --agbasm-no-gba-thumb-after-label-disasm-fix
                    When viewing an .elf file in no$gba's disassembler, if one or more
                    Thumb opcodes exist in memory, and a label is declared after the
                    Thumb opcodes, no$gba will interpret subsequent Thumb opcodes after
                    the label as arm, even though the opcodes afterwards are Thumb. This
                    is due to (presumably) how no$gba reads the elf file to determine which
                    opcodes are thumb and which opcodes are arm. What presumably happens is
                    that no$gba reads the elf's symbol table. Within the symbol table, there
                    are also special mapping symbols, which indicate whether data at the
                    address of the mapping symbol is a sequence of ARM opcodes ($a), a
                    sequence of Thumb opcodes ($t), or a sequence of data items ($d). For
                    example, if there is a $t symbol with address 0x8000000, this tells no$gba
                    that address 0x8000000 is the start of a sequence of Thumb opcodes, and
                    thus no$gba's disassembler will output Thumb opcodes until it encounters
                    another mapping symbol. However, if a symbol that is not a mapping symbol
                    is encountered, no$gba will switch to outputting arm opcodes by default.
                    This becomes an issue as when a function emits a label, any code after
                    the label will be viewed incorrectly as ARM opcodes in no$gba's
                    disassembler. This flag aims to workaround this behavior, by having
                    the assembler clobber the current mapping state, so that it will
                    output another mapping symbol after the label, and thus no$gba will
                    output the correct type of data in the disassembler after a label.

  --agbasm-help     このメッセージを出力します
  ```
</details>

## 注釈

<sup id="gas">1: Fork of GNU assembler 2.31 (arm-none-eabi-as)</sup>
