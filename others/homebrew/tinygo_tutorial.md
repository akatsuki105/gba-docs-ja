# Go言語によるGBAソフト開発のチュートリアル

このページは[Learning Go by examples: part 5 - Create a Game Boy Advance (GBA) game in Go](https://dev.to/aurelievache/learning-go-by-examples-part-5-create-a-game-boy-advance-gba-game-in-go-5944)を翻訳したものです。

## TinyGo

TinyGoは、組み込みシステムや最新のウェブに対応したGoコンパイラです。

BBC micro:bit、Arduino Uno、Nintendo Switch、Game Boy Advanceなど、多くのマイクロコントローラボードでTinyGoプログラムをコンパイルして実行することができます。

TinyGoは、非常にコンパクトなサイズのWebAssembly（WASM）コードを生成することもできます。Webブラウザはもちろん、WASI（WebAssembly System Interface）ファミリーのインターフェースをサポートするサーバーやエッジコンピューティング環境向けのプログラムをコンパイルすることができます。

MacOSの場合次のようにしてtinygoをインストールできます。

```sh
$ brew tap tinygo-org/tools
$ brew install tinygo
$ tinygo version
# tinygo version 0.19.0 darwin/amd64 (using go version go1.16.5 and LLVM version 11.0.0)
```

## 概要

今回作るゲームは次のようなゲームです。(画像をクリックするとYoutubeの紹介動画が開きます)

<a href="https://youtu.be/V-VnFMijaI8">
    <img src="https://res.cloudinary.com/practicaldev/image/fetch/s--FGl7bxCp--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6ltxyl6lato4jckwwpl9.png" width="600" />
</a>

- タイトル画面に`Gopher`と`Press START button`というテキストを映す
- タイトル画面のGopherくんは2匹
- Startボタンを押すとゲーム画面に切り替わり、プレイヤーが操作するGopherくんが現れる
- 十字キーでGopherくんを動かせる
- AボタンでGopherくんがジャンプする
- Selectボタンでタイトル画面に戻る

TinyGoのウェブサイトにあるように、ゲームボーイアドバンスは、ARM7TDMIマイクロコントローラをベースにした携帯型ビデオゲームプラットフォームです。これは有用な情報であり、このページにはGBA用の`machine`パッケージへのリンクがあります。

とはいえ、ドキュメントや最新の動作例がないからといって、私たちを止めることはできません。そこで、コードをお見せして、一歩一歩説明していきます。

このアプリではテキストメッセージを表示するので、`fonts`フォルダを作成し、使用したい Go のフォントを入れます。

```
fonts
├── freesansbold24pt7b.go
└── gophers58pt.go
```

実際のコード例は[Githubレポジトリ](https://github.com/scraly/learning-go-by-examples/tree/main/go-gopher-gba)を見てください

## 始めよう

この前提条件の後に、アプリのコードを作成します。そのために、`gopher.go`ファイルを作成して、そこに以下のコードをコピー＆ペーストしていきます。

まず、Goのコードはパッケージにまとめられています。そこで、`main`と呼ばれるパッケージと、`main`ファイルでインポートして使用する必要のあるすべての依存関係を初期化します。

```go
package main

import (
    "image/color"
    "machine"
    "runtime/interrupt"
    "runtime/volatile"
    "unsafe"

    "github.com/scraly/learning-go-by-examples/go-gopher-gba/fonts"
    "tinygo.org/x/tinydraw"
    "tinygo.org/x/tinyfont"
)
```

次に変数を初期化します。

```go
var (
    //KeyCodes / Buttons
    keyDOWN      = uint16(895)
    keyUP        = uint16(959)
    keyLEFT      = uint16(991)
    keyRIGHT     = uint16(1007)
    keyLSHOULDER = uint16(511)
    keyRSHOULDER = uint16(767)
    keyA         = uint16(1022)
    keyB         = uint16(1021)
    keySTART     = uint16(1015)
    keySELECT    = uint16(1019)

    // Register display
    regDISPSTAT = (*volatile.Register16)(unsafe.Pointer(uintptr(0x4000004)))

    // Register keypad
    regKEYPAD = (*volatile.Register16)(unsafe.Pointer(uintptr(0x04000130)))

    // Display from machine
    display = machine.Display

    // Screen resolution
    screenWidth, screenHeight = display.Size()

    // Colors
    black = color.RGBA{}
    white = color.RGBA{255, 255, 255, 255}
    green = color.RGBA{0, 255, 0, 255}
    red   = color.RGBA{255, 0, 0, 255}

    // Google colors
    gBlue   = color.RGBA{66, 163, 244, 255}
    gRed    = color.RGBA{219, 68, 55, 255}
    gYellow = color.RGBA{244, 160, 0, 255}
    gGreen  = color.RGBA{15, 157, 88, 255}

    // Coordinates
    x int16 = 100 //TODO: horizontally center
    y int16 = 100 //TODO: vertically center
)
```

ご覧のように、コードが複雑にならないように、RGBのカラーコードをハードコードしたくないので、コードで使用するいくつかの変数を定義しています。

ここからは`main()`を定義していきます。`main()`の内容は

- レジスタと画面の設定
- テキストとGopherくんを描画する関数`drawGophers()`を呼び出す
- VBlankが来たら`update()`関数を呼び出すようにハンドラを登録
- 無限ループを起こして`main()`が終了しないようにする

```go
func main() {
    // Set up the display
    display.Configure()

    // Register display status
    regDISPSTAT.SetBits(1<<3 | 1<<4)

    // Display Gopher text message and draw our Gophers
    drawGophers()

    // Creates an interrupt that will call the "update" fonction below, hardware way to display things on the screen
    interrupt.New(machine.IRQ_VBLANK, update).Enable()

    // Infinite loop to avoid exiting the application
    for {
    }
}
```

`drawGophers()`は

```go
func drawGophers() {

    // Display a textual message "Gopher" with Google colors
    tinyfont.DrawChar(display, &fonts.Bold24pt7b, 36, 60, 'G', gBlue)
    tinyfont.DrawChar(display, &fonts.Bold24pt7b, 71, 60, 'o', gRed)
    tinyfont.DrawChar(display, &fonts.Bold24pt7b, 98, 60, 'p', gYellow)
    tinyfont.DrawChar(display, &fonts.Bold24pt7b, 126, 60, 'h', gGreen)
    tinyfont.DrawChar(display, &fonts.Bold24pt7b, 154, 60, 'e', gBlue)
    tinyfont.DrawChar(display, &fonts.Bold24pt7b, 180, 60, 'r', gRed)

    // Display a "press START button" message - center
    tinyfont.WriteLine(display, &tinyfont.TomThumb, 85, 90, "Press START button", white)

    // Display two gophers
    tinyfont.DrawChar(display, &fonts.Regular58pt, 5, 140, 'B', green)
    tinyfont.DrawChar(display, &fonts.Regular58pt, 195, 140, 'X', red)
}
```

テキストメッセージを表示したいので、`tinyfont`パッケージを使い、`fonts`フォルダに入れたフォントを使用しました。

Gopherくんも表示したいですが、今のところ、画像を表示したり、スプライトを読み込んだりすることができないので、Gopherくんを表示する方法を見つけなければなりませんでした。

その問題は一旦置いておき、キー入力に反応できるように`update`関数を次のように変更します。

```go
func update(interrupt.Interrupt) {

    // Read uint16 from register regKEYPAD that represents the state of current buttons pressed
    // and compares it against the defined values for each button on the Gameboy Advance
    switch keyValue := regKEYPAD.Get(); keyValue {
    // Start the "game"
    case keySTART:
        // Clear display
        clearScreen()
        // Display gopher
        tinyfont.DrawChar(display, &fonts.Regular58pt, x, y, 'B', green)
    // Go back to Menu
    case keySELECT:
        clearScreen()
        drawGophers()
    // Gopher go to the right
    case keyRIGHT:
        // Clear display
        clearScreen()
        x = x + 10
        // display gopher at right
        tinyfont.DrawChar(display, &fonts.Regular58pt, x, y, 'B', green)
    // Gopher go to the left
    case keyLEFT:
        // Clear display
        clearScreen()
        x = x - 10
        // display gopher at right
        tinyfont.DrawChar(display, &fonts.Regular58pt, x, y, 'B', green)
    // Gopher go to the down
    case keyDOWN:
        // Clear display
        clearScreen()
        y = y + 10
        tinyfont.DrawChar(display, &fonts.Regular58pt, x, y, 'B', green)
    // Gopher go to the up
    case keyUP:
        // Clear display
        clearScreen()
        y = y - 10
        tinyfont.DrawChar(display, &fonts.Regular58pt, x, y, 'B', green)
    // Gopher jump
    case keyA:
        // Clear display
        clearScreen()
        // Display the gopher up
        y = y - 20
        tinyfont.DrawChar(display, &fonts.Regular58pt, x, y, 'B', green)
        // Clear the display
        clearScreen()
        // Display the gopher down
        y = y + 20
        tinyfont.DrawChar(display, &fonts.Regular58pt, x, y, 'B', green)
    }
}
```

## 依存パッケージのインストール

さて、まだ依存パッケージのinstallをしていなかったので、ここでしましょう。

```
$ go get tinygo.org/x/tinydraw
$ go get tinygo.org/x/tinyfont
```

### TinyFont

TinyGoは、マイコンや軽量ハードウェア用のGoアプリを構築するための独立した方法ではありませんが、メインのGitHubリポジトリには、TinyFontのようないくつかの便利なツールも含まれています。

![](https://res.cloudinary.com/practicaldev/image/fetch/s--69fHK7cO--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mq0cnpwq9famofen5azh.png)

TinyFontは、TinyGoディスプレイ用の「フォント/テキスト」パッケージです。

TinyFontのリポジトリには、アプリで使用できるいくつかのフォントが含まれており、このようにインポートすることができます。

```go
import (
    "tinygo.org/x/tinyfont/freemono"
    "tinygo.org/x/tinyfont/freesans"
    "tinygo.org/x/tinyfont/freeserif"
)
```

個人的なフォントを使用したい場合は、ツール`tinyfontgen`で次のように、BDF形式から TinyGoで使える形式に変換することができます。

```
$ tinyfontgen --package my-font --fontname MyFontRegular12pt MyFont-Regular-12pt.bdf --output MyFont-Regular-12pt.go --all
```

### TinyDraw

TinyDrawは、幾何学的な図形を描くことができる便利なツールです。

![](https://res.cloudinary.com/practicaldev/image/fetch/s--mC9dRZlW--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mgutmx73jinoe24yngd5.png)

TinyDrawを使うと次のように簡単に幾何学模様を描くことができます。

```go
    white = color.RGBA{255, 255, 255, 255}
    green = color.RGBA{0, 255, 0, 255}
    red   = color.RGBA{255, 0, 0, 255}

    // ...

    tinydraw.Line(&display, 100, 100, 40, 100, red)

    tinydraw.Rectangle(&display, 30, 106, 120, 20, white)
    tinydraw.FilledRectangle(&display, 34, 110, 112, 12, green)

    tinydraw.Circle(&display, 120, 30, 20, white)
    tinydraw.FilledCircle(&display, 120, 30, 16, red)

    tinydraw.Triangle(&display, 120, 102, 100, 80, 152, 46, white)
    tinydraw.FilledTriangle(&display, 120, 98, 104, 80, 144, 54, green)
```

### Gopher font

先程、GBAのハードウェアではTinyFontで画像やスプライトを表示することができないので、私たちの好きなGopherを表示する方法を見つける必要がありますと述べました。

なんとか方法を探していたところ、Jaana Doganが作成した素晴らしい既存の[Gopher font](http://2ttf.com/HCQ3PvcaQ4U)を使用することでこの問題が解決しました。

![](https://res.cloudinary.com/practicaldev/image/fetch/s--_llBITGf--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e03a21krkk59klhrxnx6.png)


## 動かしてみよう

```
$ tinygo run -target=gameboy-advance gopher.go
```