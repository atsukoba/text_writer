
# 0から学ぶNLP(自然言語処理)[超入門]

#### はじめに
「技術書展って興味ある？」
「今、人工知能系の書籍執筆してて、」
ある日突然こんな感じのDMが飛んできました。
ただ、技術書展のことを知らなかった私は
「なにそれ！！すごそう！！！」
くらいの認識で、気がつくとこんな感じで参加していました。
ただ、今回"技術書展", "人工知能系の書籍"など強そうなワードが飛んできて少しビビりつつもなんとか自分にできそうなこと調べて、せっかくなので今まで自分のやってないことに挑戦してみようと思い文章生成をテーマに参加させていただきました。
機械学習とか今回初めてまともに数式とか読みながらやったような感じなので、もし何か質問、ツッコミなどありましたら是非ともご連絡ください。

#### NLP(自然言語処理)とは
- NLP(Natural Language Processing)はコンピュータによる自然言語、私たちが普段話している言葉を処理する。といった分野になっています。
- この辺りをつらつらと述べても面白くないので、実例を出してみます。
- 実際には以下のようなことをします。

1. 文章分析(クラスタリング)
    - 文章のネガティブポジティブを判定してみたり、誰の書いた文章に近いかを判定してみたり、関連性の高いワードを出力してみたり
2. 文章生成
    - 文章を学習させて特定の人が書きそうな文章を書かせてみたり
3. 文章要約
    - 長文を与えて、そこから重要な語や文を選択し、まとめてみたり
 
このようなことが主に行われています。
今回は特に文章生成を主として紹介していこうと思います。
 
#### 前提知識
- 高校数学レベルの数学の知識
- python、標準ライブラリに関する知識

#### マルコフ連鎖を用いた文章生成
##### マルコフ連鎖とは
マルコフ連鎖くらい聡明な読者の皆様方であればすでに四則演算と同じレベルで扱えると思いますが、今回は全く無知であった私が本当に１から調べた上での私の理解をここに記していこうと思います。(本当にgoogleで「マルコフ連鎖」って検索するとこから始まりました。)
まあ、実際にgoogleで"マルコフ連鎖"と検索すると真っ先にwikipediaが出てきました。
> マルコフ連鎖 - wikipedia より引用
> > マルコフ連鎖（マルコフれんさ、英: Markov chain）とは、確率過程の一種であるマルコフ過程のうち、とりうる状態が離散的（有限または可算）なもの（離散状態マルコフ過程）をいう。また特に、時間が離散的なもの（時刻は添え字で表される）を指すことが多い（他に連続時間マルコフ過程というものもあり、これは時刻が連続である）。マルコフ連鎖は、未来の挙動が現在の値だけで決定され、過去の挙動と無関係である（マルコフ性）。各時刻において起こる状態変化（遷移または推移）に関して、マルコフ連鎖は遷移確率が過去の状態によらず、現在の状態のみによる系列である。特に重要な確率過程として、様々な分野に応用される。

まあこれをばーって言われたところで状態が離散的とか色々訳わからないですよね。これを簡単に言い換えてみると

- 何かを予測する際に今現在の値だけで予測をする。

ということですね。
じゃあ逆に今現在の値以外の値による予測が何か？と言われると過去のデータによる予測というものがあります。
例えば株価予測をしようとした時に今現在の情報だけ(現在の株価や、気配値表など)これだけによる予測だと、短期的な予測はできたとしても、長期的な予想というのはむずかしいですよね？
というのも、過去のデータ(過去にどのような値動きをしてきたのかなど)から、「この会社は今現在は少しさがっているが、長い目で見れば少しずつ上がっている」などと言ったようなものが過去のデータによる予測。
というふうな感じかなと思っています。
時間が絡んでくると哲学的なアレが混ざってくるのでだんだん訳わからなくなるので私は勝手にこういうことだと思っています。

> マルコフ連鎖 - wikipedia より引用 (一部改変)
>> また、マルコフ連鎖は一般に以下の式で表されます。
>> 一連の確率変数![param](https://pbs.twimg.com/media/DLwEI44VAAAiiw0.jpg "param")で現在の状態が決まって入れば過去、および未来の状態は独立である。その際に
>> 
>> ![Markov](https://pbs.twimg.com/media/DLvC1niVAAAGvAu.jpg "Markov")
と表される。

これはここでいう確率変数とは少し機械学習ぽく言い換えると学習によって決定された各単語毎の重み。と言えると思います。例えば "私" という単語の後には "は", "が", "に"などの単語が来る確率が高い。といったような重み付けです。

- 重み付けとは
  - 重み付けとは、例えば状態Aから状態BもしくはCに状態が遷移するという選択肢が存在する

際に単純に等確率でそれぞれに遷移するのではなく、Aの値によってBに遷移しやすかったり、Cに遷移しやすかったりする。その遷移しやすさの調整を"重み付け"と言います。

また、これらは有向グラフと呼ばれる表現方法で表現すると非常にわかりやすいので有向グラフで確認していきます。
ここではよく例に挙げられる天気の例を出したいと思います。
例えば、

1. 今日'晴れ'である時の翌日の天気の確率がそれぞれ
    - {'晴れ':0.5, '曇り':0.3, 'あめ':0.2}
2. 今日'曇り'である時の翌日の天気の確率がそれぞれ
    - {'晴れ':0.3, '曇り':0.4, 'あめ':0.3}
3. 今日'あめ'である時の翌日の天気の確率がそれぞれ
    - {'晴れ':0.2, '曇り':0.3, 'あめ':0.5}

であるとすると、
まず、これを表にしてみると以下のようになります。


|  |晴れ|曇り|あめ|
|:--:|:---:|:---:|:---:|
|晴れ|0.5|0.3|0.2|
|曇り|0.3|0.4|0.3|
|あめ|0.2|0.3|0.5|


これを有向グラフで表してみます。
すると、

![graph](https://pbs.twimg.com/media/DLwPyPwV4AAaR7E.jpg "graph")

このようになります。
表の時より幾分か視覚的にわかりやすくなったと思います。
この表で順にノード(晴れとか曇りとかの丸っこいやつ)から出ているエッジ(なんか数字のついてる矢印)を辿っていくと、それぞれの状態遷移の確率がわかると思います。

これは例えば、

> 今晴れである場合に翌日雨になる確率は0.2

といったように読んでいきます。
また、確率は0~1の間の小数点で表し、パーセントで表すと1が100%となります。
(ex: 0.2だと20%)

では、これを文章に当てはめて見ましょう。
ここでは、どういった単語の後にどんな単語がどの程度の確率で出現するかがあらかじめ分かっているものとします。
ここに何から単語を一つ与えてみます。
ここでは`私`という単語を使用します。
すると、以下のように次に続く語が確率順に出てきます。
![kotonoha](https://pbs.twimg.com/media/DLxa9UBV4AA2qCY.png "kotonoha")

マルコフ連鎖で文章を生成する際、このように次に続きやすい語がいくつか現れ、その中からもっともらしいもの(今までに一番多かったパターン)を選択していくことで文章を生成していく。

では、この確率をどのように決定していくのかその部分について詳しく書いていこうと思います。

しかし、この部分は意外と簡単で分かってしまえばなんか作れそうな気がしてしまいます。
実際には以下の手順を踏んで確率を決定します。

1. 大量の文章を分かち書き(MeCabなどで形態素解析して空白区切りの文を作成する。)
2. 任意の単語数(３単語くらいが多い理由は後述)くらいで一つのまとまりを作る。
3. 2.で作成した一つのまとまりで一つのデータとし、記録する。
4. これを文章が尽きるまで繰り返す。

これだけです。
なんだかんだで非常にシンプルになっています。
そして３単語くらいで区切ることが多い理由ですが、おそらく文章の中で一番シンプルなものとなると、主語　助詞　動詞　の三つで構成されており、逆にそれ以上長くなると、単語同士の関係が弱くなることがあるからだと考えています。
例えば`人は死ぬ`という文章があった際に、'人','は','死ぬ'という三つの単語には出現頻度に強い相関がありそうではありますが、これ以上長い文となると4つ目以降の単語と、一つ目の単語'私'との間にあまり関係がないように思われてしまいます。というのも
`人は死ぬ`のような文の後には大抵接続語が続き、文が続くようなケースが多いというようなことから、３単語を一つの区切りにしているのかと思われます。
また、単語数を増やすほど計算量も増える。ということも理由の一つにあるかもしれません。

#### では、なぜマルコフ連鎖を使うのか
これについては正直なところ「文章　自動生成」でググったらマルコフ連鎖が一番よさげだったからってのが大きいです。
では、あえてマルコフ連鎖の性質を踏まえて考えると、会話であると、それまでの会話内容など過去のデータに関係があるかもしれませんが、単純に文章を生成する場合は基本的に文の中での単語さえあればそのデータから次に続く単語が予測ができるので、その場合今回のマルコフ連鎖が適しているのではないかという見解です。

##### 具体的に何するの？
さて、マルコフ連鎖がどう言った特徴を持っているのかがわかったところで、実際にどうやって文章を生成していくのかの追っていきます。
1. たくさんの文章を用意する。
2. それらを形態素解析に掛け、単語毎に区切る。
3. 文章データから、どういった単語の後にどのような単語が続きやすいのか。ということを学習(データを集める)
4. ランダムに主語となりうる単語を１語選び、そこから次に来る単語を予測する。
5. 文末になるまで繰り返す。

- 使用環境
  - MacBookPro
  - python 3.6.1
  - MeCab neolog-d
そして、環境構築ですが、今回はOSはmacを、brew,pythonは導入済みで進めて行きます。(もし分からなければ'brew インストール'とかってぐぐると私なんかよりよっぽどわかりやすく説明してくれてる方々がたくさんいらっしゃいますので参考にしてください。)

- MeCab, ipadic-neologdのインストール 
こいつが結構厄介です。
[mecab-ipadic-NEologd : Neologism dictionary for MeCab-GitHub](https://github.com/neologd/mecab-ipadic-neologd/blob/master/README.ja.md)ここを参考にしながらインストールする。多分これが一番早いと思います。

MeCabやらのインストール

- `brew install mecab mecab-ipadic git curl xz`

mecab-ipadic-NEologd のインストール

- `git clone --depth 1 https://github.com/neologd/mecab-ipadic-neologd.git`
- `cd mecab-ipadic-neologd`
- `./bin/install-mecab-ipadic-neologd -n -a` 結果の確認画面でyesと入力してEnter

neologd のインストール先の確認
- ``` echo `mecab-config --dicdir`"/mecab-ipadic-neologd" ```


> 1. たくさんの文章を用意する

この部分から始めていきます
今回は今までずっとやって見たかった自分のツイートっぽいツイートを自動生成したかったのでTwitterから過去のツイート履歴を取得してきました。
さて、これでそれなりの量のテキストデータが得られました。
なんかすごいアレで学習させるぞ！！というわけにもいかず、データのクリーニングという作業が必要になります。

###### 実は自然言語処理をする上で(個人的に)一番めんどくさい部分です。

というのも、生のデータは機械にとっては非常に複雑です。
さらには目的にそぐわない文字列が多数混ざってます。
今回だと例えば誰かのTwitterIDであったり、画像何かのブログのURLだったり、様々なデータが入っています。
さらには今回、csvでデータをダウンロードするものの、普段のツイートに改行コードが混ざっているとcsvとして正しく読み込めませんでした。(非常に厄介)
また、今回botやそういった他者からのツイートをカウントしない為にsourceをチェックし、Twitter for iPhone とTweetDeckのみをカウントしてます。

正規表現を武器にこいつらを消し去ります。
また、今回は単文生成を目的としているので、文章中の改行などは全て消し去り、改行は文末とみなして進めていきます。

まず、src/cleaning.pyを使用してデータクリーニングを行います。
ここでは、ハッシュタグやリプライのユーザーID、URLなどを正規表現にて削除しています。

また、その後は` [Text Generator- o-tomox](https://github.com/o-tomox/TextGenerator)
`を元にpython３に最適化及び、今回の目的に合わせて少し修正したものを使用します。

クリーニングによって得られたデータを形態素解析にかけ、３単語ずつのまとまりを作ってデータベースを作成します。
これでいわゆる学習部分は完成です。

その後は、何かランダムな語を一語与え、データベースから辞書データを取り出し、その語のその後に続きやすい単語を調べ、単語を繋げていく。という作業になります。文末にくるような単語がくると文が終了するようになっています。

このスクリプトの応用例としては実際にTwitterのTLなどを監視しておき、キーワードっぽい単語を見つけるとそれに続きやすそうな単語を調べ、文を生成することでその時話題になっているトピックに関する文を生成することもできます。

実際には
/dataset フォルダにtweets.csvをぶち込み、
/src　フォルダの中で以下の順にコマンドを実行すると、自動生成された文章が表示されます。
`python cleaning.py`
`python storeTweetDB.py`
`python maketweet.py`


[GitHub-nekosfc text-writer](https://github.com/nekosfc/text_writer "GitHub text_writer")


今回これらの制作にあたり、
[TextGenerator](https://github.com/o-tomox/TextGenerator)
や
[マルコフ連鎖を使って自分らしい文章をツイートする](https://qiita.com/hitsumabushi845/items/647f8bbe8d399f76825c)
をがっつり参考にしました。もし詳しく知りたい方がいらっしゃればそちらをどうぞ

> 参考文献
> [Text Generator- o-tomox](https://github.com/o-tomox/TextGenerator)
> [マルコフ連鎖-wikipedia](https://ja.wikipedia.org/wiki/%E3%83%9E%E3%83%AB%E3%82%B3%E3%83%95%E9%80%A3%E9%8E%96)
> [マルコフ連鎖を使った文章自動生成プログラム-GitHub](https://github.com/o-tomox/TextGenerator)
> [マルコフ連鎖を使って自分らしい文章をツイートする-Qiita](https://qiita.com/hitsumabushi845/items/647f8bbe8d399f76825c#_reference-07639e2b4da0c1e4c457)
> [mecab-ipadic-NEologd : Neologism dictionary for MeCab-GitHub](https://github.com/neologd/mecab-ipadic-neologd/blob/master/README.ja.md)