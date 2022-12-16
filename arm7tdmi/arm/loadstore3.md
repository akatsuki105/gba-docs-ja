# ロード/ストア命令 その3

ロード/ストア命令のうち、スタックに関連する命令であるLDM, STMについて解説します

まずARMのスタックについて説明します。

## スタックのポリシー

まずは一般論から説明します。

Stack Pointer の指している先にデータがあるかないかで以下のように分かれます

- Full Stack: Stack Pointer は最後に使用された位置を指している
- Empty Stack: Stack Pointer は未使用な最初の位置を指している

また、アドレスが小さくなる方に成長するか、アドレスが大きくなる方に成長するかで以下のように分かれます。

- 下降スタック: メモリアドレスの降順に増加
- 上昇スタック: メモリアドレスの昇順に増加

だから、スタックのポリシー的には 2×2=4 通りが可能ということになります。

## ARMのスタックとそれに関する命令

ARMではスタックを表現するのに **Full&下降スタック** を使うようです。

`LDM`, `STM`命令はスタックのポリシーを考慮してさまざまな表記があります。

まず表記法として、非スタックアドレッシングとスタックアドレッシングがあります。

非スタックアドレッシングは

- LDMDA (Decrement After)
- LDMIA (Increment After)
- LDMDB (Decrement Before)
- LDMIB (Increment Before)
- STMDA (Decrement After)
- STMIA (Increment After)
- STMDB (Decrement Before)
- STMIB (Increment Before)

という表記になります。

上で述べたようにARMでは Fullスタック、下降スタックを採用するというので、 pushするときにはSTMDB命令を使用し、 popするときにはLDMIA命令を使用しないといけません。

これらのことを毎回考えるのは面倒なので、スタックアドレッシングの表記も用意されています。

- LDMFA (Full Ascending)
- LDMFD (Full Desending)
- LDMEA (Empty Ascending)
- LDMFD (Empty Descending)
- STMFA (Full Ascending)
- STMFD (Full Desending)
- STMEA (Empty Ascending)
- STMFD (Empty Descending)

非スタックアドレッシングとスタックアドレッシングの関係は次のようになります。

```
Non-stack | Stack adressing
LDMDA = LDMFA
LDMIA = LDMFD (= pop)
LDMDB = LDMEA
LDMIB = LDMED
STMDA = STMED
STMIA = STMEA
STMDB = STMFD (= push)
STMIB = STMFA
```

## フォーマット

 bit  |  内容
---- | ----
31-28 | 条件
27-25 | 0b100
24 | P - Post/Pre (後述 0=post 1=pre)
23 | U - Up/Down Bit (0=down; subtract offset from base, 1=up; add to base)
22 | S - PSR & force user bit (0=No, 1=load PSR or force user mode)
21 | W - Write-back bit (0=no write-back, 1=write address into base)
20 | L - オペコード
19-16 | Rn - ベースレジスタ (R0-R14)
15-0 | Rlist - レジスタのリスト

### オペコード(bit20)

- `{amod}`については後述
- `{!}`: Write-Back (W)
- `{^}`: PSR/User Mode (S)

```
0: STM{cond}{amod} Rn{!},<Rlist>{^}  ; Store (Push)
1: LDM{cond}{amod} Rn{!},<Rlist>{^}  ; Load  (Pop)
```

フラグ: 不変

実行時間(n=転送する合計ワード数):

- LDM: nS+1N+1I
- LDM PC: (n+1)S+2N+1I
- STM: (n-1)S+2N

## アドレッシング {amod}

PとUのbitの組み合わせで、IB,IA,DB,DAといったアドレッシングを指定します。

Suffix | expl | P | U 
-- | -- | -- | -- 
IB | increment before | 1 | 1 
IA | increment after  | 0 | 1 
DB | decrement before | 1 | 0 
DA | decrement after  | 0 | 0 

例:

```
LDMIA r13!, {r4-r7, r9, r15}
STMDB r13!, {r4-r7, r9, r14}
```

### アセンブリ

上で述べたように、`LDM`, `STM`命令にはaliasがあるものが存在します。これらはアセンブリの記述に使用可能で同じものとして扱われます。

例えば `STMDB r13!, {r4-r7, r9, r14}` という命令は 

- `STMFD r13!, {r4-r7, r9, r14}`
- `push {r4-r7, r9, r14}` 

と書けます。

## 順番

Rlistのレジスタはレジスタ番号が低いものが、

- 低いアドレスからロードを行う
- 低いアドレスにストアされる

順番で処理されます。

**例**

```
STMDB r13!, {r4-r7, r9, r14}
 -> STMDBするたびにストア先のアドレスが小さくなっていくので r14 -> r9 -> r7 -> ... -> r4 の順番
STMIA r13!, {r4-r7, r9, r14}
 -> STMIAするたびにストア先のアドレスが大きくなっていくので r4 -> r5 -> ... -> r14 の順番
LDMIB r13!, {r4-r7, r9, r15}
 -> LDMIBするたびに読み出し元のアドレスが大きくなっていくので r4 -> r5 -> ... -> r15 の順番
LDMDA r13!, {r4-r7, r9, r15}
 -> LDMDAするたびに読み出し元のアドレスが小さくなっていくので r15 -> r9 -> ... -> r4 の順番
```

## When S Bit is set (S=1)

### LDMでRlistにR15が含まれるとき

モードが変化します。

R15のロード時に、自動的に`CPSR=SPSR_${current_mode}`が行われます。

### それ以外のとき (User bank transfer)

RlistはユーザーバンクレジスタのR0～R15を指します。（R14_svcなどの現在のモードに関連するレジスタではありません）

Base write-back should not be used for User bank transfer.

**注意(LDMのときのみ)**

LDMの次の命令がバンクされたレジスタ（例：R14_svc）から読み出す場合、CPUは代わりにR14を読み出す可能性があり、必要に応じてダミーのNOPを挿入します。

