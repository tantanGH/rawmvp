# RAWMVP.X

Raw Movie File Player for X68000Z/X680x0 (試行版)

---

## About This

ディスク上の無圧縮動画データを逐次読み取りしながら指定した画面モード・fpsで再生します。

 - ディスクから逐次読み取りを行うため再生開始までの待ち時間が短い
 - ディスクから直接GVRAMに転送を行うため高フレームレートが期待できる
 - 256x256x65536色, 384x256x65536色, 512x256x65536色の3種類の画面モードに対応
 - ADPCMによる音声の同期再生対応(要PCM8A.X)
 - 16bitステレオPCMの内部同期再生対応(要Mercury-UNIT + PCM8PP.X)
 - 16bitステレオPCMの外部同期再生対応(要Raspberry Pi + s44raspd)

基本的に X68000Z ファームウェアバージョン 1.3.1以降でUSBメモリによる擬似SCSI機能を使っている環境を推奨環境としています。

新規作成したHDS上に動画ファイルを作成し、そのHDSファイルをFAT32フォーマットしたばかりのUSBメモリにコピーしてある前提で、

 - 256x256x65536色x30FPS
 - 384x256x65536色x24FPS

がADPCM音声付きでほぼコマ落ちなく再生できることを確認しています。

実際の再生例(X68000Z EAK)

[<img src='images/rawmvp1.png'/>](https://youtu.be/u_Py9P0lbjs)

---

## 必要なメモリ搭載量

ADPCM音声データはメインメモリに丸ごと読み込むため、その分のメインメモリは必要です。X68000Zでは12MB標準なのでRAMDISKを大きく確保していない限りは特に気にする必要はありません。

16bitPCMデータをPCM8PP.Xを使って再生する場合はPCMデータをすべてハイメモリに読み込むため、ハイメモリ環境が必要です。X68000Zでは対応していません。

---

## RAWMVPのインストール

RAWMPxxx.ZIP をダウンロードして、RAWMVP.X をパスの通ったディレクトリにコピーします。

---

## PCM8Aのインストール

ADPCM音声付きでの再生には PCM8A.X (原作:philly氏) が必要になります。

TcbnErik氏によるパッチ版の使用を推奨します。

* [https://github.com/kg68k/pcm8a](https://github.com/kg68k/pcm8a)


常駐の際の推奨コマンドライン:

    pcm8a -v0 -s1 off

無音モードで使う場合は不要です。

---

## CRTMOD16のインストール

横384の正方形ドットモードを使う場合は、CRTMOD16.X (M.Kamada氏) が必要になります。

CRTMOD16.X は XEiJ のインストールパッケージの中の misc ディレクトリに含まれています。

* [https://stdkmd.net/xeij/miscfiles.htm](https://stdkmd.net/xeij/miscfiles.htm)    

常駐の際の推奨コマンドライン:

    crtmod16 -e

上記の通りIOCS _CRTMOD拡張モードで常駐させてください。横384モードを使わない場合は不要です。

---

## XEiJ + HFS での注意点

XEiJはこれを書いている時点での最新リリースバージョン(0.23.08.08)までの場合、ADPCM再生中にHFSからGVRAMにデータ読み込みを行うとADPCM音声が乱れます。

最新のテストバージョン(0.23.08.23)を利用することでこれを回避でき、GVRAMダイレクトロードによる低負荷での高フレームレート再生が可能です。

XEiJ 0.23.08.23 TEST
* [https://stdkmd.net/xeijtest/](https://stdkmd.net/xeijtest/)

XEiJ + HDSの再生は推奨しません。

---

## 動作確認環境

こちらで確認した環境は以下の通りです。

|機種|サイズ・FPS|音声|結果|
-|-|-|-
|X68000Z EAK 1.3.1 10MHz|256x200 30fps|PCM8A.X(ADPCM)|ほぼ問題ないが、わずかに音声が途切れることがある。|
|X68000Z EAK 1.3.1 10MHz|384x200 24fps|PCM8A.X(ADPCM)|ほぼ問題ないが、わずかに音声が途切れることがある。|
|X68030実機 + 060turbo 50MHz + ハイメモリRAMDISK|256x200 30fps|PCM8A.X(ADPCM)|問題なし|
|X68030実機 + 060turbo 50MHz + ハイメモリRAMDISK|256x200 30fps|PCM8PP.X(S32)|問題なし|
|X68030実機 + 060turbo 50MHz + ハイメモリRAMDISK|256x200 30fps|PCM8PP.X(S48)|音声にノイズが乗る|
|X68000XVI実機 16.7MHz + Nereid バンクメモリRAMDISK(16MB)|256x192 15fps|PCM8A.X(ADPCM)|問題なし|
|XEiJ 0.23.08.23 テスト版 + HFS + XVI 10MHzモード|256x200 30fps|PCM8A.X(ADPCM)|問題なし|
|XM6g 3.36 + WindrvXM + X68030 25MHz|256x200 30fps|PCM8PP.X(S48)|問題なし|
|XM6g 3.36 + WindrvXM + X68030 25MHz|384x192 24fps|PCM8PP.X(S48)|問題なし|


---

## データ作成のためのツールのインストール

RAWMVPで再生するデータはご自身で用意して頂く必要があります。
以下のツールを導入しておくことをお勧めします。

* [BMP2RAW.X](https://github.com/tantanGH/bmp2raw-x68k)
* [LARGECP.X](https://github.com/tantanGH/largecp)
* [PCM3PCM.X](http://noz.ub32.org/68fsw.html)

---

## データの準備

データは以下の流れで作成します。画像データのファイル(\*.raw)と、音声データのファイル(\*.pcm)の2つを用意する必要があります。

1. Windows/macOSなどを使い、任意の動画データから各フレーム画像を連番BMPとして保存する。

BMPは縦幅256以下、横幅256,384,512のいずれか、24bitカラーであることが必要です。

連番BMPは [ffmpeg](https://www.ffmpeg.org/) などで作ることができます。ffmpegは出力fpsを自由に決めることが可能です。

後述する BMP2RAW.X でもfps変換は可能です。その場合でも連番BMPが何fpsで出力されたものであるかは知っておく必要があります。

|指定FPS|出力FPS 横256/512|出力FPS 横384|
-|-|-
|30|27.729|28.136|
|24|22.183|22.509|
|20|18.486|18.757|
|15|13.865|14.068|
|12|11.092|11.254|
|10|9.243|9.379|
|6|5.546|5.627|
|5|4.622|4.689|
|4|3.697|3.751|
|3|2.723|2.814|
|2|1.849|1.876|

ffmpegを使った出力例：

    ffmpeg -i hogehoge.mp4 -filter_complex "[0:v] fps=27.729,scale=256:200" -vcodec bmp "output_bmp/output_%05d.bmp"

2. 音声データを48kHz 16bitステレオPCM(big endian)形式(.s48)で抜き出しておく。

こちらも ffmpeg で可能です。ボリュームの調整もできます。

    ffmpeg -i hogehoge.mp4　-f s16be -acodec pcm_s16be -filter:a "volume=0.8" -ar 48000 -ac 2 hogehoge.s48

3. 連番のBMPデータをX680x0 GVRAM形式に変換した上で連結して一つのファイルとする。

横256の画像の場合、2フレームを左右に繋げて512x256の画像にする必要があります。
横384の画像の場合、右側にパディングして横512にする必要があります。

Human68k上で拙作の [BMP2RAW.X](https://github.com/tantanGH/bmp2raw-x68k) を使うと自動的に作ることができます。

4. 48kHz PCMデータを 15.625kHz ADPCM形式に変換する。

Human68k上で NOZさんの [PCM3PCM.X](http://noz.ub32.org/68fsw.html) を使うのが手軽です。

    pcm3pcm hogehoge.s48 hogehoge.pcm

5. (オプション)マニフェストとなるRMVファイルを作成しておく。

RAWMVPで再生しやすいように、以下の5行からなるテキストファイルを.RMVとして作成しておくことをお勧めします。

    画像横幅
    画像縦幅
    fps
    RAW動画データファイル名
    PCMデータファイル名

ファイル名がフルパスで無い場合は、RMVファイルが存在するディレクトリからの相対パスとみなされます。

PCMデータファイル名を '-' (ハイフン) とすると音声なしでの再生となります。

大きなファイルのコピーには拙作の[LARGECP.X](https://github.com/tantanGH/largecp)を使うのをお勧めします。
標準のCOPYコマンドより遥かに速い上、進捗表示や途中キャンセルも可能です。

---

## 再生方法

    RAWMVP.X - Raw movie player version 0.x.x by tantan
    usage: rawmvp [options] <screen-width> <view-height> <fps> <raw-movie-file> <pcm-file>
      screen-width ... 256, 384, 512
      view-height ... 1 - 256
              fps ... 30, 24, 20, 15, 12, 10, 6, 5, 4, 3, 2
          raw-file ... raw movie file (.raw)
          pcm-file ... pcm file (.pcm|.s22|.s24|.s32|.s44|.s48)

    usage: rawmvp [options] <rmv-file>
          rmv-file ... 5-line text manifest file
                        screen-width
                        view-height
                        fps
                        raw-movie-file
                        pcm-file

    options:
          -l[n] ... repeat n times (0:endless, 1:default)
          -f ... do not skip frames
          -m ... do not load image data to GVRAM directly
          -x ... load 16bit PCM data to high memory directly
          -h ... show help message

再生方法は2通りあり、

* 5つのコマンドラインパラメータを指定する
* 1つのRMVファイルを指定する

のいずれかになります。RMVファイルについては前のセクションを参照してください。

`-l` オプションでリピート再生を行うことができます。

* screen-width

256, 384 または 512 を指定します。元データの各フレームの横幅と一致している必要があります。

* view-height

表示する画像の高さを1から256で指定します。元データの各フレームの縦幅と一致している必要があります。
16:9ソースの場合は、上下に黒帯部分を含めて縦256のデータを作るより、実画像部分だけで縦200程度のデータにした方がデータサイズの点でもフレームレートの点でもお勧めです。

* fps

30, 24, 20, 15, 12, 10, 6, 5, 4, 3, 2 のいずれかを指定します。

なお、横256/512の時と横384の時でVSYNC周波数の違いによる厳密なfpsが変わります。
詳細な数値は前のセクションを参照してください。MACSDRV(改造版)のようなインテリジェントな自動調整機能はありません。

なお、再生が間に合わない場合は、デフォルトでは1フレームあたり1フレームまではスキップします。多少の遅れはそれでカバーできますが、それでも間に合わない場合は音と絵がだんだんずれていきます。`-f` オプションをつけるとフレームスキップを行いません。

* raw-movie-file

ベタ画像データファイル名を指定します。ファイルサイズは1.8GB程度までは確認しています。Human68kの扱える最大ファイルサイズは2GBのはずです。

* pcm-file

拡張子.pcmの場合はモノラルADPCM 15625Hzとみなします。

'-' (ハイフン) のみ指定すると無音での再生となります。

拡張子.s22/.s24/.s32/.s44/.s48 の場合はまーきゅりーゆにっと向け16bitステレオPCM(big endian)とみなします。この場合は PCM8A.X ではなく PCM8PP.X を常駐させておく必要があります。さらに丸ごとメモリにロードするだけのハイメモリとハイメモリドライバが必要になります。(X68000Z EAK 1.3.1では現在利用できません) 

`-m` オプションをつけるとディスクからGVRAMへのダイレクトロードを行う代わりに、メインメモリ(ハイメモリがあればハイメモリ)を経由します。XEiJでHFS上のファイルを再生する際には必ず指定してください。指定しないとADPCM音声が途切れます。

`-x` オプションをつけると、16bitPCMデータをハイメモリに直接高速ロードします(メインメモリを経由しない)。TS16FILE.Xなどのドライバが必要になる場合があります。

ファイル名の先頭を `RASP:` から始めた場合、[s44raspd](https://github.com/tantanGH/s44rasp-x68k)と組み合わせることでまーきゅりーゆにっと相当の16bitステレオPCM外部同期再生を行うことができます。この機能は X68000Z EAK 1.3.1 でも利用可能です。

---

## 変更履歴

* 0.6.0 (2023/08/24) ... メモリ経由転送部分をアセンブラで書き直して高速化した。(XM6g+PCM8PP時のみ自動有効)
* 0.5.9 (2023/08/23) ... XEiJでのメモリ転送自動有効を行わないようにした。
* 0.5.8 (2023/08/22) ... -xの挙動を元に戻した。XEiJを自動認識してメモリ転送を自動有効にするようにした。
* 0.5.7 (2023/08/20) ... XEiJでHFSから再生するとADPCMが途切れてしまうのを防ぐ-mオプション追加
* 0.5.6 (2023/08/20) ... -xの意味を逆にした。16bitPCM音声データの周波数がPCM8PP再生周波数より高いと正常に再生できていなかったのを修正。
* 0.5.5 (2023/08/17) ... 0.5.3でADPCMの途切れが目立っていたデグレを修正。テキストVRAMモード削除
* 0.5.3 (2023/08/17) ... テキストプレーン初期化ではなく単に非表示にするようにした
* 0.5.2 (2023/08/16) ... 起動時にテキストプレーン2,3を初期化するようにした
* 0.5.0 (2023/08/16) ... テキストVRAMを使う512x512x16, 768x512x16モードに実験的に対応した
* 0.4.5 (2023/08/15) ... 24fpsに対応した
* 0.4.4 (2023/08/14) ... 環境によってループオプションが有効にならなかった不具合修正
* 0.4.3 (2023/08/13) ... ADPCMデータはハイメモリが使える状態でも必ずメインメモリに読み込むようにした
* 0.4.2 (2023/08/13) ... pcm8pp 使用時の不具合修正
* 0.4.1 (2023/08/13) ... リピート再生に対応した。
* 0.4.0 (2023/08/13) ... RMVファイルに対応した。処理落ちが大きい時の挙動とドロップ率表示を改善した。PCM8PPに対応した。
* 0.3.1 (2023/08/10) ... crtmod16.x の常駐チェックを横幅384の時だけにした。
* 0.3.0 (2023/08/10) ... s44rasp を使ったリモート16bitステレオPCM同期再生に対応した。
* 0.2.1 (2023/08/09) ... 再生が追いつかない時のフレームスキップに対応した。フレームドロップ率を表示するようにした。
* 0.1.0 (2023/08/07) ... 初版