# [UARTモード](https://mgba-emu.github.io/gbatek/#siouartmode)

```
Note: 筆者(pokemium)がハードウェアに詳しくないため、間違った記述があるかもしれません。
```

UART = Universal Asynchronous Receiver/Transmitter（汎用非同期送受信機）

このモードでは、RS232シリアルポートのような挙動をします。ただし電圧は明確になっていません。+/-12Vより0/3Vだろうと言われています。

SIとSOはクロスケーブルなデータ回線です。

SC and SD signalize Clear to Send (with crossed wires also, which requires special cable when linking between two GBAs ?)

## レジスタ

### 4000134h - RCNT (R) - モード選択レジスタ

SIOCNTのbit13-12と合わせて、転送モードを設定するのに利用します。

使われているのはbit15だけです。

ビット | 内容
---- | ---- 
0-3  | 未定義 (current SC,SD,SI,SO state, as for General Purpose mode)
4-8  | 不使用 (常に0が望ましいが1を書き込むこともできる)
9-13 | 不使用 (常に0,読み取り専用)
14   | 不使用 (常に0が望ましいが1を書き込むこともできる)
15   | 0 (モードをノーマル/マルチプレイ/UARTにしたい場合)

### 4000128h - SCCNT_L - SIO制御レジスタ (R/W)

ビット | 内容
---- | ---- 
0-1 | Baud Rate  (0-3: 9600,38400,57600,115200 bps)
2   | CTS Flag   (0=Send always/blindly, 1=Send only when SC=LOW)
3   | Parity Control (0=Even, 1=Odd)
4   | Send Data Flag      (0=Not Full,  1=Full)    (Read Only)
5   | Receive Data Flag   (0=Not Empty, 1=Empty)   (Read Only)
6   | Error Flag          (0=No Error,  1=Error)   (Read Only)
7   | Data Length         (0=7bits,   1=8bits)
8   | FIFO Enable Flag    (0=Disable, 1=Enable)
9   | Parity Enable Flag  (0=Disable, 1=Enable)
10  | Send Enable Flag    (0=Disable, 1=Enable)
11  | Receive Enable Flag (0=Disable, 1=Enable)
12  | Must be "1" for UART mode
13  | Must be "1" for UART mode
14  | IRQ Enable          (0=Disable, 1=IRQ when any Bit 4/5/6 become set)
15  | Not used            (Read only, always 0)

### 400012Ah - SIODATA8 - usage in UART Mode (R/W)

Addresses the send/receive shift register, or (when FIFO is used) the send/receive FIFO. 

In either case only the lower 8bit of SIODATA8 are used, the upper 8bit are not used.

The send/receive FIFO may store up to four 8bit data units each. 

For example, while 1 unit is still transferred from the send shift register, it is possible to deposit another 4 units in the send FIFO, which are then automatically moved to the send shift register one after each other.

## Send/Receive Enable, CTS Feedback

The receiver outputs SD=LOW (which is input as SC=LOW at the remote side) when it is ready to receive data (that is, when Receive Enable is set, and the Receive shift register (or receive FIFO) isn't full.

When CTS flag is set to always/blindly, then the sender transmits data immediately when Send Enable is set, otherwise data is transmitted only when Send Enable is set and SC is LOW.

## Error Flag

The error flag is set when a bad stop bit has been received (stop bit must be 0), when a parity error has occurred (if enabled), or when new data has been completely received while the receive data register (or receive FIFO) is already full.

The error flag is automatically reset when reading from SIOCNT register.

## Init & Initback

The content of the FIFO is reset when FIFO is disabled in UART mode, thus, when entering UART mode initially set FIFO=disabled.

The Send/Receive enable bits must be reset before switching from UART mode into another SIO mode!
