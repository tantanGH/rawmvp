# RAWMVP.X

Raw Movie File Player for X68000Z/X680x0 (試行版)

---

## About This

ディスク上の無圧縮動画データを逐次読み取りしながら指定した画面モード・fpsで再生します。

 - ディスクから逐次読み取りを行うため再生開始までの待ち時間が短い
 - ディスクから直接GVRAMに転送を行うため高フレームレートが期待できる
 - 256x256x65536色, 384x256x65536色, 512x256x65536色の3種類の画面モードに対応
 - 30FPS, 20FPS, 15FPSの3種類のフレームレートに対応
 - ADPCMによる音声の同期再生対応(要PCM8A.X)
 - 16bitステレオPCMの外部同期再生対応(要Raspberry Pi)

ADPCM音声データはメインメモリに丸ごと読み込むため、その分のメインメモリは必要です。

基本的に X68000Z ファームウェアバージョン 1.3.1以降でUSBメモリによる擬似SCSI機能を使っている環境を前提としています。

試行版とあるように、技術検証が目的であり、厳密なデータフォーマットを定めたりはしていません。


実際の再生例

[<img src='images/rawmvp1.png'/>](https://youtu.be/u_Py9P0lbjs)

---

## インストール

RAWMPxxx.ZIP をダウンロードして、RAWMVP.X をパスの通ったディレクトリにコピーします。

CRTMOD16.X および PCM8A.X を事前に常駐させておく必要があります。

CRTMOD16.X (M.Kamada氏) は XEiJ のインストールパッケージの中の misc ディレクトリに含まれています。

* [https://stdkmd.net/xeij/miscfiles.htm](https://stdkmd.net/xeij/miscfiles.htm)    

`crtmod16 -e` でIOCS _CRTMOD拡張モードで常駐させてください。

PCM8A.X (原作:philly氏) は TcbnErik氏によるパッチ版の使用を推奨します。

* [https://github.com/kg68k/pcm8a](https://github.com/kg68k/pcm8a)

---

## データの準備

データは以下の流れで作成します。画像データのファイル(\*.raw)と、音声データのファイル(\*.pcm)は別のままで、2つのファイルとなります。

1. 任意の動画データから、各フレーム画像を連番BMPとして保存する。

BMPは縦幅256以下、横幅256,384,512のいずれか、24bitカラーであることが必要です。

[ffmpeg](https://www.ffmpeg.org/) などで作ることができます。この際にfpsを最終的に出力する厳密なfpsに合わせておく必要があります。

|-|横256/512|横384|
-|-|-
|30|27.729|28.136|
|20|18.486|18.757|
|15|13.865|14.068|

ffmpegを使った出力例：

    ffmpeg -i hogehoge.mp4 -filter_complex "[0:v] fps=27.729,scale=256:200" -vcodec bmp "output_bmp/output_%05d.bmp"

2. 音声データを48kHz 16bitステレオPCM(big endian)形式(.s48)で抜き出しておく。

こちらも ffmpeg で可能です。ボリュームの調整もできます。

    ffmpeg -i hogehoge.mp4　-f s16be -acodec pcm_s16be -filter:a "volume=0.8" -ar 48000 -ac 2 hogehoge.s48

3. 連番のBMPデータをX680x0 GVRAM形式に変換した上で連結して一つのファイルとする。

横256の画像の場合、2フレームを左右に繋げて512x256の画像にする必要があります。
横384の画像の場合、右側にパディングして横512にする必要があります。

拙作の [BMP2RAW.X](https://github.com/tantanGH/bmp2raw-x68k) や [bmp2raw](https://github.com/tantanGH/bmp2raw) を使うと自動的に作ることができます。BMP2RAW.X は Human68kで動作します。bmp2raw はPythonが使える環境で動作します。

4. 48kHz PCMデータを 15.625kHz ADPCM形式に変換する。

Human68k上で NOZさんの [PCM3PCM](http://noz.ub32.org/68fsw.html)を使うのが手軽です。

    pcm3pcm hogehoge.s48 hogehoge.pcm


---

## 再生方法

    usage: rawmvp <screen-width> <view-height> <fps> <raw-movie-file> <adpcm-file>

5つのコマンドライン引数を指定します。

* screen-width

256, 384 または 512 を指定します。元データの各フレームの横幅と一致している必要があります。

* view-height

表示する画像の高さを1から256で指定します。元データの各フレームの縦幅と一致している必要があります。

* fps

30, 20, 15 のいずれかを指定します。

なお、横256/512の時と横384の時でVSYNC周波数の違いによる厳密なfpsが変わります。
元データは必ずこの数値に合わせて作っておく必要があります。MACSDRV(改造版)のようなインテリジェントな自動調整機能はありません。

|指定値|横256/512|横384|
-|-|-
|30|27.729|28.136|
|20|18.486|18.757|
|15|13.865|14.068|

なお、再生が間に合わない場合は、1フレームあたり1フレームまではスキップします。多少の遅れはそれでカバーできますが、それでも間に合わない場合は音と絵がだんだんずれていきます。

* raw-movie-file

ベタ画像データファイル名を指定します。ファイルサイズは1.8GB程度までは確認しています。Human68kの扱える最大ファイルサイズは2GBのはずです。

* adpcm-file

ADPCMファイル名を指定します。15625Hz前提となります。

ファイル名の先頭を `RASP:` から始めた場合、[s44raspd](https://github.com/tantanGH/s44rasp-x68k)と組み合わせることでまーきゅりーゆにっと相当の16bitステレオPCM外部同期再生を行うことができます。

---

## 変更履歴

* 0.3.0 (2023/08/10) ... s44rasp を使ったリモート16bitステレオPCM同期再生に対応した。
* 0.2.1 (2023/08/09) ... 再生が追いつかない時のフレームスキップに対応した。フレームドロップ率を表示するようにした。
* 0.1.0 (2023/08/07) ... 初版