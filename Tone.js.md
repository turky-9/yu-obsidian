https://bakkyalo.hatenablog.jp/entry/2024/06/01/043217

#### とりあえず鳴らしてみる - triggerAttackRelease()

以下を HTML の `body` の末尾に置いてください。[*1](https://bakkyalo.hatenablog.jp/entry/2024/06/01/043217#f-527359d1 "必ずしも末尾でなくても良いですが、少なくとも \"playButton\" が見つかる状況にはしてください。")  

``` js
document.getElementById("playButton").addEventListener("click", () => {
    const synth = new Tone.Synth().toDestination();
    synth.triggerAttackRelease("C4", "8n");
});
```

これで "Play" ボタンを押すと「真ん中のド」(C4) が鳴るようになるはずです。

- `Synth()` で音源を作り、`toDestination()` でそれをコンピュータのスピーカーで鳴らすよう指示します。
- `triggerAttackRelease()` は、その瞬間に音を鳴らし始め (Attack)、指定された長さの後に離せ (Release) ということを意味します。Attack と Release を別々に指定することもできますが (`triggerAttack()` と `triggerRelease()`)、今回は使いません。
    - `"C4"` の部分に音名 (`note`) を指定します。C4 がいわゆる「真ん中のド」です。
    - `"8n"` の部分は音を鳴らす長さ (`duration`) を指定します。この場合は 8 分音符分だけ鳴らすことを意味します。

Tone.js で音を鳴らす時は毎回このような手続きを踏むことになります。

  

音名 (`note`) は [Wikipedia](https://ja.wikipedia.org/wiki/%E9%9F%B3%E5%90%8D%E3%81%A8%E9%9A%8E%E5%90%8D) で「科学的ピッチ」と言われている形式で指定します。

- ドイツ語式 (C, D, E, F, G, A, **H**) ではなく英語式 (C, D, E, F, G, A, **B**) を使うことに注意してください。
- `#` でシャープ、`b` (小文字のビー) でフラットになります。例えば `"G#4"` で「真ん中のソ」のシャープ、`"Ab4"` で「真ん中のラ」のフラットになります (`"Gis4"` や `"As4"` では通じません)。当然ですが、これらは同じ音が鳴ります ([異名同音](https://d.hatena.ne.jp/keyword/%B0%DB%CC%BE%C6%B1%B2%BB))。
音の長さ (`duration`) の細かい仕様は以下をご覧ください。

[Time · Tonejs/Tone.js Wiki · GitHub](https://github.com/Tonejs/Tone.js/wiki/Time)

とりあえず当記事では以下のことが分かってれば大丈夫です。

- `"1n"` で[全音](https://d.hatena.ne.jp/keyword/%C1%B4%B2%BB)符分, `"2n"` で 2 分音符分, `"4n"` で 4 分音符分, `"8n"` で 8 分音符分, `"16n"` で 16 分音符分, ... のように n を単位にすると「～分音符分」になります。小数のような中途半端な値でもちゃんと動きます。
- `"4n."` のように末尾にドットを付けると付点 4 分音符分になります。他についても同様です。
- `"1m"` で 1 小節分 (measure) になります。
- `"8t"` は 4 分音符の 1/3、`"16t"` は 8 分音符の 1/3 です。3 連符の duration に使えます。

とりあえず現時点での HTML の例: [*2](https://bakkyalo.hatenablog.jp/entry/2024/06/01/043217#f-618d6431 "VSCode で HTML 拡張を入れると、\"!\" と打つだけでこのようなスニペットが貼られます。")
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://unpkg.com/tone"></script>
    <title>Document</title>
</head>
<body>
    <input type="button" value="Play" id="playButton">

    <script>
        document.getElementById("playButton").addEventListener("click", () => {
            const synth = new Tone.Synth().toDestination();
            synth.triggerAttackRelease("C4", "8n");
        });
    </script>
</body>
</html>
```

#### 時刻の概念 - Tone.now()

さて、今度は単音ではなく旋律を作っていきたいですが、その前に Tone.js の時間の扱いについて知っておく必要があります。

Tone.js には現在時刻 (`time`) という概念があります。これは何かというと、ブラウザでページを読み込んだ瞬間を 0 とした時の、そこから経過した時間の秒数のことを指します。  
単に `synth.triggerAttackRelease("C4", "8n");` とすればその瞬間にド (C4) が 8 分音符分 (8n) だけ鳴ってくれるのですが、例えばその 1 秒後にソ (G4) の音を鳴らしたいと思った場合は

``` js
synth.triggerAttackRelease("C4", "8n");
synth.triggerAttackRelease("G4", "8n", 1);    // これではダメです！！
```

のように書いてはいけません。これだと、ページを読み込んだ 1 秒後にソ (G4) を鳴らせ、という意味になってしまい、(場合によっては) 過去に対する指示になってしまうのでエラーが起きます。ちゃんと「現在時刻の 1 秒後」と指示してあげる必要がある訳です。

現在時刻が欲しい場合は `Tone.now()` を使います。

``` js
const synth = new Tone.Synth().toDestination();
const now = Tone.now();        // 現在時刻の取得
synth.triggerAttackRelease("C4", "8n");
synth.triggerAttackRelease("G4", "8n", now + 1);    // 現在時刻の 1 秒後に鳴らせ！　→ ok!
```

これで、ド (C4) が鳴った 1 秒後にちゃんとソ (G4) が鳴ると思います。

このことを踏まえて、きらきら星の冒頭を鳴らしてみましょう。このように書けるはずです。
``` js
document.getElementById("playButton").addEventListener("click", () => {
    const synth = new Tone.Synth().toDestination();
    const now = Tone.now();
    synth.triggerAttackRelease("C4", "8n");
    synth.triggerAttackRelease("C4", "8n", now + 0.5);
    synth.triggerAttackRelease("G4", "8n", now + 1);
    synth.triggerAttackRelease("G4", "8n", now + 1.5);
    synth.triggerAttackRelease("A4", "8n", now + 2);
    synth.triggerAttackRelease("A4", "8n", now + 2.5);
    synth.triggerAttackRelease("G4", "4n", now + 3);

    synth.triggerAttackRelease("F4", "8n", now + 4);
    synth.triggerAttackRelease("F4", "8n", now + 4.5);
    synth.triggerAttackRelease("E4", "8n", now + 5);
    synth.triggerAttackRelease("E4", "8n", now + 5.5);
    synth.triggerAttackRelease("D4", "8n", now + 6);
    synth.triggerAttackRelease("D4", "8n", now + 6.5);
    synth.triggerAttackRelease("C4", "4n", now + 7);
});
```

最初の `triggerAttackRelease()` には `now` を指定しないでください。タッチの差で過去への指示になってしまいエラーが起きます。
現在の HTML の例:
``` html
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://unpkg.com/tone"></script>
    <title>Document</title>
</head>
<body>
    <input type="button" value="Play" id="playButton">

    <script>
        document.getElementById("playButton").addEventListener("click", () => {
            const synth = new Tone.Synth().toDestination();
            const now = Tone.now();
            synth.triggerAttackRelease("C4", "8n");
            synth.triggerAttackRelease("C4", "8n", now + 0.5);
            synth.triggerAttackRelease("G4", "8n", now + 1);
            synth.triggerAttackRelease("G4", "8n", now + 1.5);
            synth.triggerAttackRelease("A4", "8n", now + 2);
            synth.triggerAttackRelease("A4", "8n", now + 2.5);
            synth.triggerAttackRelease("G4", "4n", now + 3);

            synth.triggerAttackRelease("F4", "8n", now + 4);
            synth.triggerAttackRelease("F4", "8n", now + 4.5);
            synth.triggerAttackRelease("E4", "8n", now + 5);
            synth.triggerAttackRelease("E4", "8n", now + 5.5);
            synth.triggerAttackRelease("D4", "8n", now + 6);
            synth.triggerAttackRelease("D4", "8n", now + 6.5);
            synth.triggerAttackRelease("C4", "4n", now + 7);
        });
    </script>
</body>
</html>
```

なお、今後も [JavaScript](https://d.hatena.ne.jp/keyword/JavaScript) のコードを載せていくことになると思いますが、その際はこのように `"playButton"` の `addEventListener()` の中に追記していくとよいかと思います。
#### 和音 - Tone.Synth() と Tone.PolySynth()

さて、勢いに乗って今度は和音を作っていきたいところですが、同時刻を指定すればええやろってノリで

``` js
const synth = new Tone.Synth().toDestination();
synth.triggerAttackRelease("C4", "8n");    // ここではよくても
synth.triggerAttackRelease("E4", "8n");    // ここで過去の話になってしまう
synth.triggerAttackRelease("G4", "8n");    // 同上
```

とやるとまた過去に対する指示だってエラーになってしまいます。かといって

``` js
const synth = new Tone.Synth().toDestination();
const now = Tone.now();
synth.triggerAttackRelease("C4", "8n", now + 1);    // これで
synth.triggerAttackRelease("E4", "8n", now + 1);    // 和音に
synth.triggerAttackRelease("G4", "8n", now + 1);    // なる？ (ならない)
```

とすれば 1 秒後に一斉に 3 音が鳴ってくれるかと言えばそうではありません。おそらく 1 音しか聞こえてこないと思います。

これは今まで使ってきた音源 `Tone.Synth()` が単音にしか対応していないためで、和音のための音源はちゃんと別に用意されています。その一つが `Tone.PolySynth()` です。

[https://tonejs.github.io/docs/latest/classes/PolySynth](https://tonejs.github.io/docs/latest/classes/PolySynth)

`PolySynth` では、同時に鳴らしたい音を配列で指定することができます。[*3](https://bakkyalo.hatenablog.jp/entry/2024/06/01/043217#f-4926b0ab "1 音ずつ指定することもできます。ただし、その場合は例の如く過去への指示にならないように注意してください。")  

``` js
const polySynth = new Tone.PolySynth().toDestination();
polySynth.triggerAttackRelease(["C4", "E4", "G4"], "2n");    // C Major
```

この例では ド、ミ、ソ、と、 C を根音とする長三和音、いわゆる C Major が鳴ると思います。
3 音の音の長さ (`duration`) が相異なっている場合も、下のように duration を配列で指定することで対応できます。

``` js
const polySynth = new Tone.PolySynth().toDestination();
polySynth.triggerAttackRelease(["C4", "E4", "G4"], ["4n", "2n", "1n"]);    // C Major
```

この場合は C4 が 1 拍、E4 が 2 拍、G4 が 4 拍になります。

さて、これで和音の進行も鳴らせるようになりました。試しに [C dur](https://d.hatena.ne.jp/keyword/C%20dur) の T → S → D → T ([カデンツ](https://d.hatena.ne.jp/keyword/%A5%AB%A5%C7%A5%F3%A5%C4)第 2 型) を鳴らしてみましょう。

``` js
const polySynth = new Tone.PolySynth().toDestination();
const now = Tone.now();

polySynth.triggerAttackRelease(["E4", "G4", "C5"], "2n");           // I
polySynth.triggerAttackRelease(["F4", "A4", "C5"], "2n", now + 1);  // IV
polySynth.triggerAttackRelease(["D4", "F4", "G4", "B4"], "2n", now + 2); // V7
polySynth.triggerAttackRelease(["E4", "G4", "C5"], "2n", now + 3);  // I
```

#### テンポと拍 - Tone.Transport

そろそろ Tone.js の雰囲気に慣れてきたと思いますが、まだちょっと気持ち悪いですね。そうです、テンポと拍の扱いです。

今まではテンポも拍も意識せずにただ何秒後にどの音を鳴らすかで指示をしてきたわけですが、楽譜という人類の素晴らしき発明に慣れ親しんできた我々からするとこれは苦行でしかありません。苦行というか、時代に遡行 (挑戦？) しているとさえ言えます。

例えば、テンポが 80 ([bpm](https://d.hatena.ne.jp/keyword/bpm)) の 3/4 拍子の曲を鳴らそうとした場合に、8 小節目の 2 拍目の時刻 [sec] は

now + (60.0 / 80.0) * (3 * 7 + 1)

とかやってられません。ちゃんとテンポや拍を意識しつつ、小節番号や拍数をダイレクトに指定できるようになりたいものです。

この辺りの設定は、`Tone.Transport` から行います。

``` js
Tone.Transport.bpm.value = 80;           // これでテンポが 80 (bpm) になります
Tone.Transport.timeSignature = [3, 4];   // これで 3/4 拍子になります
```

今まで黙っていて申し訳ありませんでしたが、デフォルトではテンポが 120 ([bpm](https://d.hatena.ne.jp/keyword/bpm))、拍子は 4/4 に設定されています。

他にも、再生/停止/ポーズの操作だとか、リピートの範囲だとか、加速 (アッチェレランド) の仕方だとかの設定も `Tone.Transport` からできます。詳細は document へ。  
しかし、Transport なんてあんまり聞かない用語ですね。日本語で何というんだろう。。。

次に小節番号ですが、これまた Time の話に戻ります。

[Time · Tonejs/Tone.js Wiki · GitHub](https://github.com/Tonejs/Tone.js/wiki/Time)

例えばここの [Wiki](https://d.hatena.ne.jp/keyword/Wiki) の Transport Time の節にある例だと

- `"4:3:2"` で、4 小節分 + 4 分音符 3 個分 + 16 分音符 2 個分

つまり、 (4 拍子の曲の場合) 5 小節目の最後の 8 分に差し掛かるまでの時間を表していることになります。


実際に `triggerAttackRelease()` で音を鳴らす時刻を指定する時は `now` にこの値を足す必要がありますが、`now + "4:3:2"` だと文字列の結合扱いされて値がおかしくなってしまうので、次のように `"4:3:2"` を秒数に変換した後加える必要があります。

now + Tone.Time("4:3:2").toSeconds()

今までの話をまとめてみましょう。  
前節の[カデンツ](https://d.hatena.ne.jp/keyword/%A5%AB%A5%C7%A5%F3%A5%C4)をテンポ 80 ([bpm](https://d.hatena.ne.jp/keyword/bpm)) の 3/4 拍子の曲だとして 1 拍ずつ鳴らしてみると次のようなコードになります。

``` js
const polySynth = new Tone.PolySynth().toDestination();
const now = Tone.now();    // 現在時刻

Tone.Transport.bpm.value = 80;           // これでテンポが 80 (bpm) になります
Tone.Transport.timeSignature = [3, 4];   // これで 3/4 拍子になります

polySynth.triggerAttackRelease(["E4", "G4", "C5"], "4n");       // I
polySynth.triggerAttackRelease(["F4", "A4", "C5"], "4n", now + Tone.Time("0:1:0").toSeconds());  // IV
polySynth.triggerAttackRelease(["D4", "F4", "G4", "B4"], "4n", now + Tone.Time("0:2:0").toSeconds()); // V7
polySynth.triggerAttackRelease(["E4", "G4", "C5"], "2n", now + Tone.Time("1:0:0").toSeconds());  // I
```

#### 音と長さとタイミングを詳細に指定 - Tone.Part()

だんだんと[西洋音楽](https://d.hatena.ne.jp/keyword/%C0%BE%CD%CE%B2%BB%B3%DA)らしくなってきましたね (←)。

しかし、音を鳴らすたびに `triggerAttackRelease()` を毎回呼ぶのは少々まどろっこしいです。加えて、せっかく小節番号や拍数でタイミングを指定できるようになったのに、`now + Tone.Time().ToSeconds()` をかませないといけないのもあまり賢くはなさそうですね。

音を鳴らすタイミング、音名、音の長さが統一的に管理された楽譜のようなものを外部に置いておくことができるのであれば、可読性や保守などの面でも嬉しいところです。

`Tone.Part()` を使えばそれを実現できます。

[Part | Tone.js](https://tonejs.github.io/docs/14.9.17/classes/Part.html)

とりあえず、先ほどの[カデンツ](https://d.hatena.ne.jp/keyword/%A5%AB%A5%C7%A5%F3%A5%C4)を `Tone.Part()` 用に書き換えたコードを見てみましょう。
``` js
const polySynth = new Tone.PolySynth().toDestination();
Tone.Transport.bpm.value = 80;           // テンポを 80 bpm にする
Tone.Transport.timeSignature = [3, 4];   // 3/4 拍子にする

// 楽譜
const score = [
    {"time": "0:0:0", "note": ["E4", "G4", "C5"], "duration": "4n"},
    {"time": "0:1:0", "note": ["F4", "A4", "C5"], "duration": "4n"},
    {"time": "0:2:0", "note": ["D4", "F4", "G4", "B4"], "duration": "4n"},
    {"time": "1:0:0", "note": ["E4", "G4", "C5"], "duration": "2n"}
];

// パート
const part = new Tone.Part((time, note) => {
    polySynth.triggerAttackRelease(note.note, note.duration, time);
}, score).start();

Tone.Transport.start();    // 音を鳴らすのに必須
```

今までのとだいぶ雰囲気が変わりましたが、幾許かはすっきり見え、、なくはないとは言えなくもないのかな。少なくとも、楽譜は音を鳴らす部分から独立できてはいます。`now` に `Tone.Time().toSeconds()` を足すみたいな煩わしい操作もなくなっています。

しかし `Part` 周辺が何だかよくわかりませんね。解説します。
- `Tone.Part` のコンスト[ラク](https://d.hatena.ne.jp/keyword/%A5%E9%A5%AF)タは `Tone.Part(【関数】, 【楽譜】).start();` のように使用します。
    - `Part` くんが `【楽譜】` を parse してくれて、そこから鳴らすべき時刻 (`time`) と音の情報 (`note`) を抽出して順々に `【関数】` の引数に渡していってくれます。私たちはそれら `time`, `note` を `【関数】` の中で自由に使うことができます。なお、引数名は別に `time` や `note` でなくても良いですが、`【楽譜】` を辞書式に定義する場合は時刻の情報を `"time"` で指定する必要があります (→ [Part | Tone.js](https://tonejs.github.io/docs/14.9.17/classes/Part.html)) 。
    - `【楽譜】` は時刻を表す `"time"` キーさえあればあとはどんな key-[value](https://d.hatena.ne.jp/keyword/value) の組があっても良いです。 名前も型も要[素数](https://d.hatena.ne.jp/keyword/%C1%C7%BF%F4)も特に制限がありません。ただ、`triggerAttackRelease()` の仮引数の名前が `notes`, `duration`, `time` (, `velocity`) になっているのでそれに合わせているだけです (→ [PolySynth | Tone.js](https://tonejs.github.io/docs/15.0.4/classes/PolySynth.html#triggerAttackRelease))。複数形の s がないじゃんとか言わない。
- `Part` を定義しただけでは音は鳴りません。実際に音を鳴らすには `Tone.Transport.start();` を置いておく必要があります。

まぁうだうだ説明しましたが、結局は使いながら慣れていくものだと思います。

`Part` はその名の通りパートの役割を担うことができます。例えばピアノソロ曲の場合、右手用のパートと左手用のパートを別々に作り、それらを同時に再生、といったことにも対応できます。[*4](https://bakkyalo.hatenablog.jp/entry/2024/06/01/043217#f-b93ac700 "当然ですが、Synth のように同時に音を鳴らすのに対応していない音源ではできないです。")

せっかくなので有名曲の冒頭でその挙動を確認してみましょう。[モーツァルト](https://d.hatena.ne.jp/keyword/%A5%E2%A1%BC%A5%C4%A5%A1%A5%EB%A5%C8)の[ソナタ](https://d.hatena.ne.jp/keyword/%A5%BD%A5%CA%A5%BF) No.16 K.545 から。

``` js
const polySynth = new Tone.PolySynth().toDestination();
Tone.Transport.bpm.value = 152;         // テンポを 152 bpm にする
Tone.Transport.timeSignature = [4, 4];  // 4/4 拍子にする (デフォルトでなるので指定不要)

// 右手の楽譜
const rightHandScore = [
    {"time": "0:0:0", "note": "C5", "duration": "2n"},
    {"time": "0:2:0", "note": "E5", "duration": "4n"},
    {"time": "0:3:0", "note": "G5", "duration": "4n"},

    {"time": "1:0:0", "note": "B4", "duration": "4n."},
    {"time": "1:1:2", "note": "C5", "duration": "16n"},
    {"time": "1:1:3", "note": "D5", "duration": "16n"},
    {"time": "1:2:0", "note": "C5", "duration": "4n"}
];

// 左手の楽譜
const leftHandScore = [
    {"time": "0:0:0", "note": "C4", "duration": "8n"},
    {"time": "0:0:2", "note": "G4", "duration": "8n"},
    {"time": "0:1:0", "note": "E4", "duration": "8n"},
    {"time": "0:1:2", "note": "G4", "duration": "8n"},
    {"time": "0:2:0", "note": "C4", "duration": "8n"},
    {"time": "0:2:2", "note": "G4", "duration": "8n"},
    {"time": "0:3:0", "note": "E4", "duration": "8n"},
    {"time": "0:3:2", "note": "G4", "duration": "8n"},

    {"time": "1:0:0", "note": "D4", "duration": "8n"},
    {"time": "1:0:2", "note": "G4", "duration": "8n"},
    {"time": "1:1:0", "note": "F4", "duration": "8n"},
    {"time": "1:1:2", "note": "G4", "duration": "8n"},
    {"time": "1:2:0", "note": "C4", "duration": "8n"},
    {"time": "1:2:2", "note": "G4", "duration": "8n"},
    {"time": "1:3:0", "note": "E4", "duration": "8n"},
    {"time": "1:3:2", "note": "G4", "duration": "8n"}
];

// 右手のパート
const rightPart = new Tone.Part((time, note) => {
    polySynth.triggerAttackRelease(note.note, note.duration, time);
}, rightHandScore).start();

// 左手のパート
const leftPart = new Tone.Part((time, note) => {
    polySynth.triggerAttackRelease(note.note, note.duration, time);
}, leftHandScore).start();

Tone.Transport.start();
```
#### 【番外編】 少し特殊な Tone.Sequence()

スコアを parse するツールには他に `Tone.Sequence()` というものがあります。  
`Tone.Part()` では時刻の情報をコンスト[ラク](https://d.hatena.ne.jp/keyword/%A5%E9%A5%AF)タ第二引数の `【楽譜】` から抽出していましたが、こちらの場合は、自動で 1 拍ずつ刻むようになっています。

まぁ、具体例を見るのが早いと思います。[ピアノ曲](https://d.hatena.ne.jp/keyword/%A5%D4%A5%A2%A5%CE%B6%CA)ではないですが、[グリーグ](https://d.hatena.ne.jp/keyword/%A5%B0%A5%EA%A1%BC%A5%B0)の『朝』。

``` js
const polySynth = new Tone.PolySynth().toDestination();
Tone.Transport.bpm.value = 90;          // テンポを 90 bpm にする
Tone.Transport.timeSignature = [6, 8];  // 6/8 拍子にする

// 右手の楽譜
const rightHandScoreSeq = [
    {"note": "B5", "duration": "8n"}, 
    {"note": "G#5", "duration": "8n"}, 
    {"note": "F#5", "duration": "8n"}, 
    {"note": "E5", "duration": "8n"}, 
    {"note": "F#5", "duration": "8n"}, 
    {"note": "G#5", "duration": "8n"}, 

    {"note": "B5", "duration": "8n"}, 
    [{"note": "G#5", "duration": "16t"}, {"note": "A5", "duration": "16t"}, {"note": "G#5", "duration": "16t"}], 
    {"note": "F#5", "duration": "8n"}, 
    {"note": "E5", "duration": "8n"}, 
    [{"note": "F#5", "duration": "16n"}, {"note": "G#5", "duration": "16n"}],
    [{"note": "F#5", "duration": "16n"}, {"note": "G#5", "duration": "16n"}]
];

// 左手の楽譜
const leftHandScoreSeq = [
    {"note": ["E4", "B4"], "duration": "2n."},
    null,
    null,
    null,
    null,
    null,

    {"note": ["E4", "B4"], "duration": "2n."},
    null,
    null,
    null,
    null,
    null,
];

// 右手の sequence
const rightSequence = new Tone.Sequence((time, note) => {
    polySynth.triggerAttackRelease(note.note, note.duration, time);
}, rightHandScoreSeq).start();
rightSequence.loop = 1;      // sequence のループ回数

// 左手の sequence
const leftSequence = new Tone.Sequence((time, note) => {
    polySynth.triggerAttackRelease(note.note, note.duration, time);
}, leftHandScoreSeq).start();
leftSequence.loop = 1;       // sequence のループ回数

Tone.Transport.start();
```

使い方自体は `Tone.Part()` とほとんど一緒ですが、音を鳴らすタイミングは自動で 1 拍ずつ刻む仕様になっているので、何もすることがない場所では `null` を指定する必要があります。逆に同じ場所に複数の辞書がある場合はその数だけの連符になります。また、 1 つの辞書に対して複数の note がある場合は和音になります。

辞書の配列にするか、値が配列になっている辞書にするかで、連符か和音かが変わるので、混同しないように注意しましょう。

`Tone.Part()` のように音を鳴らす時刻を直接指定するわけではなく、拍の数だけ辞書を用意しておく必要があるので少し奇妙に思われるかもしれません。実際私も何がしたいんだ状態でした。しかし、休符を明示的に書くことでリズム感覚が明確になったり、連符が簡単に書けたりと、それなりにメリットもあります。  
`Tone.Part()` とどちらを使うかは個人の趣味の範疇になるでしょう。拍や休符の感覚を大事にしたいという方であれば `Tone.Part()` よりもこちらの方が肌に合っているかもしれませんね。

#### 音源を任意に指定する - Tone.Sampler()

さあ、`Tone.Part()` と `Tone.Sequence()` を習得したあなたは既にあらゆる曲を奏でられるようになったはずです [*5](https://bakkyalo.hatenablog.jp/entry/2024/06/01/043217#f-d9012ec9 "一応、途中でテンポを変えるとか反復記号をどう反映させるかの話がまだありますが... 今回はそこまでやりません。document をご参照ください。")。ここまでお疲れさまでした。

最後にやりたいのは、今まで流し続けてきた電子音源 `Tone.PolySynth()` をピアノ音源に変えることですね。

幸運なことに、そのためのツールとピアノ音源は既に公式によって用意されています。

[Sampler | Tone.js](https://tonejs.github.io/docs/15.0.4/classes/Sampler.html)  
[Sampler](https://tonejs.github.io/examples/sampler)  
[audio/salamander at master · Tonejs/audio · GitHub](https://github.com/Tonejs/audio/tree/master/salamander)

書くべきことは決まっています。何も考えずにあなたのコードに以下を導入しましょう。

``` js
const piano = new Tone.Sampler({
    urls: {
        A0: "A0.mp3",
        C1: "C1.mp3",
        "D#1": "Ds1.mp3",
        "F#1": "Fs1.mp3",
        A1: "A1.mp3",
        C2: "C2.mp3",
        "D#2": "Ds2.mp3",
        "F#2": "Fs2.mp3",
        A2: "A2.mp3",
        C3: "C3.mp3",
        "D#3": "Ds3.mp3",
        "F#3": "Fs3.mp3",
        A3: "A3.mp3",
        C4: "C4.mp3",
        "D#4": "Ds4.mp3",
        "F#4": "Fs4.mp3",
        A4: "A4.mp3",
        C5: "C5.mp3",
        "D#5": "Ds5.mp3",
        "F#5": "Fs5.mp3",
        A5: "A5.mp3",
        C6: "C6.mp3",
        "D#6": "Ds6.mp3",
        "F#6": "Fs6.mp3",
        A6: "A6.mp3",
        C7: "C7.mp3",
        "D#7": "Ds7.mp3",
        "F#7": "Fs7.mp3",
        A7: "A7.mp3",
        C8: "C8.mp3",
    },
    baseUrl: "https://tonejs.github.io/audio/salamander/", 
    onload: () => {
        // ここに音源 piano に対する処理を書く
    }
}).toDestination();
```
- いくつかの音を mp3 ファイルに直接関連付けさせます。実際のピアノのように 88 音すべてに対して指定する必要はなく、指定されていない音に関しては `Tone.Sampler()` 君が自動で補完してくれます。

実用の上では、`onload` の中にいま作った音源 `piano` に対する処理を書いていきます。イメージとしては、今まで書いてきたコードの `polySynth` を `piano` に書き換えてそこに貼り付けるといった感じです。

#### [MIDI](https://d.hatena.ne.jp/keyword/MIDI) ファイルの扱い

直前の fetch [API](https://d.hatena.ne.jp/keyword/API) の話も含め、今回は `Tone.Part()` を使うという基本方針で設計をしましたが、ひとつ難点があります。楽譜の .[json](https://d.hatena.ne.jp/keyword/json) ファイルの中身が独自規格になってしまうという点です。

今回は最低限旋律を流せれば良いという事で、`Tone.Part()` の仕様にかなり合わせた形の楽譜になりましたが、こんな形式を使っている人は世界中に誰もいません。「楽譜ファイル作ったよ」と言って誰かに配布しても、それを再生できる環境が私のページとこの記事に載っているコードを使う一部の物好きくらいしかないので、もんのすごい[ガラパゴス](https://d.hatena.ne.jp/keyword/%A5%AC%A5%E9%A5%D1%A5%B4%A5%B9)になります。  
また、途中でテンポを変える機能だとか、リピートを再現するだとか、さまざまな機能を付けていこうと考えると、再度設計し直しになり、場合によっては今までの独自 .[json](https://d.hatena.ne.jp/keyword/json) が読めないようになってしまう可能性すらあります。

コンピュータ上で音楽データを扱う際に広く一般的に使われている形式はおそらく [MIDI](https://d.hatena.ne.jp/keyword/MIDI) でしょう。.[midi](https://d.hatena.ne.jp/keyword/midi) を使う方針であれば、再生も編集も誰でもできるので、使ってくれる人がそれなりに出てくる、かもしれません。  
ユーザーもどこぞのいかがわしいゲーム三昧オタクが作った独自の .[json](https://d.hatena.ne.jp/keyword/json) 形式なんかよりも .[midi](https://d.hatena.ne.jp/keyword/midi) を使いたいと思うはずです。

実は Tone.js には [MIDI](https://d.hatena.ne.jp/keyword/MIDI) ファイルを取り扱う [API](https://d.hatena.ne.jp/keyword/API) が用意されています。

[Midi](https://tonejs.github.io/docs/13.8.25/Midi)  
[GitHub - Tonejs/Midi: Convert MIDI into Tone.js-friendly JSON](https://github.com/Tonejs/Midi)

さらに、[MIDI](https://d.hatena.ne.jp/keyword/MIDI) ファイルを読み込んで Tone.js でそれを再生する example もすでにできています。

[https://tonejs.github.io/Midi/](https://tonejs.github.io/Midi/)

何というか、この記事でやってきたことが果たして意味があったのかというくらい普遍的かつ強力なツールですね。

今後、Tone.js を使った Web アプリの開発をすることがもしあるとするなら、少なくとも [MIDI](https://d.hatena.ne.jp/keyword/MIDI) くらいは扱うことができるような設計、あるいは計画を意識していかないといけないでしょうね。