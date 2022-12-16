# CodeBreaker

## フォーマット

<table>
 <tbody>
  <tr>
   <td>Master Code #1</td>
   <td colspan="2" rowspan="2">
     &nbsp;<br />
     xxxx is the CRC value (the "Game ID" converted to hex)<br/>
     <code>address = ((d << 0x16) + 0x08000100)</code><br />
     <br /><br />
     Flags ("yyyy"):<br />
     0008 - CRC Exists (CRC is used to autodetect the inserted game) <br />
     0002 - Disable Interupts
     &nbsp;<br />&nbsp;
   </td>
  </tr>
  <tr>
   <td>
    <code>0000xxxx yyyy</code>
    <code>1aaaaaaa 0007</code>
   </td>
  </tr>
  <!--  -->
  <tr>
   <td>単純書き込み(8bit)</td>
   <td colspan="2" rowspan="2">
    &nbsp;<br />
    RAMアドレス<code>aaaaaaa</code>に8bit値<code>yy</code>を連続的に書き込みつづけます。
    &nbsp;<br />&nbsp;
   </td>
  </tr>
  <tr>
   <td><code>3aaaaaaa 00yy</code></td>
  </tr>
  <!--  -->
  <tr>
   <td>Slide Code</td>
   <td colspan="2" rowspan="2">
    &nbsp;<br />
    2行で1つの改造コードです。
    アドレス<code>aaaaaaa</code>を始点として、16bit値<code>yyyy</code>を<code>xxxxxxxx</code>回、書き込みます。<br/>
    書き込みのたびにアドレスが<code>iiii</code>だけインクリメントされます。<br/>
    この改造コードは、通常、特定の値でメモリを埋めるために使用されます。
    &nbsp;<br />&nbsp;</td>
  </tr>
  <tr>
   <td>
    <code>4aaaaaaa yyyy</code>
    <code>xxxxxxxx iiii</code>
   </td>
  </tr>
  <!--  -->
  <tr>
   <td>16-Bit Logical AND</td>
   <td colspan="2" rowspan="2">
   &nbsp;<br />
   Performs the AND function on the address provided with the value provided. <br />
   I'm not going to explain what AND does, so if you'd like to know I suggest you see the instruction manual for a graphing calculator. <br />
   This is another advanced code type you'll probably never need to use.
   &nbsp;<br />&nbsp;</td>
  </tr>
  <tr>
   <td><code>6aaaaaaa yyyy</code></td>
  </tr>
  <!--  -->
  <tr>
   <td>条件比較</td>
   <td colspan="2" rowspan="2">
     &nbsp;<br />
     アドレス<code>aaaaaaa</code>のデータと、16bit値<code>yyyy</code>に対して、条件(<code>c</code>)に合致する場合のみ次の行の改造コードを実行します。
     <br />
     <code>c</code>: 7: =、A: ≠、B: >、C: <、 D: and (<code>yyyy</code>が右辺)
     &nbsp;<br />&nbsp;
   </td>
  </tr>
  <tr>
   <td><code>7aaaaaaa yyyy</code></td>
  </tr>
  <!--  -->
  <tr>
   <td>単純書き込み(16bit)</td>
   <td colspan="2" rowspan="2">
    &nbsp;<br />
    RAMアドレス<code>aaaaaaa</code>に16bit値<code>yyyy</code>を連続的に書き込みつづけます。
    &nbsp;<br />&nbsp;
   </td>
  </tr>
  <tr>
   <td><code>8aaaaaaa yyyy</code></td>
  </tr>
  <!--  -->
  <tr>
   <td>Change Encryption Seeds<br />(When 1st Code Only!)</td>
   <td colspan="2" rowspan="2">&nbsp;<br />Works like the DEADFACE on GSA. Changes the encryption seeds used for the rest of the codes. &nbsp;<br />&nbsp;</td>
  </tr>
  <tr>
   <td><code>9yyyyyyy yyyy</code></td>
  </tr>
  <!--  -->
  <tr>
   <td>キー入力判定書き込み</td>
   <td colspan="2" rowspan="2">
    &nbsp;<br />
    <code>yyyy</code>と16bitのキー入力値に対して条件(<code>c</code>)を満たすときに、次の行の改造コードを実行します。<br />
    16bitのキー入力値、<a href="../keypad.md#0x0400_0130---keyinput---キー入力レジスタ-r">KEYINPUT</a>と同じbit配置ですがキーが押されているときに1になります。<br/>
    <code>c</code>: 1: ≠, 2: =
    &nbsp;<br />&nbsp;</td>
  </tr>
  <tr>
   <td><code>D00000c0 yyyy</code></td>
  </tr>
 </tbody>
</table>
