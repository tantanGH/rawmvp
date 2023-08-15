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
 - 16bitステレオPCMの外部同期再生対応(要Raspberry Pi + S44RASP.X)

ADPCM音声データはメインメモリに丸ごと読み込むため、その分のメインメモリは必要です。
16bitPCMデータはハイメモリに読み込むため、ハイメモリ環境が必要です。(X68000Zでは対応していません)

基本的に X68000Z ファームウェアバージョン 1.3.1以降でUSBメモリによる擬似SCSI機能を使っている環境を前提としています。

新規作成したHDS上に動画ファイルを作成し、そのHDSファイルをFAT32フォーマットしたばかりのUSBメモリにコピーしてある前提で、

 - 256x256x65536色x30FPS
 - 384x256x65536色x24FPS

がADPCM音声付きでほぼコマ落ちなく再生できることを確認しています。

なお、試行版とあるように、技術検証が目的であり、厳密なデータフォーマットを定めたりはしていません。


実際の再生例

[<img src='images/rawmvp1.png'/>](https://youtu.be/u_Py9P0lbjs)

---

## インストール

RAWMPxxx.ZIP をダウンロードして、RAWMVP.X をパスの通ったディレクトリにコピーします。

CRTMOD16.X および PCM8A.X を事前に常駐させておく必要があります。

CRTMOD16.X (M.Kamada氏) は XEiJ のインストールパッケージの中の misc ディレクトリに含まれています。

* [https://stdkmd.net/xeij/miscfiles.htm](https://stdkmd.net/xeij/miscfiles.htm)    

`crtmod16 -e` でIOCS _CRTMOD拡張モードで常駐させてください。横384モードを使わない場合は不要です。

PCM8A.X (原作:philly氏) は TcbnErik氏によるパッチ版の使用を推奨します。

* [https://github.com/kg68k/pcm8a](https://github.com/kg68k/pcm8a)

無音モードで使う場合は常駐不要です。

---

## データの準備

データは以下の流れで作成します。画像データのファイル(\*.raw)と、音声データのファイル(\*.pcm)の2つを用意する必要があります。

1. 任意の動画データから、各フレーム画像を連番BMPとして保存する。

BMPは縦幅256以下、横幅256,384,512のいずれか、24bitカラーであることが必要です。

[ffmpeg](https://www.ffmpeg.org/) などで作ることができます。この際にfpsを最終的に出力する厳密なfpsに合わせておく必要があります。

|-|横256/512|横384|
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
出力ファイルサイズが大きくなるので、最終的なコピー先となるHDSドライブに対していきなり書き出してしまうのがお勧めです。

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

---

## 再生方法

    RAWMVP.X - Raw movie player version 0.4.0 by tantan
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
          -x ... load 16bitPCM data to high memory directly
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
元データは必ずこの数値に合わせて作っておく必要があります。MACSDRV(改造版)のようなインテリジェントな自動調整機能はありません。

|指定値|横256/512|横384|
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

なお、再生が間に合わない場合は、デフォルトでは1フレームあたり1フレームまではスキップします。多少の遅れはそれでカバーできますが、それでも間に合わない場合は音と絵がだんだんずれていきます。`-f` オプションをつけるとフレームスキップを行いません。

* raw-movie-file

ベタ画像データファイル名を指定します。ファイルサイズは1.8GB程度までは確認しています。Human68kの扱える最大ファイルサイズは2GBのはずです。

* pcm-file

拡張子.pcmの場合はモノラルADPCM 15625Hzとみなします。

'-' (ハイフン) のみ指定すると無音での再生となります。

拡張子.s22/.s24/.s32/.s44/.s48 の場合はまーきゅりーゆにっと向け16bitステレオPCM(big endian)とみなします。この場合は PCM8A.X ではなく PCM8PP.X を常駐させておく必要があります。さらに丸ごとメモリにロードするだけのハイメモリとハイメモリドライバが必要になります。(X68000Z EAK 1.3.1では現在利用できません) 

`-x` オプションをつけると、16bitPCMデータをハイメモリに直接高速ロードします(メインメモリを経由しない)。TS16FILE.Xなどのドライバが必要になる場合があります。

ファイル名の先頭を `RASP:` から始めた場合、[s44raspd](https://github.com/tantanGH/s44rasp-x68k)と組み合わせることでまーきゅりーゆにっと相当の16bitステレオPCM外部同期再生を行うことができます。この機能は X68000Z EAK 1.3.1 でも利用可能です。

---

## 既知の不具合

* XEiJ + HFS で動作させると音が途切れます。XEiJ + HDS であれば大丈夫です(が、コマ落ちすると思います)
* その他たくさん(ぉ

---

## 変更履歴

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