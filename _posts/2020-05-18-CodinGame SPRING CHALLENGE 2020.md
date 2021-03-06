---
layout: post
title:  "CodinGame SPRING CHALLENGE 2020に参加しました(じゃんけんパックマン)"
tags: [competitive programming, CodinGame]
---

去年のcode a la mode以来2回目の参加で，世界52位，日本9位でした．  

![rank](https://qLethon.github.io/images/CodinGame%20SPRING%20CHALLENGE%202020/rank.png)  

code a la modeは世界383位でゴールド止まりだったので成長です．  
Masterにもなれました．

![rating](https://qLethon.github.io/images/CodinGame%20SPRING%20CHALLENGE%202020/rating.png)  

レジェンドになった時のコード(最終的に出したのもこれ)は576行でした．

## ゲームルール
[tuskammoさんのブログ](https://tsukammo.hatenablog.com/entry/2020/05/08/064319)を読むと良いです．

## 基本的な方針
- 各pacからdfsする(id順)
- dfsする時の盤面の評価は敵の情報とペレットの情報，既に決定したルートで決める
- dfsする時にi歩目のスコアはiで割る(実際に行く確率、行ってもその評価が実際に得られる確率が低いため)

最初からこの基本方針はあまり変わっていません.

dfsは同じ場所を通ることを許した深さ10の探索と，同じ場所を通ることを許さない深さ20の探索をして評価が良い方を採用しました．  
敵の評価は敵がいるマスを含めて敵から距離3までの場所に対して，距離が遠いほど影響を低くして評価しました．  
今経路を決めようとしている自分のパックからみて，敵の各パックの状態によって次のように評価を分けます．
* 敵が確定で倒せる位置にいて，手でも勝っているパックなら評価は大幅プラス [30, 15, 15, 3]
* 確定で倒せないけど手では勝っている．もしくはあいこなら微マイナス [-1, -1, 0, 0]
* 手で負けているなら大幅にマイナス [-30, -15, -15, -3]

括弧の中は最後の提出の実際の評価値です．左から距離0，1，2，3のマスの評価です．

## コンテスト中に考えたこと

コンテスト中のメモが[hackmd](https://hackmd.io/@qLethon/HkdH2oVcI)にあるので，このメモを見ながら自分でコメントしていきます．  
コンテストの途中で上から書くようにしたので一番上は途中からになっています．時系列順で最初の部分は真ん中くらいにあります．

### 改善
> 閉じ込められた時に相手との距離が1ターンになるまで変えない

やってません．

> 1 <= turn % 10 <= 6以外は3ターン読んでいいかも

敵が見えない時に1ターン前の情報があればそれを使って通常の評価を2で割って使っていたのですが，敵がSPEEDを使っていない時は移動が遅いので，その時は2ターン前まで見ていいかなと思ってやったけどなぜか弱くなったので破棄．

> 伏兵潜ませて追い込み漁業やりたい
> - 相手が確実に漁ってない場所は一応分かるから予測は経つけど，めんどくさいね

めんどくさかったしターンがかかってあまり強くない気がしたからやめた．

> 複数に閉じ込められた時に一番近いのに反応するようにする

閉じ込められた時に勝てる手に変えるやつを実装していたけど複数体に閉じ込められている場合は遠い敵に反応することがあった(id順)．これは自明改善だけど機会がそんなに多くないので直さず．

> 相手がCHANGEできなかったら倒すやつやってないね

なにこれ．多分相手がSWITCH使えない状態で倒せるなら倒すみたいなやつだけどとっくにやってる．書く前からやってる．

> doubledない方が強いかもしれない 38位
> - いやこれINを2倍にしたらそりゃ弱くない？そのターン限りにする

自分と敵のスコアの合計が全体の2/3を越したら目視したペレットは通常の2倍の評価値にしていたが，見えてる間は毎ターン2倍にしていて壊れていた．ターン毎にリセットするように直したらなぜか弱かったのでこのまま．本当はターン毎に減衰するようにしたかった．

> パックが多いほど2倍を早くしていい? 2 / n

上述の2倍する処理のタイミングをパックが多いほど早くした方が良いかと思った(パックが多い方が展開が早いので)けどそうでもなかったから棄却．

> 後半になったら見えているものは*2する

これがそうだね．実装した．

> 取る予定のものを0にしているけど距離減衰をかけるか

dfsで決まったルート上の評価はそのターンのうちは0にしていた(以降のパックのdfsの評価に含めないため．これをしないといい場所にみんなで行くことになる)けど，遠いものは行く確率が低くなるため，(距離/dfsの最大の深さ)をかけた．

> パレット取るまでは最短移動

試してない．10点ペレットのこと．これが強い気がするけど．

> 取り残しが気になるなあ…

後半2倍にしたことで気にならなくなった気がする．

> 後出しジャンケン(直線上にいるとき)
> - 後出ししてくる相手にはSPEED打った方が強い

実装した．  
前提として，敵とぶつかってスタックしている時敵は自分のパックと同じタイプなので見えていなくても敵のパックのタイプが分かる．その時相手に先にSWITCHをさせて自分は相手の1マス前に移動，相手は自分に勝てるようにSWITCHしてくるはずなので，自分を食べに来るはず．その時に相手に勝つようにSWITCHすれば倒せる．というのが流行っていた．直線上にいる時だけこの処理をすればいいと思っていたらしいがいつでもやる．敵もこの処理を組んでいるとずっとぶつかったままなので相手が後出し処理を組んでいそうなら先にSPEEDを発動したら1ターン先制できて有利．

> 相手がSPEEDじゃなくて自分がSPEEDなら追える
> - これターンずらしたら強い
> - 弱い

弱い．
判定を入れること自体は採用したが，SPEEDのターンをずらすのは次の要因で弱い．
* どこかでSPEEDを使うのを5ターンくらい待たなければならない
* 上位に来ると敵も追う処理を実装しているため，5ターン待っただけ損

turn % 10 == 5の時にSPEEDを使うようにずらすとこの効果を最大化できるぞ．

> 後出しさせて他のSPEEDで狩る

スタックしている時にわざと後出しさせSWITCHを使わせて他のSPEEDを使った有利パックで狩る．  
実装はしてない．

> 一個後ろを付いてきていたらチェンジして倒す．

当然やってない．

> パックの数で重さを変える

敵パックの評価値の重さを敵が少ないほど高くする．弱かったので辞めた．

> 自分と敵で挟んでるときってあるやん(追える判定にする)

よく分からん．  
閉じ込めていて相手がスキル持っていなかったら倒す処理を入れていたけど，その処理は壁で閉じ込めているかどうかしか判定していなかった．パックの位置も考慮して判定するという意味だと思うが，最短距離を使って判定していたため後から入れるのは面倒だった．

> 一本道のカドを見に行ってパレットがなかったら奥もなさそう

そうかもね

> シンプルに一筆書きをするのが無駄な動きなくて強いんじゃないの？竹雄さんとか絶対コレだろ(違います)

違います．  
試したかったけどめんどくさかった．

> 初ターンにかなり探索していそう
 
ごるど行くまで初ターンに何もしていなかった

> 近くの評価が高い人からdfs(5!は重いので)

強そうだがやってない．

#### ここが時系列順で最初

> 重み付きスコア
> - マップに素点を降る
> - ターン毎に重みを付ける これを真面目に考える ちゃんと線形に下がるように 最小公倍数
>   - 1/xって線形じゃなくないか困ったな
>   - 線形と非線形は同じくらい？
>   - dfs2対応したら非線形の方が強いかも
> まだ遠回りしたほうがいい評価になってる時がある
> じゃんけんで勝てる相手は素点+(相手のスキルがある時は別)

最初intで評価を持っていたため各ターンの最小公倍数をかけて整数の範囲で除算できるようにしていた．しかし[1, 20]の最小公倍数をかけると64bit必要だしいっそのことdoubleにした．そしたらバグった．

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">doubleがintにキャストされて情報が無に帰していた</p>&mdash; 真紅色に染まるぷーん (@pu__Ne) <a href="https://twitter.com/pu__Ne/status/1261870992135081985?ref_src=twsrc%5Etfw">May 17, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

  

> ルートがなかったらvisited = trueして探索
> - 点が低かったらとかの方が良さそう？
> - それでもなかったら重心とかに移動(一番近いペレットでもいい)

戻れるdfs深さ10でなかったら戻れないdfs深さ20．結局どっちもやって評価の高い方を取った．それでもない場合が見た限りなかったので次の処理は入れてない．もしかしたら見てないだけで止まってるパックがいたかも．

> 10ターンまでしか読まないのは弱い
> - 行き止まりに行く必要があるか(効率が悪い)？→要検証(場が少なくなってきたら変えるとか)
> - 枝狩りかなあ…

行き止まりは取った方が良いっぽい．枝狩りはしたかったけどどうやればいいか分からなかったしいれてない(3往復とかは明らかにしなくていいよね．していた)

> 初期は対象なんだから敵の位置分かるね
> - これ最初のターンだけ分かっても意味ないね(最初はどうせSPEEDなので) 
> - ペレットが確定するまでは3とかにしておくか？
> - まず視界を確保する

結局1ターン前の情報は1/2の評価にして使うことにした．  
3にするは意味がわからない．けど確定したら2倍にした．  
視界の確保は何もやってない．

> 敵が前回いた場所を引き継ぐ？係数を下げて
> - ターンごとに幅を広げていく？
> - とりあえずこれ弱いので却下で…
> - 幅を広げて行くかなんかしないと凄い弱い
> - 1ターンくらい残しておいていいか

残した

> 何度もぶつかっている場合はタイプを変える（ぶつかっている場合は同じタイプなので）

これごるどの途中までは刺さったけど途中で後出しじゃんけんに変更．

> 1, 5, 10, 5, 1

敵に対する評価の分布のイメージ．10が敵がいるマスで離れるほど下がっていく．

> 最初にまず10点ペレットを取る(担当を割り振る)
> - 割り振られたら最短距離で移動でいいかなあ

近いのに割り振ったけど最短移動はしてない．探索に任せた．した方が良かったかな．

> 10点ペレットが消えたら更新する(何ターンで消えたかでルートも算出できる説)

実は10点ペレットは常に見えているのでいつ消えたかが分かるのですよね．ルートの算出はしてない．カス．

> 10点に一番近い人から始めるか…

やってない

> 角に追い込めない場合は殺さなくていいかな(追い込むターン数まで計算して)
> - 追い込めててスキルあれば殺せる
> - 逆に追い込まれている時は変更しようね

角に追い込めて相手がSWITCHできない時に倒す処理は自明改善．追い込まれてる時に変更するのは実は弱くて，相手が近くに来てからSWITCHした方がいい(後出しじゃんけん)．でも実装してない．

> 相性悪いので誘ってCHANGE

やってない．

> 相性がいいの角待ち

やってない．

> パックが少ない場合は全滅させる

無理

> 実際にぶつかることはなくても避けちゃうなあ…

そうなの？そうかも．

> 正面からぶつかるのは良いんだけどな…

敵にぶつかるのを避けていたけどぶつかったらSWITCHする処理を入れたので，同じ種類ならぶつかった方が得やろと思ってたけど普通に相手も同じ処理を入れてるし後出しじゃんけんも入れだしたので全然得じゃない．一回休み．

> 相手の近くの分岐まで自分の方が近ければ閉じ込められそう
> 最初に全頂点間距離と分岐かどうかを求めておく
> 閉じ込められるなら全ての分岐に対して自分は早く到達できるはず

そうだね．本当は奥に1マスしかない場合は分岐にしない処理を入れたかったけどめんどくさいし強いかどうかは微妙．

> SPEEDだと？

上の処理は自分や相手にスキルがあるかどうかで変わってくる．そこら辺はうまくやる(やった)．

> 20読むんだからdfs2の方が強くて当たり前じゃないか？

10歩読むdfsと20歩読むdfs両方を比較してスコアの良いものを取るようにしたらほとんど20歩の方が良くなっていた．当たり前．20歩側に$\frac{\sum_{k = 1}^{10} \frac{1}{k}}{\sum_{k = 1}^{20} \frac{1}{k}} \approx 0.814113$をかけて解決．

> 浅いなら交差点扱いにしない

やってない．

> dfsの順番
> - それぞれでやってスコア大きい順かなあ…(間に合うのか？)

やってない．これかなりやった方が良かったらしい．失敗．

> 勝てるけど閉じ込められない時は+0.1くらい

追いかけるの弱いから負にしたほうが良い．

```
for (p in superpallet)
    cur[n];
    dist[n];
    dist[n] + dist(cur, p)が最小に割当
```

10点ペレットを人に割り当てる時の疑似コード

### done
> SPEED
> じゃんけんで勝てるなら突っ込む

doneがこの2つだけってヤバイだろ．  
doneに放り込むと完全に目に入らなくなるので実装してしばらくしても改善に残していたらdoneに何も来なくなった．

### 要調整
> nターン後の場所を保存

既に決まった経路と被らないようにnターン後に行く場所をメモっていた．新たな経路を探索する時にdfsでi <= kならi歩目までの他の経路と被らないようにしていた．最終的にkは5にしていたが何も根拠はない．調整もしていない．

### バグ
> この状況でしなないで
> - https://www.codingame.com/replay/465919113

リプレイを見てもイマイチ分からない．というかなんで最初にみんなチョキになってるんだ？

> 確実に見えてるやつは2倍にしてよさそう(後半だけか？)

これはバグじゃないだろ．

> 自分にはまだぶつかってそう

そういう時期もあったのかな．これを書いたのは後半だった気がするけど．何もしてない．

> 死んだ時に2マス目取れてないのに取れてるようになってるとかありそう

なかった．

### fixed

> SPEED使った時マップが壊れる

しばらくSPEEDを使っていなかったのでこれに気付いたのはだいぶ後だった．各ターンにパックがいる位置のペレットをなくす処理しか入れていなかったために壊れていた．前回の位置からの最短経路を計算してその間のペレットを除いて対応した．この処理をnターンの場合に拡張していれば...

> dfsで取った時に0にした方が良い

dfs中に通ったマスの評価をそのままにしていたためずっとスーパーペレットの場所を往復する経路で評価していた．カス．

> 行き止まりがバグっていそう
> - visitedなくすことで解決？

最初は戻れないdfsしかしてなかったため行き止まりがだいぶ不利だったので直した．

> 評価同じ時行ったり来たりしてないか(多分平気)(重み付きによる解決)
> - 敵の隣にくると動けなくなる

k歩目のスコアをkで割ることで遠くのものほどスコアが低くなるようにした．(戻るようなルートを考えると，さっきいた場所の方が行き先に近く評価が高いため，今選ぶならその時にそのルートを選んでいるはず．よってそのようなルートは選ばれない)．  
敵の周りは負の評価にしていたため，敵の隣にいると動かず0点を取るのが最適行動になっていた．ZOCか？自分から先は敵の評価を反映させないことにして解決．

> 人とぶつかる
> - 移動をコストにする
> - 人の近くの評価を低く？

上述の通り移動はコストにせず重みを付けた．人の近くの評価を低くもひゃった．

> 見えてなかったライン消してなさそう…(ペレット)

ブロンズに上がって視界が制限された時に，視界のペレットがないことが判明してもそのままあると思って虚無を食べ続けていた．



## あとがき

今回こどげに参加して，アルゴと比べるとまだ未成熟だなと思いました．  
アルゴは蟻本が出ていてAtCoder Problemsのようなサイトもあるし，トッププレイヤーでなくともかなりそれ専用の精進を積んでいます．CodinGameを中心に生活をしていて精進をしている人は今回のコンテストだと上位20人くらいなのかなと勝手に思いました．(過去問を解いてそれなりに精進している人はもっといると思うけど)．なので，初心者でもまだわんちゃん上位に食い込めるという感じがしました．

レジェンドになりたい君、今すぐCodinGameを始めよう！([ここ](https://www.codingame.com/multiplayer/bot-programming/spring-challenge-2020)で私のbotと対戦できます)