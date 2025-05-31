# 0x0400_0002 - GREENSWP - グリーンスワップ (R/W)

> [!WARNING]
> 信頼性低

Normally, red green blue intensities for a group of two pixels is output as BGRbgr (uppercase for left pixel at even xloc, lowercase for right pixel at odd xloc). 

When the Green Swap bit is set, each pixel group is output as BgRbGr (ie. green intensity of each two pixels exchanged).

 bit  |  内容
---- | ----
0 | グリーンスワップ (0=Normal, 1=Swap)
1 | 未使用

この機能は、最終的に生成される画像(つまり別々のBGとOBJレイヤを混合した後)に適用されるようです。 

最終的には、他のディスプレイタイプ（他のピンアウトを持つ）のために意図されています。

通常のGBAハードウェアでは、面白い汚れの効果を生み出しているだけです。

NDSのDISPCNTレジスタは32ビット(0x0400_0000...0x0400_0003)なので、NDSモードではグリーンスワップは存在しませんが、GBAモードではNDSはグリーンスワップをサポートしています。

