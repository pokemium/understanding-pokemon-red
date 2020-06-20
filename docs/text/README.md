# テキスト

## 文字コードについて

[文字コード](charcode.md)を参照

## テキストデータに関連するマクロ

macros/text_macros.asmで定義されている

 マクロ  |  役割
---- | ----
 text ●● |  ここからテキストの描画を開始し●●というテキストを表示
 next ●●  |  次の行に●●というテキストを表示(図鑑などで使われる。改行文字みたいなもの) 
 line ●●  |  テキストボックスの2行目に●●というテキストを配置(テキストボックス用の改行文字みたいなもの)
 para ●●  |  次のパラグラフ(スクロールではなく新しいテキストボックス)を開始し、●●というテキストを表示
 cont ●●  |  次の行にテキストボックスをスクロールさせ、●●というテキストを表示
 done ●●  |  ●●というテキストでテキストボックスを終了させる。(イベントなし)
 prompt ●●  |  ●●というテキストでテキストボックスを終了させる。(この後ほかのイベントが開始する)

## テキストデータの解釈

 例えば次のテキストデータは次のように解釈できる

```asm
_PalletTownText5::
	text "PALLET TOWN"	
	line "Shades of your"
	cont "journey await!"
	done
```

1. テキストボックスが開いて『PALLET TOWN』を1行目に配置
2. テキストボックスの2行目に『Shades of your』を配置
3. テキストボックスの3行目(Aボタンを押すとテキストボックスが下にスクロール)に『journey await!』を配置
4. テキスト終了

## テキストの描画

[テキストの描画](./text_render.md)参照

## テキストコマンドの処理

テキストデータは、画面に描画する用途以外にも独自の内部コマンドとして使われる用途も持っている。

`TextCommandProcessor`でbcレジスタの示すアドレスにある文字列をさながらスクリプトのように解釈する。

このテキストコマンドによってプロンプト(▼)の点滅や、テキストボックスのスクロールなどの処理を呼び出せたりする。
