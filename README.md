# RAWMVP.X

Raw Movie File Player for X68000Z/X680x0

---

## About This

ディスク上の無圧縮動画データを逐次読み取りしながら指定した画面モード・fpsで再生します。

 - ディスクから逐次読み取りを行うため再生開始までの待ち時間が短い
 - 256x256x65536色モード及び384x256x65536色モードの2種類の画面モードに対応
 - 30FPS,20FPS,15FPSの3種類のフレームレートに対応
 - ADPCMによる音声の同期再生対応(要PCM8A.X)

ADPCM音声データはメインメモリに丸ごと読み込むため、その分のメインメモリは必要です。
基本的に X68000Z ファームウェアバージョン 1.3.1以降 で PSEUDO SCSI 機能を使っている環境を前提としています。

---

## How to Install

RAWMPxxx.ZIP をダウンロードして、RAWMVP.X をパスの通ったディレクトリにコピーします。

CRTMOD16.X および PCM8A.X を事前に常駐させておく必要があります。

    `crtmod16 -e`
    `pcm8a`

CRTMOD16.X (M.Kamada氏) は XEiJ のインストールパッケージの中の misc ディレクトリに含まれています。

    https://stdkmd.net/xeij/miscfiles.htm

PCM8A.X (原作:philly氏) は TcbnErik氏によるパッチ版の使用を推奨します。

    https://github.com/kg68k/pcm8a

---

## Usage

    usage: rawmvp <screen-width> <view-height> <fps> <raw-movie-file> <adpcm-file>

5つのコマンドライン引数を指定します。

screen-width

    256 または 384 を指定します。

view-height

    表示する画像の高さを指定します。(1-256)

fps

    30, 20, 15 のいずれかを指定します。

    横256の時と横384の時でVSYNC周波数の違いによる厳密なfpsが変わります。元データは必ずこの数値に合わせて作っておく必要があります。自動調整機能はありませ

|指定値|横256|横384|
-|-|-
|30|27.729|28.137|
|20|18.486|18.758|
|15|13.865|14.068|

raw-movie-file


---

## SCSIディスクから再生を行う場合

SCSIディスクからDMAの届かないハイメモリへダイレクトにデータ転送を行う必要があるため、TS16FILE.X が必須になります。
VDISK(PhantomX), HFS(XEiJ) の場合は必要ありません。Windrvは環境が無いので未確認です。

また、TS16FILE.Xの後にSCSI関連のデバイスドライバを組み込まないようにしてください(SUSIE.Xなど)。

---
---

## History

* 0.1.0 (2023/08/07) ... 初版