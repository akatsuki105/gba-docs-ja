# サウンド

GBAのサウンドには以下の2種類のサウンドチャンネルが用意されています。

- トーンとノイズ用の4つの「アナログ」サウンドチャンネル（CGBサウンドとほぼ互換性あり）
- 2つの「デジタル」サウンドチャンネル（PCMサウンドチャンネル、DMAサウンドチャンネルとも。8bitDMAサンプルデータの再生に使用可能）

GBAにはモノラルスピーカーが1つだけ内蔵されているだけなので、通常は音声は1チャンネルにミキシングされて出力されます。

（ステレオヘッドフォンなどの）外部のラインアウト端子を使えば、各チャンネルを左右どちらかのチャンネルに出力することができます。

## 1️⃣ CH1 - 矩形波(スイープあり)

### 0x0400_0060 - SOUND1CNT_L (NR10) - CH1制御レジスタ(スイープ) (R/W)

> Sweep: 一定の速度で周波数を変化させる事

```
  Bit        Expl.
  0-2   R/W  スイープ時のシフト量 (n: 0-7)
  3     R/W  スイープの増減
               0: + (周波数は大きくなる)
               1: - (周波数は小さくなる)
  4-6   R/W  スイープ時間(スイープの起こる間隔)
                0b000: スイープしない
                0b001: 7.8 ms  (1/128Hz)
                0b010: 15.6 ms (2/128Hz)
                0b011: 23.4 ms (3/128Hz)
                0b100: 31.3 ms (4/128Hz)
                0b101: 39.1 ms (5/128Hz)
                0b110: 46.9 ms (6/128Hz)
                0b111: 54.7 ms (7/128Hz)
  7-15  -    不使用
```

スイープタイム(bit4-6)を0にすることでスイープが無効になりますが、その場合はdirectionビット(bit3)を1にセットする必要があります。

各シフトにおける周波数の変化(NR13,NR14)は、`X(0)`を初期周波数、`X(t-1)`を最終周波数として、以下の式で算出されます。

```c
  X(t) = X(t-1) ± X(t-1)/2^n
```

### 0x0400_0062 - SOUND1CNT_H (NR11, NR12) - CH1制御レジスタ(音色・長さ・音量) (R/W)

```
  Bit        Expl.
  0-5   W    サウンドの長さ (t1: 0-63)
                音声の長さは (64-t1)*(1/256) 秒
                NR14のbit6が設定されている場合にのみ使用
  6-7   R/W  デューティ比
                0b00: 12.5% ( -_______-_______-_______ )
                0b01: 25%   ( --______--______--______ )
                0b10: 50%   ( ----____----____----____ ) (normal)
                0b11: 75%   ( ------__------__------__ )
  8-10  R/W  エンベロープ速度 (n: 0-7)
                音量の変化(エンベロープ)のインターバルは n/64 秒
                nが小さいほど音量の変化は早い (nが0だと変化なし)
  11    R/W  音量変更方向 (0=小さくなっていく, 1=大きくなっていく)
  12-15 R/W  エンベロープ初期音量 (0-0Fh) (0=音無し)
```

### 0x0400_0064 - SOUND1CNT_X (NR13, NR14) - CH1制御レジスタ(周波数) (R/W)

```
  Bit        Expl.
  0-10  W    周波数; 131072/(2048-n)Hz  (n: 0-2047)
  11-13 -    不使用
  14    R/W  長さカウンタ有効フラグ(1=NR11の音声の長さが0になったとき音声が止まる)
  15    W    Initial (1=Restart Sound)
  16-31 -    不使用
```

## 2️⃣ CH2 - 矩形波

このサウンドチャンネルは、スイープレジスタがないことを除けば、チャンネル1と同じように動作します。

### 0x0400_0068 - SOUND2CNT_L (NR21, NR22) - Channel 2 Duty/Len/Envelope (R/W)
### 0x0400_006A - 不使用
### 0x0400_006C - SOUND2CNT_H (NR23, NR24) - Channel 2 Frequency/Control (R/W)

詳しくはチャンネル1の説明をみてください。

## 3️⃣ CH3 - 波形メモリ

このサウンドチャンネルはデジタルサウンドを出力するためのものです。サンプルバッファ（波形メモリ）の長さは、32桁または64桁（4ビットサンプル）です。

また、波形メモリを矩形波で初期化することで、通常の音を出力することもできます。このチャンネルには、エンベロープ機能はありません。

### 0x0400_0070 - SOUND3CNT_L (NR30) - Channel 3 Stop/Wave RAM select (R/W)

```
  Bit        Expl.
  0-4   -    不使用
  5     R/W  波形メモリのレイアウト   (0=One bank/32 digits, 1=Two banks/64 digits)
  6     R/W  波形メモリのバンク番号 (0-1, see below)
  7     R/W  Sound Channel 3 Off  (0=Stop, 1=Playback)
  8-15  -    不使用
```

現在bit6で選択されているバンク番号が再生され、Wave RAMへの読み書きは、もう一方の（選択されていない）バンクをアドレスとします。

`NR30.5=1`(2バンク)の場合、出力は現在選択されているバンクの再生から始まります。

### 0x0400_0072 - SOUND3CNT_H (NR31, NR32) - Channel 3 Length/Volume (R/W)

```
  Bit        Expl.
  0-7   W    Sound length; units of (256-n)/256s  (0-255)
  8-12  -    不使用
  13-14 R/W  Sound Volume  (0=Mute/Zero, 1=100%, 2=50%, 3=25%)
  15    R/W  Force Volume  (0=Use above, 1=Force 75% regardless of above)
```

長さの値(NR34.0-7)は、`NR34.6=1`の場合にのみ使用されます。

### 0x0400_0074 - SOUND3CNT_X (NR33, NR34) - Channel 3 Frequency/Control (R/W)

```
  Bit        Expl.
  0-10  W    サンプルレート(n=0..2047); = 2097152/(2048-n) Hz
  11-13 -    不使用
  14    R/W  Length Flag  (1=Stop output when length in NR31 expires)
  15    W    Initial      (1=Restart Sound)
  16-31 -    不使用
```

サンプルレートはbit0-10の値をnとすると、`2097152/(2048-n) Hz`となります。

このサンプルレートは、1秒あたりのWave RAMの桁数を指定するもので、実際のトーン周波数はWave RAMの内容に依存します。次に例を示します。

```
  Wave RAM, single bank 32 digits   Tone Frequency
  FFFFFFFFFFFFFFFF0000000000000000  65536/(2048-n) Hz
  FFFFFFFF00000000FFFFFFFF00000000  131072/(2048-n) Hz
  FFFF0000FFFF0000FFFF0000FFFF0000  262144/(2048-n) Hz
  FF00FF00FF00FF00FF00FF00FF00FF00  524288/(2048-n) Hz
  F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0  1048576/(2048-n) Hz
```

### 0x0400_0090 - WAVE_RAM0_L - Channel 3 Wave Pattern RAM (W/R)
### 0x0400_0092 - WAVE_RAM0_H - Channel 3 Wave Pattern RAM (W/R)
### 0x0400_0094 - WAVE_RAM1_L - Channel 3 Wave Pattern RAM (W/R)
### 0x0400_0096 - WAVE_RAM1_H - Channel 3 Wave Pattern RAM (W/R)
### 0x0400_0098 - WAVE_RAM2_L - Channel 3 Wave Pattern RAM (W/R)
### 0x0400_009A - WAVE_RAM2_H - Channel 3 Wave Pattern RAM (W/R)
### 0x0400_009C - WAVE_RAM3_L - Channel 3 Wave Pattern RAM (W/R)
### 0x0400_009E - WAVE_RAM3_H - Channel 3 Wave Pattern RAM (W/R)

このレジスタでCH3の波形メモリ(32x4bits)にアクセスできます。

データは、第1バイトのMSB、第1バイトのLSB、第2バイトのMSB......という順序で再生されますが、16ビットデータ単位でWave RAMを埋めていくと、4-7、0-3、12-15、8-11ビットにサンプルが配置されるなど、混乱した順序になってしまいます。

GBAでは、2つのWaveパターン（各32×4bits）が存在し、どちらか一方を再生し（NR30レジスタで選択）、もう一方のバンクにユーザーがアクセスすることができます。32個のサンプルがすべて再生された後、同じバンク（またはNR30で指定された他のバンク）の出力が自動的に再開されます。

内部的には、Wave RAMは巨大なシフトレジスタであり、現在再生されている桁を指すポインタは存在しません。その代わり、128ビット全体がシフトされ、最下位4ビットが出力されます。
そのため、Wave RAMからデータを読み出す際には、データの位置が変わっている可能性があります。また、Wave RAMに書き込む際には、すべてのデータを更新する必要があります（古いデータが、以前に書き込まれたのと同じ位置にあると考えるのはよくありません）。

## 4️⃣ CH4 - ノイズ

ホワイトノイズを出力するためのチャンネルです。これは、一定の周波数で、振幅を高低の間でランダムに切り替えることによって行われます。周波数に応じて、ノイズは「より硬く」または「より柔らかく」見えます。

また、ランダムジェネレータの機能に影響を与えて、出力がより規則的になるようにすることも可能で、その結果、ノイズの代わりにトーンを出力するという限定的な機能になります。

### 0x0400_0078 - SOUND4CNT_L (NR41, NR42) - Channel 4 Length/Envelope (R/W)

```
  Bit        Expl.
  0-5   W    Sound length; units of (64-n)/256s  (0-63)
  6-7   -    不使用
  8-10  R/W  Envelope Step-Time; units of n/64s  (1-7, 0=No Envelope)
  11    R/W  Envelope Direction                  (0=Decrease, 1=Increase)
  12-15 R/W  Initial Volume of envelope          (1-15, 0=No Sound)
  16-31 -    不使用
```

長さ(`NR41.0-5`)の値は、`NR44.6=1`の場合のみ使用されます。

### 0x0400_007C - SOUND4CNT_H (NR43, NR44) - Channel 4 Frequency/Control (R/W)

振幅は与えられた周波数で高低をランダムに切り替えます。周波数が高いほど、ノイズは「ソフト」になります。

bit3を設定すると、出力はより規則的になり、いくつかの周波数はノイズよりもトーンに近い音になります。

```
  Bit        Expl.
  0-2   R/W  Dividing Ratio of Frequencies (r)
  3     R/W  モード; 0=15bit LFSR, 1=7bit LFSR
  4-7   R/W  Shift Clock Frequency (s)
  8-13  -    不使用
  14    R/W  Length Flag  (1=Stop output when length in NR41 expires)
  15    W    Initial      (1=Restart Sound)
  16-31 -    不使用
```

```
周波数 = 524288 Hz / r / 2^(s+1)     ; r=0のときはr=0.5として扱います
```

### Noise Random Generator (aka Polynomial Counter)

ノイズはHIGHとLOWの間をランダムに行ったり来たりし、出力のレベルは選択された周波数においてシフトレジスタ(X)を用いて次のように計算します。

```
 7bit:  X=X SHR 1, IF carry THEN Out=HIGH, X=X XOR 60h ELSE Out=LOW
 15bit: X=X SHR 1, IF carry THEN Out=HIGH, X=X XOR 6000h ELSE Out=LOW
```

サウンドをリスタートしたときの初期値は`X=0x40(7bit)`か`X=0x4000(15bit)`です。 データストリームは、`0x7F(7bit)`または`0x7FFF(15bit)`のステップを経て繰り返されます。

## 🔉 PCMサウンドチャンネルA/B

GBAには2つのDMAサウンドチャンネル（AとB）があり、それぞれ int8 のPCM音源を再生することができます。PCMサウンドチャンネルとも言われています。

データは、DMA1またはDMA2を使用して、内部メモリ（外部メモリのときに動作するかどうかは不明）からFIFOに転送することができ、サンプルレートはタイマーのいずれかを使用して生成されます。

### 0x0400_00A0 - FIFO_A_L - Sound A FIFO, Data 0 and Data 1 (W)
### 0x0400_00A2 - FIFO_A_H - Sound A FIFO, Data 2 and Data 3 (W)

これらの2つのレジスタは、32bit（4バイト）のオーディオデータ（データ0〜3、データ0は最初に再生される最下位バイトに位置する）を受け取ることができます。

内部的には、FIFOの容量は8×32bit（32バイト）で、少量のサンプルをバッファリングすることができます。First In First Outという名の通り、最も古いデータが最初に再生されます。

### 0x0400_00A4 - FIFO_B_L - Sound B FIFO, Data 0 and Data 1 (W)
### 0x0400_00A6 - FIFO_B_H - Sound B FIFO, Data 2 and Data 3 (W)

サウンドチャネルBのためのもので、詳細は上記と同じです。

### Initializing DMA-Sound Playback

- Select Timer 0 or 1 in SOUNDCNT_H control register.
- Clear the FIFO.
- Manually write a sample byte to the FIFO.
- Initialize transfer mode for DMA 1 or 2.
- Initialize DMA Sound settings in sound control register.
- Start the timer.

### DMA-Sound Playback Procedure

次の疑似コードで表される処理が自動的に繰り返されます。

```
If Timer overflows then
  Move 8bit data from FIFO to sound circuit.
  If FIFO contains only 4 x 32bits (16 bytes) then
    Request more data per DMA
    Receive 4 x 32bit (16 bytes) per DMA
  Endif
Endif
```

この再生メカニズムは、サンプルバッファの実際の長さにかかわらず、永遠に繰り返されます。

### Synchronizing Sample Buffers

バッファの終端は、サウンドタイマーのIRQ（各サンプルバイト）、またはサウンドDMAのIRQ（それぞれ16番目のサンプルバイト）をカウントすることで判断できます。

どちらの方法も多くのCPU時間(IRQ処理)を必要とし、また、割り込み禁止時間が長くなると失敗します。

より良い解決策は、サンプルレートまたはバッファ長をVBlankと同期させるか、希望のサンプル数の後にIRQを生成する第2のタイマー（カウントアップ/スレーブモード）を使用することです。

### サンプルレート

GBAのハードウェアは、(SOUNDBIASのデフォルト設定では)サウンド出力を内部的に 32768Hz にリサンプリングしています。

そのため、DMA/Timerのレートを 32768Hz、 16384Hz、 8192Hz、つまり32768Hzの1倍, 1/2倍, 1/4倍にすることで、最高のリサンプリング精度が得られます。

## 🎛 サウンド制御レジスタ

### 0x0400_0080 - SOUNDCNT_L (NR50, NR51) - PSG音量制御 (R/W)

```
  Bit        Expl.
  0-2   R/W  Sound 1-4 Master Volume RIGHT (0-7)
  3     -    不使用
  4-6   R/W  Sound 1-4 Master Volume LEFT (0-7)
  7     -    不使用
  8-11  R/W  Sound 1-4 Enable Flags RIGHT (each Bit 8-11, 0=Disable, 1=Enable)
  12-15 R/W  Sound 1-4 Enable Flags LEFT (each Bit 12-15, 0=Disable, 1=Enable)
```

### 0x0400_0082 - SOUNDCNT_H (GBA only) - DMA Sound Control/Mixing (R/W)

```
  Bit        Expl.
  0-1   R/W  Sound # 1-4 Volume   (0=25%, 1=50%, 2=100%, 3=Prohibited)
  2     R/W  DMA Sound A Volume   (0=50%, 1=100%)
  3     R/W  DMA Sound B Volume   (0=50%, 1=100%)
  4-7   -    不使用
  8     R/W  DMA Sound A Enable RIGHT (0=Disable, 1=Enable)
  9     R/W  DMA Sound A Enable LEFT  (0=Disable, 1=Enable)
  10    R/W  DMA Sound A Timer Select (0=Timer 0, 1=Timer 1)
  11    W?   DMA Sound A Reset FIFO   (1=Reset)
  12    R/W  DMA Sound B Enable RIGHT (0=Disable, 1=Enable)
  13    R/W  DMA Sound B Enable LEFT  (0=Disable, 1=Enable)
  14    R/W  DMA Sound B Timer Select (0=Timer 0, 1=Timer 1)
  15    W?   DMA Sound B Reset FIFO   (1=Reset)
```

### 0x0400_0084 - SOUNDCNT_X (NR52) - Sound on/off (R/W)

bit0-3は、音声出力開始時に自動的に設定され、音声が終了すると自動的にクリアされます。

つまり、長さが有効になっている限り、その長さ分の再生が終了するとクリアされます。また、ボリュームエンベロープが終了しても、ビットはリセットされません

```
  Bit        Expl.
  0     R    CH1再生中
  1     R    CH2再生中
  2     R    CH3再生中
  3     R    CH4再生中
  4-6   -    不使用
  7     R/W  PSG/FIFO Master Enable (0=Disable, 1=Enable) (Read/Write)
  8-31  -    不使用
```

bit7がクリアされている間は、PSGとFIFOの両方のサウンドが無効になり、4000060h~4000081hのすべてのPSGレジスタがゼロにリセットされます。（サウンドを再度有効にした後、再度初期化する必要があります）

ただし、レジスタ4000082hと4000088hは読み書き可能です。このうち、4000082hはサウンドがオフのときには機能しませんが、4000088hはサウンドがオフでも機能します。

### 0x0400_0088 - SOUNDBIAS - PWM制御レジスタ (R/W)

このレジスタは、最終的なサウンド出力を制御します。デフォルトの設定は`0x200`で、通常はこの値を変更する必要はありません。

```
  Bit        Expl.
  0     -    不使用
  1-9   R/W  バイアス値 (Default=100h, converting signed samples into unsigned)
  10-13 -    不使用
  14-15 R/W  振幅分解能 (Default=0, see below)
               0b00: 9bit / 32.768kHz   (Default, best for DMA channels A,B)
               0b01: 8bit / 65.536kHz
               0b10: 7bit / 131.072kHz
               0b11: 6bit / 262.144kHz  (Best for PSG channels 1-4)
  16-31 -    不使用
```

このレジスタの詳細については、後述の説明をみてください。

### Max Output Levels (with max volume settings)

2つのFIFOは、それぞれ出力範囲の最大値（±200h）に対応しています。 4つのPSGは、それぞれ出力範囲の4分の1（±80h）に対応します。6つのチャンネルの現在の出力レベルは、ハードウェアによって加算されます。

つまり、FIFOとPSGを合わせると、出力範囲の3分の1（±600h）に達します。

BIAS値はその符号付きの値に加えられます。デフォルトのBIAS(200h)では、可能な範囲は-400h~+800hとなりますが、符号なし10ビットの出力範囲0～3FFhを超える値は、MinMax(0,3FFh)にクリップされます。

### Resampling to 32.768kHz / 9bit (default)

PSGのチャンネル1〜4は内部的に262.144kHzで生成されており、DMAのサウンドA〜Bは理論的には16.78MHzまでのタイマーレートで生成可能です。

しかし、最終的に出力される音は、32.768kHzのレートで、9bitの深さ（上記の10bitの値を2で割った値）にリサンプリングされます。

必要に応じて、32.768kHzよりも高いレートをSOUNDBIASレジスタで選択することができますが、その場合は9bitよりも小さい深さになります。

### PWM (Pulse Width Modulation) Output 16.78MHz / 1bit

さて、いよいよ実際の出力です。GBAは2つの電圧（LowとHigh）しか出力できませんが、この「ビット」はシステムクロック速度（16.78MHz）で出力されます。

デフォルトの32.768kHzのサンプリングレートを使用した場合、1サンプルあたり512ビットが出力されます（512*32K=16M）。

各サンプル値（9ビット範囲、N=0\~511）は、N個の下位ビットと512\~N個の上位ビットで出力されます。

このようにして得られた「ノイズ」は、コンデンサやスピーカー、そして人間の聴覚によって平滑化され、32kHzのサンプリングレートで9ビットの電圧をD/A変換したクリーンな音になります。

### BIASレベルの変更について

通常、きれいな音を出力するには200hを使用します。音が出ない期間（PWM回路がロービットのみを出力することで、消費電力を抑えたり、32KHzのノイズを防いだりしている）は、000hの値を使用すると良いでしょう。

またSoundBias機能(SWI19h)を使用すると，ハードスクラッチノイズを発生させずに，ゆっくりとBIASレベルを増減させることができます。

### 低電力モード

サウンド出力を使用しない場合は、4000084h（PSG/FIFO）と4000088h（BIAS）の両方を0にすることで、消費電力を抑えることができます。
