# ロード/ストア命令

LDR, STR が該当します

LDRH, LDRSB, LDRSH, STRH は [その2](./loadstore2.md)で解説しています。

LDM, STM は [その3](./loadstore3.md)で解説しています。

## 📜 命令のフォーマット

### LDR

`Rd=[Rn + offset]` or `Rd=[Rn - offset]`

メモリから1byte読み込む時は、Rdの上位24bitは0埋めされます。

#### I=0

I(Immediate)というフラグ名ですが、**I=0のときにオフセットに即値**を用いることに注意

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | 0b01
25 | 0 (I - オフセットは即値)
24 | P - Post/Pre (後述 0=post 1=pre)
23 | U - マイナス(-offset) か プラス(+offset) か (0=マイナス, 1=プラス)
22 | B - ロードの単位(0=32bit, 1=8bit)
21 | [後述](#-bit21について)
20 | 1 (オペコード `LDR{cond}{B}{T} Rd, ${Addr}`)
19-16 | Rn - ベースレジスタ (R0..R15) (このとき R15はPC+8)
15-12 | Rd - ターゲットレジスタ (R0..R15) (このとき R15はPC+12)
11-0  | 符号無し12bitのオフセット (0-4095)

#### I=1

I(Immediate)というフラグ名ですが、**I=1のときにオフセットにレジスタ**を用いることに注意

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | 0b01
25 | 1 (I - オフセットはレジスタ)
24 | P - Post/Pre (後述 0=post 1=pre)
23 | U - マイナス(-offset) か プラス(+offset) か (0=マイナス, 1=プラス)
22 | B - ロードの単位(0=32bit, 1=8bit)
21 | [後述](#-bit21について)
20 | 1 (オペコード `LDR{cond}{B}{T} Rd, ${Addr}`)
19-16 | Rn - ベースレジスタ (R0..R15) (このとき R15はPC+8)
15-12 | Rd - ターゲットレジスタ (R0..R15) (このとき R15はPC+12)
11-7 | Is - シフト量 (1-31, 0=ALUと同じ)
6-5  | シフトの種類 (0=LSL, 1=LSR, 2=ASR, 3=ROR)
4    | 0b0
3-0  | Rm - オフセットを格納したレジスタ (R0..R14)

### STR

`[Rn + offset]=Rd` または `[Rn - offset]=Rd`

メモリに32bit書き込みをする場合、対象のメモリアドレスはword単位のアラインメントをしておく必要があります。

#### I=0

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | 0b01
25 | 0 (I - オフセットは即値)
24 | P - Post/Pre (後述 0=post 1=pre)
23 | U - マイナス(-offset) か プラス(+offset) か (0=マイナス, 1=プラス)
22 | B - ストアの単位(0=32bit, 1=8bit)
21 | [後述](#-bit21について)
20 | 0 (オペコード `STR{cond}{B}{T} Rd, ${Addr}`)
19-16 | Rn - ベースレジスタ (R0..R15) (このとき R15はPC+8)
15-12 | Rd - ターゲットレジスタ (R0..R15) (このとき R15はPC+12)
11-0  | 符号無し12bitのオフセット (0-4095)

#### I=1

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | 0b01
25 | 1 (I - オフセットはレジスタ)
24 | P - Post/Pre (後述 0=post 1=pre)
23 | U - マイナス(-offset) か プラス(+offset) か (0=マイナス, 1=プラス)
22 | B - ストアの単位(0=32bit, 1=8bit)
21 | [後述](#-bit21について)
20 | 0 (オペコード `STR{cond}{B}{T} Rd, ${Addr}`)
19-16 | Rn - ベースレジスタ (R0..R15) (このとき R15はPC+8)
15-12 | Rd - ターゲットレジスタ (R0..R15) (このとき R15はPC+12)
11-7 | Is - シフト量 (1-31, 0=ALUと同じ)
6-5  | シフトの種類 (0=LSL, 1=LSR, 2=ASR, 3=ROR)
4    | 0b0
3-0  | Rm - オフセットを格納したレジスタ (R0..R14)

## 🔴 Post/Preについて

- Post: add offset after transfer 
- Pre: before trans.

## 🔴 bit21について

bit21は P(bit24)の値によって意味が変わってきます。

### P=0(Post)のとき

bit21はメモリ管理の仕方を表すフラグ(T)となります。

- 0: Normal
- 1: Force non-privileged access

### P=1(Pre)のとき

bit21はライトバックの有無を表すフラグ(W)となります。

- 0: ライトバックしない
- 1: Rnにアドレスをライトバックする

## 🛠 アセンブリ

アセンブリでロード/ストア命令を表す表記について説明します。

### Preアドレッシング(P=0)

```
  [Rn]                          ; offset = 0
  [Rn, <#{+/-}expression>]{!}   ; offset = 即値
  [Rn, {+/-}Rm{,<shift>} ]{!}   ; offset = Isだけシフトされたレジスタ
```

### Postアドレッシング(P=1)

```
  [Rn], <#{+/-}expression>      ;offset = immediate
  [Rn], {+/-}Rm{,<shift>}       ;offset = register shifted by immediate
```

### その他

```
  <shift>  immediate shift such like LSL#4, ROR#2, etc. (see ALU opcodes).
  {!}      exclamation mark ("!") indicates write-back (Rn will be updated).
```

## 🔎 補足

シフト量が0のときは特殊な挙動となります。詳しくは[ALU](./alu.md)を参照

フラグ:

- 不変(v4)
- v5では `LDR PC, $op`を実行した時に Tフラグに $opのbit0が格納されます

実行時間:

- LDR: 1S+1N+1I
- STR: 2N

## PLD

PLD = Prepare Cache for Load

PLD must use following settings cond=1111b, P=1, B=1, W=0, L=1, Rd=1111b, the address may not use post-indexing, and may not use writeback, the opcode is encoded identical as `LDRNVB R15,<Address>`.

PLDは、特定のメモリアドレスが間もなくアクセスされることをメモリシステムに知らせ、メモリシステムはこのヒントを利用してキャッシングやパイプライニングの準備をすることができます。それ以外は、PLDはプログラムロジックに何の影響も与えず、NOPと同じ動作をします。

PLDはARMv5TE以降でのみサポートされており、ARMv5やARMv5TExPではサポートされていません。
