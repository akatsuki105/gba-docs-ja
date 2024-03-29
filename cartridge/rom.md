# カートリッジROM

## ROM Size

基本的に8MBや16MBが主流です。

FZeroとスーパーマリオアドバンスは4MBです。

## ROM Waitstates

GBAは起動時にカートリッジのWaitStateを`N=4, S=2`に設定し[プリフェッチ](./prefetch.md)は無効にします。これらの設定は[WAITCNT](../system.md#0x0400_0204---waitcnt---waitstate制御レジスタ-rw)を介して変更可能です。

例えば、FZeroとスーパーマリオアドバンスはWaitStateを`N=3, S=1`にしプリフェッチを有効にしています。

プリフェッチ機能を有効にしてWaitStateを短く設定すると、ROMから多くのデータとオペコードを高速に読み取ることができますが、消費電力が増加することに注意してください。

## ROMチップ

24bitのアドレスがカートリッジのバスを通過するため、カートリッジには、ファーストアクセス時に下位16bitのアドレスをラッチし、セカンドアクセス時にこれらのビットをインクリメントする回路が必要となります。

任天堂はこの回路をROMチップに直接組み込んでいるようです。

また、カートリッジは16bit幅のデータバス、または2つの8bitのデータをwaitstateの間に1つの16bitデータに変換する回路のどちらかを持っていなければなりません。
