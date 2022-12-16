# 伸縮回転

BgAffineSet, ObjAffineSet

## BgAffineSet: swi 0x0e(GBA)

BGの伸縮回転パラメータを計算するのに使用します。

```
引数:
  r0: ソースデータへのポインタ
    s32  変換前のデータのX座標 (8bitの小数部分あり)
    s32  変換前のデータのY座標 (8bitの小数部分あり)
    s16  変換後のデータの中心X座標
    s16  変換後のデータの中心Y座標
    s16  X方向の伸縮率 (8bitの小数部分あり)
    s16  Y方向の伸縮率 (8bitの小数部分あり)
    u16  回転角度 (8bitの小数部分あり) (0x0~0xffff)
  r1: ターゲットデータへのポインタ
    s16  Difference in X coordinate along same line
    s16  Difference in X coordinate along next line
    s16  Difference in Y coordinate along same line
    s16  Difference in Y coordinate along next line
    s32  X座標の開始点
    s32  Y座標の開始点
  r2: 計算数(ソースデータおよびターゲットデータは配列になっていてその要素数)

返り値: なし
```

## ObjAffineSet: swi 0x0f(GBA)

OBJのアフィン変換のパラメータを拡大縮小率と回転角度から計算してセットする機能です。

アフィン変換のパラメータはSrc(r0)が示すパラメータから計算されます。

4つのアフィン変換のパラメータ(pa, pb, pc, pd)は、Dst(r1)が示すアドレスから、Offsetバイト(r3)ごとに設定されます。

Offsetの値が2であれば、パラメータは連続して保存(`0x0`, `0x2`, `0x4`, `0x6`, ...)され、値が8の場合、OAMの構造と一致します。(`0x0`, `0x8`, `0x10`, `0x18`, ...)

Srcのポインタが配列になっている場合は、Num(r2)を指定することにより、その数だけ連続して計算を行います。

```
引数:
  r0: ソースデータへのポインタ
    s16  X方向の伸縮率 (8bitの小数部分あり)
    s16  Y方向の伸縮率 (8bitの小数部分あり)
    u16  回転角度 (8bitの小数部分あり) (0x0~0xffff)
  r1: ターゲットデータへのポインタ
    s16  Difference in X coordinate along same line
    s16  Difference in X coordinate along next line
    s16  Difference in Y coordinate along same line
    s16  Difference in Y coordinate along next line
  r2: 計算数(ソースデータおよびターゲットデータは配列になっていてその要素数)
  r3: Offset

返り値: なし
```

## 補足

Bg-、ObjAffineSetともに、回転角度は0x0～0xffff（0~360度）で指定できますが、GBAのBIOSは上位8bitのみを参照し、下位8bitには端数が含まれている可能性がありますが、BIOSでは無視しています。
