# 算術処理

Div, DivArm, Sqrt, ArcTan, ArcTan2

## Div: swi 0x06(GBA) & swi 0x09(NDS)

```
(r0, r1, r3) = r0/r1
```

```
引数:
  r0: 分子
  r1: 分母

戻り値:
  r0: 商
  r1: 余り
  r3: 商の絶対値
```

例:

(r0, r1) = (-1234, 10) のとき

(r0, r1, r3) = (-123, -4, +123)

0で割った時は無限ループに陥ります。

## DivArm: swi 0x07(GBA)

```
(r0, r1, r3) = r1/r0
```

引数の分母分子が逆以外はDivと同じです。 

ARMライブラリとの互換を保つための関数でDivより3クロックだけ遅いです。

## Sqrt: swi 0x08(GBA) & swi 0x0d(NDS)

```
r0: uint16 = sqrt(r0: uint32)
```

返り値は整数になります。例えばsqrt(2)は1になります。

## ArcTan: swi 0x09(GBA)

ArcTanを計算します。

```
引数:
  r0 Tan, 16bit (15bit: 符号部分, 14bit: 整数部分, 13-0bit: 小数部分)

戻り値:
  r0 = 0xc000-0x4000 (= `-PI/2<THETA/<PI/2` に対応)
```

`THETA<-PI/4, PI/4<THETA`において計算結果が不正確なバグがあるようです。

## ArcTan2: swi 0x0a(GBA)

```
引数:
  r0: X, 16bit (15bit: 符号部分, 14bit: 整数部分, 13-0bit: 小数部分)
  r1: Y, 16bit (15bit: 符号部分, 14bit: 整数部分, 13-0bit: 小数部分)

戻り値:
  r0 = 0x0000-0xffff (= `0<=THETA<2PI` に対応)
```
