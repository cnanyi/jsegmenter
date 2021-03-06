iOS用のセグメント分割プログラム

mp3やmpegts(h.264 + AAC もしくは h.264 + mp3)のデータを分割して、VideoOnDemandやLiveStreamingできるようにするためのプログラム
メディアファイルの生成は自動ではないので、ffmpegあたりをつかって生成してください。

オプション説明
 -duration <arg> 各セグメントファイルの長さ(秒)指定
  これより長い切断できる部分で、切断を実施します。
 -help ヘルプを表示します。
 -http <arg> m3u8ファイルに書き込むhttpのprefixデータ
  無い場合は単にファイル名の出力になります。
 -input <arg> 入力ファイル
  無い場合は標準入力が入力候補になります。(ffmpegの出力をパイプでつなぐとき等に利用できます。)
 -limit <arg> m3u8上の保持ファイル数指定
  無い場合は全ファイル書き込みます。
 -live -limit 3 -strictと同じ指定になります。
  この状態のストリームはデバイス上でシークができなくなります。
 -m3u8 <arg> m3u8ファイルを指定します。
  省いた場合には、メディアデータの分割のみ実施します。
 -prefix <arg> 各ファイルの出力prefixを指定します。相対パスでも絶対パスでもOKです。
  省いた場合は"file"になります。
 -strict このオプションが入っている場合は、m3u8の指定から外れたファイルは削除されます。

ライセンス
 ライセンスはlgplとしておきます。

つくった動機
 appleのHttpLiveStreamingはwowzaやFlashMediaServerが対応してるし、red5でもプラグインが(不完全ながら)あるし、nginxのrtmpModuleでもサポートされています。
segmenterというlibavformatを参照しつつ分けるプログラムもあるし、それの派生プロジェクトのm3u8Segmenterというのもあるんですが以下の点が気に入りませんでした。
1:mpegtsの分割時にキーフレームに考慮せず実行する。
2:mpegtsしかサポートしていない。mp3はだめ
3:分割後の処遇をいじることができない。
この３つが気に入りませんでした。
 まず1、キーフレームを考慮せず切ります。どうやら世に出回っているツールは時間で分割することに念頭をおいているみたいですね。
この方法で分割すると、ライブストリームでやっている場合、再生の先頭にきたファイルによっては、始まるまで待たされることがあります。
音声だけなって、しばらく黒画面みたいな感じになります。どうしてかというと、h.264のデータはキーフレームから開始しないと、表示が乱れたり
表示すらできなかったりするからです。
とりあえず今回のプログラムでは、キーフレームを考慮して、durationの指定を超えた状態でかつはじめにキーフレームがきたところで分割するようにしています。
 続いて2、mp3に対応していない件
別にmp3に対応していなくても音声のみのmpegtsを生成して分割すればいいんですが、ブラウザの挙動にちょっと違いがでてきます。
mp3の場合は、モバイルサファリを閉じて別のアプリにいってもBGMが鳴ったままにすることができるんですが、mpegtsの形式の場合は停止してしまいます。
ゲームしながら・・・とかやる場合には、やっぱりmp3で流す方が都合がよかったので、今回いれてみました。
 最後に3、ファイルの生成タイミングに処理を追加することができません。
segmenterのc言語のプログラムをいじってもいいのですが、これだとコンパイルが結構面倒です。なので今回javaで書いてみました。
特にeventListenerとかは追加していませんがまぁ、問題ないかなと思っています。

 このプロジェクトは本来flvのダウンロード->xuggleでのコンバート->segmenterでの分割を一気にするつもりだった
https://github.com/taktod/streaming/tree/streaming
こちらのプロジェクトで利用しようとおもっていた動作の再利用です。
ちょっとxuggleの複数コンバートが芳しくないので、あきらめて別の動作でFMSとの連携を作ろうと考えています。
その副産物です。

バージョン履歴
2012/11/10 0.1とりあえずつくったバージョン

参考データ
mp3:
  http://mpgedit.org/mpgedit/mpeg_format/mpeghdr.htm

mpegts:
  http://www.arib.or.jp/english/html/overview/doc/2-STD-B10v4_7.pdf
  http://en.wikipedia.org/wiki/MPEG_transport_stream
  http://en.wikipedia.org/wiki/Packetized_elementary_stream
