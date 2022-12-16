# PAR

## 用語解説

- GSA: GameShark Advance。プロアクションリプレイ本体のこと。アメリカではGameSharkという名前で売られている。
- PARスイッチ: プロアクションリプレイ本体についているボタン。別名、GSボタン。

## フォーマット

<table>
  <tbody>
    <tr>
      <td>単純書き込み(8bit)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        RAMアドレス<code>aaaaaaa</code>に値<code>xx</code>を連続的に書き込みつづけます。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>0aaaaaaaa 000000xx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>単純書き込み(16bit)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        RAMアドレス<code>aaaaaaa</code>に16bit値<code>xxxx</code>を連続的に書き込みつづけます。アドレスは2でアラインメントされている必要があります。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>1aaaaaaaa 0000xxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>単純書き込み(32bit)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        RAMアドレス<code>aaaaaaa</code>に32bit値<code>xxxxxxxx</code>を連続的に書き込みつづけます。
        アドレスは4でアラインメントされている必要があります。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>2aaaaaaaa xxxxxxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>複数同時書き込み(32bit)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        複数のアドレスに32bit値<code>xxxxxxxx</code>を書き込みます。
        対象のアドレスの個数は<code>cccc</code>です。対象のアドレスはxxxxxxxxの後に続きます。
        この機能にはバグがあり、書き込む値<code>xxxxxxxx</code>もアドレスと認識してしまうようです。
        <br/><br/>
        例: <code>30000004 01010101 03001FF0 03001FF4 03001FF8 00000000</code>
        <br/>
        01010101を01010101, 03001FF0, 03001FF4, 03001FF8に対して書き込みます。
        <br />
        '00000000' is used for padding, to ensure the last code encrypts
        correctly.
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>3000cccc xxxxxxxx aaaaaaaa</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>単純ROM書き込み(16bit)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        08000000h以降のROMエリアを変更する場合に使用します。<br />
        このコードの実行中、GSAは、指定したROMアドレスの読み込みを傍受し、値xxxxを返すことで実現されています。
        対象のアドレスは<code>aaaaaaa << 1</code>です。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>6aaaaaaa 0000xxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>単純ROM書き込み(16bit) その2</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        Similar to first ROM patch code, except patch is enabled before the game starts, instead of waiting for the code handler to enable the patch. <br/>
        対象のアドレスは<code>aaaaaaa << 1</code>です。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>6aaaaaaa 1000xxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>単純ROM書き込み(16bit) その3</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        上記二つとの違いは不明
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>6aaaaaaa 2000xxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>スイッチ書き込み(8bit)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        PARスイッチ押下時のみ8ビットRAM書き込みをします。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>8a1aaaaa 000000xx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>スイッチ書き込み(16bit)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        PARスイッチ押下時のみ16ビットRAM書き込みをします。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>8a2aaaaa 0000xxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>スイッチ書き込み(32bit)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        PARスイッチ押下時のみ32ビットRAM書き込みをします。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>8a4aaaaa xxxxxxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>スローモーション</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        PARスイッチを押すことによってゲームがスローモーションになります。xの値で速度を調整します。FFFFが最低速。再度押すと元に戻ります。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>80F00000 0000xxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>条件比較(1)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        アドレス<code>aaaaaaa</code>のデータと、16bit値<code>xxxx</code>を比較し、条件(<code>c</code>)に合致する場合のみ次の行の改造コードを実行します。
        <br />
        <code>c</code>: 0: =、1: ≠、2: ≦、3: ≧ (<code>xxxx</code>が右辺)
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>Daaaaaaa 0c00xxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>条件比較(2)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        (1)と同様ですが、条件(<code>c</code>)に合致しない場合、<code>zz</code>行だけ以降の改造コード群を一気にスキップします。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>E0zzxxxx caaaaaaa</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>Hook Routine (For Enablers)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        Used to insert the GS code handler routine where it will be executed at least 20 times per second. Without this code, GSA can not write to RAM.
        <br /><br />
        xxxx:<br />
        0001 - Executes code handler without
        backing up the $lr register. Must turn GSA off before loading game.<br />
        0002 - Executes code handler and backs up the $lr register. Must turn GSA off before loading game. <br />
        0003 - Replaces a 32-bit pointer used for long-branches. Must turn GSA off before loading game. <br />
        0101 - Executes code handler without backing up the $lr register. <br />
        0102 - Executes code handler and backs up the $lr register. <br />
        0103 - Replaces a 32-bit pointer used for long-branches.
        <br />&nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>Faaaaaaa 0000xxxx</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>ID Code (For Enablers)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        Used by GSA only for auto-detecting the inserted game.
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>xxxxxxxx 001DC0DE</code></td>
    </tr>
    <!--  -->
    <tr>
      <td>DEADFACE(暗号化のシードの変更)</td>
      <td colspan="2" rowspan="2">
        &nbsp;<br />
        DEADFACE改造コードは、暗号化シードを変更するために使用されます。本来の目的はおそらく、誰かが通常シードの暗号を解読した場合に、コードを再暗号化することです。
        &nbsp;<br />&nbsp;
      </td>
    </tr>
    <tr>
      <td><code>DEADFACE 0000xxxx</code></td>
    </tr>
  </tbody>
</table>

## 暗号化


