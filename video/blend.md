# ブレンド

ブレンドはBGやスプライトから2つのグループを選択し、一定の割合で色を混ぜて表示するアルファブレンドと、白か黒を一定の割合で混ぜて表示するフェードがあります

## 例

**アルファブレンド**

ロックマンゼロのオープニングムービーでゴーレムのレーザー(緑)とレジスタンスのオイル(赤)部分の重なりにアルファブレンドが使用されています。

<img src="../images/alphablend.png" width="240" />

**白フェード**

ロックマンエグゼ2の起動時、真っ白な画面からカプコンロゴが徐々にフェードインしてくる際に白フェードが使用されています。

<img src="../images/whitefade.png" width="240" />

**黒フェード**

ロックマンエグゼ2のカプコンロゴ表示後、カプコンロゴが徐々にフェードアウトする際に黒フェードが使用されています。

<img src="../images/blackfade.png" width="240" />

## 0x0400_0050 - BLDCNT - ブレンド制御レジスタ (R/W)

 bit | 内容
---- | ----
0     | BG0を1stターゲットに
1     | BG1を1stターゲットに
2     | BG2を1stターゲットに
3     | BG3を1stターゲットに
4     | OBJを1stターゲットに (Top-most OBJ pixel)
5     | BD(Backdrop)を1stターゲットに 
6-7   | 特殊効果の種類 (後述)  
8     | BG0を2ndターゲットに
9     | BG1を2ndターゲットに
10    | BG2を2ndターゲットに
11    | BG3を2ndターゲットに
12    | OBJを2ndターゲットに (Top-most OBJ pixel)
13    | BD(Backdrop)を2ndターゲットに
14-15 | 未使用

<table>
    <thead>
        <tr>
            <th>Addr</th>
            <th colspan=2 class="td-colspan">7-6</th>
            <th colspan=1 class="td-colspan">5</th>
            <th colspan=1 class="td-colspan">4</th>
            <th colspan=1 class="td-colspan">3</th>
            <th colspan=1 class="td-colspan">2</th>
            <th colspan=1 class="td-colspan">1</th>
            <th colspan=1 class="td-colspan">0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0x0400_0050</td>
            <td colspan=2 class="td-colspan">特殊効果</td>
            <td colspan=1 class="td-colspan">BD/1st</td>
            <td colspan=1 class="td-colspan">OBJ/1st</td>
            <td colspan=1 class="td-colspan">BG3/1st</td>
            <td colspan=1 class="td-colspan">BG2/1st</td>
            <td colspan=1 class="td-colspan">BG1/1st</td>
            <td colspan=1 class="td-colspan">BG0/1st</td>
        </tr>
        <tr>
            <td>0x0400_0051</td>
            <td colspan=2 class="td-colspan">不使用(0)</td>
            <td colspan=1 class="td-colspan">BD/2nd</td>
            <td colspan=1 class="td-colspan">OBJ/2nd</td>
            <td colspan=1 class="td-colspan">BG3/2nd</td>
            <td colspan=1 class="td-colspan">BG2/2nd</td>
            <td colspan=1 class="td-colspan">BG1/2nd</td>
            <td colspan=1 class="td-colspan">BG0/2nd</td>
        </tr>
    </tbody>
</table>

第1ターゲットと第2ターゲットに、どのBGやスプライトを割り当てるかの設定です。両方のグループに同じものを割り当てることもできます。

BG0-BG3はその番号のBG、OBJはスプライトすべて、BDは背景(詳細は不明)です。

フェードモードのときは第1ターゲットに設定したものだけが対象になります。

スプライトはスプライト同士の色の混ぜ合わせは行われないようです。

すべてのレイヤーが第1ターゲットと第2ターゲットの両方を選択することも可能で、 その場合は最上位の画素が第1ターゲットとなり、 次の下位の画素が第2ターゲットとなります。

### 特殊効果(bit6-7)

- 0: 特殊効果なし
- 1: 2つのグループの色を混ぜ合わせるアルファブレンド
- 2: 白フェード。グループと白色を一定の割合で混ぜます
- 3: 黒フェード。グループと黒色を一定の割合で混ぜます

## 0x0400_0052 - BLDALPHA - アルファブレンド (R/W)

<table>
    <thead>
        <tr>
            <th>bit</th>
            <th>15</th>
            <th>14</th>
            <th>13</th>
            <th>12</th>
            <th>11</th>
            <th>10</th>
            <th>9</th>
            <th>8</th>
            <th>7</th>
            <th>6</th>
            <th>5</th>
            <th>4</th>
            <th>3</th>
            <th>2</th>
            <th>1</th>
            <th>0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>内容</td>
            <td colspan=3 class="td-colspan">不使用</td>
            <td colspan=5 class="td-colspan">EVB</td>
            <td colspan=3 class="td-colspan">不使用</td>
            <td colspan=5 class="td-colspan">EVA</td>
        </tr>
    </tbody>
</table>

```
EVA: 第1ターゲット色割合 (0..16 = 0/16..16/16, 17..31=16/16)
EVB: 第2ターゲット色割合 (0..16 = 0/16..16/16, 17..31=16/16)
```

各ピクセルのRGBは次のように計算されます。

```
R = min(31, R1*EVA + R2*EVB)
G = min(31, G1*EVA + G2*EVB)
B = min(31, B1*EVA + B2*EVB)
```

## 0x0400_0054 - BLDY - フェード (W)

<table>
    <thead>
        <tr>
            <th>bit</th>
            <th colspan=27 class="td-colspan">31-5</th>
            <th colspan=5 class="td-colspan">4-0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>内容</td>
            <td colspan=27 class="td-colspan">不使用</td>
            <td colspan=5 class="td-colspan">EVY</td>
        </tr>
    </tbody>
</table>

```
EVY: フェード率 (0..16 = 0/16..16/16, 17..31=16/16)
```

各ピクセルのRGBは次のように計算されます。

**白フェード**

BLDCNTのbit6-7が2(白フェード)のときは次のようになります。

```
R = R1 + (31-R1)*EVY
G = G1 + (31-G1)*EVY
B = B1 + (31-B1)*EVY
```

どれも255に近づいているので明るくなり白に近づいています。

**黒フェード**

BLDCNTのbit6-7が3(黒フェード)のときは次のようになります。

```
R = R1 - (R1)*EVY
G = G1 - (G1)*EVY
B = B1 - (B1)*EVY
```

## 半透明のOBJ

OAMで半透明と定義されているOBJは、常に第1ターゲットとして選択され、常にアルファブレンドが適用されます。これはBLDCNT.4とBLDCNT.6-7に関係なく適用されます。

BLDCNTレジスタは、OBJ（または 他のBG/BDレイヤー）に対してフェードを行うために使用することもできます。

しかし、半透明のOBJピクセルが第2ターゲットのピクセルと重なっている場合、半透明なOBJに対するアルファブレンドが優先され、（第1ターゲットも第2ターゲットも）フェードは行われません。

## 補足

アルファブレンドは、OBJ-to-BGやBG-to-OBJには使用できますが、OBJ-to-OBJには使用できません。
