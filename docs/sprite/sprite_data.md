# スプライトデータ

スプライトの保持しているデータについての詳細なドキュメント

## 概要

現在のマップ上に存在するスプライトのデータは[wram.asm](../../wram.asm)の`wSpriteDataStart`で保持されている

16スプライト分の大きさのデータ領域が2種類存在している

どちらのデータ領域も1つのスプライトごとに16バイトの大きさを持っている

(つまり16 * 16 * 2バイトの領域が全体で確保されている)

## wSpriteStateData1(1つ目のデータ領域)

1つ目のデータ領域は以下のような構造を取っている

 アドレス  | ラベル |  内容
---- | ---- | ----
 $C1x0  | picture ID  |  スプライトID、spriteIDとも呼ばれる。 <br/>`InitMapSprites` などでスプライトの識別に用いる
 $C1x1  | movement status  |  スプライトの状態<br/>0: 未初期化, 1: 準備完了, 2: クールタイム中, 3: 移動中<br/>またプレイヤーのほうを見ているときは7bit目が立つ
 $C1x2  | sprite image index  |  スプライトが画面上にどのように表示されるか  <br/>\$ff -> スプライト非表示 <br/>スプライトの方向や歩きモーションの進行具合、VRAMオフセットによって値が変わる。<br/>このおかげで、VRAMのどのタイルデータを使うか、反転させるかなどがわかり、結果としてスプライトが画面上にどのように表示されるかがわかる
 $C1x3  | Y screen position delta  |  スプライトのY座標変化 <br/>-1/0/1のどれか スプライトの更新時にC1x4に加算される
 $C1x4  | Y screen position  |  スプライトのY座標 <br/>ピクセル単位 画面内での位置 常にグリッド(16*16)の4ピクセル上にあるため、スプライトはタイルの中央に表示される 立体的に見せるため
 $C1x5  | X screen position delta  |  スプライトのX座標変化 <br/>-1/0/1のどれか スプライトの更新時にC1x6に加算される
 $C1x6  | X screen position  |  スプライトのX座標 <br/>ピクセル単位 画面内での位置 移動中でないならグリッド(16*16)にぴったりおさまる
 $C1x7  | intra-animation-frame counter  |  0から4までのアニメーションフレームカウンタ<br/>4になるとc1x8がインクリメントされる 歩きモーションなどのアニメーションのフレームカウントに利用
 $C1x8  | animation frame counter  |  0から3までのカウンタ <br/>歩きモーションなどのアニメーションの状態を表すのに利用 つまり歩きモーションには16フレームかかる 
 $C1x9  | facing direction  |  スプライトの方向 <br/>0: 下, 4: 上, 8: 左, $c: 右
 $C1xa  | undefined  |  ???
 $C1xb  | undefined  |  ???
 $C1xc  | undefined  |  ???
 $C1xd  | undefined  |  ???
 $C1xe  | undefined  |  ???
 $C1xf  | undefined  |  ???

#### sprite image index(\$C1x2)

sprite image index によって、人などのスプライトがどのように画面上に表示されるかがわかる  

sprite image indexは 上位ニブル(XXXX0000) と 下位ニブル(0000YYYY)の 2パートに分かれている

上位ニブルは スプライトのVRAM内でのオフセット(`VRAMオフセット`)を指し `[C2xe] - 1`で表される

主人公は\[C2xe\]の値が1なので上位ニブルは 0 になる

下位ニブルは次の式で算出される [参考: .calcImageIndex](./../../engine/overworld/movement.asm)

```
[$C1x2]の下位ニブル = [$C1x8] + [$C1x9] = animation_frame_counter(0~3) + facing_direction(00, 04, 08, 0c)
```

つまり \$C1x2は

```
[$C1x2] = (VRAMオフセット * 0x10) + animation_frame_counter(0~3) + facing_direction(00, 04, 08, 0c)
```

#### animation frame counter(\$C1x7,\$C1x8)

アニメーションフレームカウンタが\$C1x7と\$C1x8の2つの領域に分かれているのは、アニメーション自体には16フレームかかるが、取りうるアニメーションの画像は4パターンしかないので、アニメーションフレームを階層構造を持たせてカウントするためだと考えられる

つまり\$C1x8を見ればどのアニメーション画像を使えばいいかがわかり、\$C1x7と\$C1x8の両方を合わせることでアニメーション16フレームのうち何フレーム目かがわかる

## wSpriteStateData2(2つ目のデータ領域)

2つ目のデータ領域は以下のような構造を取っている

 アドレス  | ラベル |  内容
---- | ---- | ----
 $C2x0  | walk animation counter  |  歩きモーションのアニメーションカウンタ <br/>$10から移動した分だけ減っていく
 $C2x1  | ???  |  用途不明
 $C2x2  | Y displacement  |  8で初期化 スプライトが初期座標から離れすぎないために設定されていると考えられるがバグがある
 $C2x3  | X displacement  |  8で初期化 スプライトが初期座標から離れすぎないために設定されていると考えられるがバグがある
 $C2x4  | Y position  |  マップ上での Y 座標 <br/>16\*16のマスのどこにいるかを表している <br/>一番上のマスにいるときは4となるようになっている <br/>例: 一番上のマスから1マス下にいるときは5になる
 $C2x5  | X position  |  マップ上での X 座標 <br/>16\*16のマスのどこにいるかを表している <br/>一番左のマスにいるときは4となるようになっている
 $C2x6  | movement byte 1  |  スプライトの動きを決めるデータその1 [movement byte](./update.md#movement-byte-1)参照
 $C2x7  | ???  |  草むらにスプライトがいるとき$80になってそれ以外では$0になっている<br/>おそらくスプライトの上に草むらを描画するのに利用
 $C2x8  | delay until next movement  |  次の動きまでのクールタイム <br/>どんどん減って行って, 0になるとC1x1が1にセットされる
 $C2x9  | undefined  |  ???
 $C2xa  | undefined  |  ???
 $C2xb  | undefined  |  ???
 $C2xc  | undefined  |  ???
 $C2xd  | undefined  |  ???
 $C2xe  | sprite image base offset  |  スプライトの2bppタイルデータがVRAMのどこにあるかを示すオフセット(VRAMオフセット) <br/>プレイヤーは常に1となる <br/>\$C1x2の計算に利用される
 $C2xf  | undefined  |  ???


#### VRAMオフセット

0xC2Xe の領域にはスプライトの VRAMオフセットが格納される

スプライトの2bppタイルデータは[スプライト > VRAM 上のタイルデータ](./README.md) のように格納される

3面スプライト(上下左)は合計12タイル(1面が2*2なので)使い、VRAMタイルデータ領域の先頭から配置される。

1面スプライトは合計4タイル使い、VRAMタイルデータ領域の後ろから配置される(0x8780-0x87c0, 0x87c0-8800)

VRAMオフセットがわかれば、スプライトのタイルデータがVRAM上にこのように配置されているとき、どこのアドレスに配置されているかがわかる

## wMapSpriteData

32バイトの領域。

各スプライトごとに2バイトずつ、$C1X0, $C2X0 のスプライト16個に対応している

各スプライトのエントリは [movement byte 2, テキストID] を表している  

#### movement byte 2

[movement byte 2](./update.md#movement-byte-2) 参照

## ロード処理

スプライトのロード処理は `LoadMapHeader` で行われる