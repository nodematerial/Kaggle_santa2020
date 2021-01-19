# Santa2020competition
<img src="figure/titlefigure.png">
Kaggle diary for Santa 2020 - The Candy Cane Contest

このリポジトリはSantaコンペ2020のKaggle日記です。

## コンペ情報
強化学習っぽい、内容はそこまで難しくはないかな。

>この大会は、確率ベースの強化学習問題である「マルチアームド・バンディット問題」をモデルにしています。
>
>この問題では、両方の参加者が同じセットの100台の自動販売機を使って行動します。各自動販売機は、その自販機に固有の確率分布に基づいたランダムな報酬を提供します。
>各プレイヤー>が自動販売機を選ぶ（「引く」）毎に、報酬の可能性は3%減少する。
>
>各エージェントは他のエージェントの動きを見ることができますが、**それぞれの自動販売機の引きで報酬が得られたかどうかはわかりません。**
>
>このエピソードはプレイヤー1人あたり2000ラウンド(合計4000プル)まで続きます。
らしい。相手の情報は限られている。
>エージェントは、自分の合計報酬、前のターン(lastActions)に両プレイヤーが引いた盗賊、
>競争の現在のステップ、残りのOverageTimeを含むオブザベーションを受け取る。

らしい


## timeline
1/14　joined<br>
2/2　　deadline

## 目標
金　上位11
銀　上位50
銅　上位100

銅メダル、できれば銀メダル
### 1/14
***
Lindadaさんの公開カーネル：pull_vegas_slot_machines add weaken rate continue5　をそのまま提出。**1st commit**<br>
https://www.kaggle.com/a763337092/pull-vegas-slot-machines-add-weaken-rate-continue5<br>
 →結果、レート1030、順位200位ぐらいに落ち着いた。
同カーネルの内容を解読した。
#### わかったこと
```
def get_next_bandit():
    best_bandit = 0
    best_bandit_expected = 0
    for bnd in bandit_dict:
        expect = (bandit_dict[bnd]['win'] - bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp'] - (bandit_dict[bnd]['opp']>0)*1.5 + bandit_dict[bnd]['op_continue']) \
                 / (bandit_dict[bnd]['win'] + bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp']) \
                * math.pow(0.97, bandit_dict[bnd]['win'] + bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp'])
        if expect > best_bandit_expected:
            best_bandit_expected = expect
            best_bandit = bnd
    return best_bandit
```
のコードの意味、
Kaggle environmentではもともと次のような辞書が用意されている。
```
 observation = {
    'remainingOverageTime': 60,
    'agentIndex': 1, # 0 or 1
    'reward': 92, # total reward
    'step': 184, # [0-1999]
    'lastActions': [84, 94]
 }
 configuration={
    'episodeSteps': 2000,
    'actTimeout': 0.25,
    'runTimeout': 1200,
    'banditCount': 100,
    'decayRate': 0.97,
    'sampleResolution': 100
}
```
remainingOverageTimeは制限時間?あまり重要ではなさそう。rewardはtotalのreward、lastActionsは前の行動
```
辞書:{a:[1,2,3,4,5....],b:[1,2,3,4,5....],c:[1,2,3,4,5....]}　のような形式<br>
bandint dict={  0: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1},
                1: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1},
                2: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1},
                3: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1},
                4: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1}}
```
というような形。辞書の中に辞書が入っている
### 1/15
***
この時点で、閾値は

| 金 | 1250 |
|----|----|
|銀|1150|
|銅|1100|

となっている。明らかに運ゲーではないだろう。

hansung.devさんのカーネルを使って情報を集めたhttps://www.kaggle.com/hansungdev/santa-2020-beginner-w-a-simple-bandit

#### わかったこと

>未知の報酬活動は0.97の割合で減少する。

見落としていた。見るからに後半の方でスコアの伸びが小さかったのは、このせいだろう。

>上記の最適化問題は、古典的な多腕バンディット（MAB）問題です。強化学習の第一段階でよく説明されているので、以下に抜粋してみました。
>
>強化学習の最も単純な形は、n本の手（＝腕）を持つバンディット、つまり多腕バンディットです。バンディットをn個のハンドルを持つスロットマシンと考えるのは簡単です。
>各ハンドルは異なる確率で報酬を提供します。エージェントの目標は、最も高い報酬を提供するハンドルを見つけ、常にそれを選択することによって、時間の経過とともにリターンの報酬>を最大化することである。
>
>n個のスロットマシンが強化学習に最初に降りてくるときの良い出発点である理由は、時間の依存性や状態の依存性を心配する必要がないからです。n個の手を持つスロットマシンで考えな>ければならないのは、どの行動にどのような報酬が関連しているのか、そして最善の行動を選択するようにすることだけです。

>Naive(またはRamdom)探索 : greedyポリシーにノイズを追加 (例: Epsilon-greedy, SoftMax)
>楽観的な初期化：そうでないと証明されるまで最良のものと仮定する
>不確実性に直面したときの楽観主義 . 不確実な値を持つ行動を好む（例：UCB
>確率マッチング : ベストである確率に応じてアクションを選択する (例: トンプソンサンプリング)
>情報状態検索：情報の価値を取り入れたルックヘッド検索

MAB問題について知る必要がありそうだ。

幸いQiitaに記事があったので、読んでみることにしよう。

強化学習入門：多腕バンディット問題：https://qiita.com/tsugar/items/b809f8d6399cc988aa69

kernelもqiita記事も、1-εで今までで一番スコアが良かったものを選び、εでランダム探索を行うという内容である。しかし、この手法はそれほど強くないようである。

今回のコンペで厄介な点は、同じ自動販売機を選び続けたら当たりの確率が小さくなることと、相手の情報の一部を使えるという点だろうか???そのため単純な上のアルゴリズムはそのまで機能しないということだろうか、うーん、確率が時間変動するので当然といえば当然であるが...

#### やったこと
とりあえずmath.pow(0.97,〜)の部分をmath.pow(0.96,〜)にしてみた、意味はよくわかっていない。**3rd commit**<br>
1st commitと比較→前のほうが強そうだった<br>
とりあえず提出→強い スコア:1120~1130　銅圏に入った(!?)<br>
やけくそになって、math.pow(0.97,〜)の部分をmath.pow(0.98,〜)にしてみた、まさに運ゲー**4th commit**<br>
1st commitと比較→よくわからない<br>
とりあえず提出→弱い スコア:930 増やすのは駄目だと分かる<br>
これらの結果を受けて、math.pow(0.95,〜)にしてみた**5th commit**<br>
1st commitと比較→分からない...<br>
とりあえず提出→そこそこ強い　スコア:1105<br>

```
def multi_armed_probabilities(observation, configuration):
    (省略)
        if last_reward > 0:
            my_pull = my_last_action
        else:
            if observation['step'] >= 4:
                if (my_action_list[-1] == my_action_list[-2]) and (my_action_list[-1] == my_action_list[-3]):
                    if random.random() < 0.5:
                        my_pull = my_action_list[-1]
                    else:
                        my_pull = get_next_bandit()
                else:
                    my_pull = get_next_bandit()
            else:
                my_pull = get_next_bandit()

    return my_pull
```
どうやら、上で定義したget_next_bandit()と、my_last_action(前の行動)のどちらかを選択するようである。<br>
詳しい中身を解読する。まず、`if last_reward > 0:`であるが、前の引きで当たりだったら必ずもう一回選択するようだ

```
else:
        last_reward = observation['reward'] - total_reward
        total_reward = observation['reward']
```
ここでrewardの更新、及び前の試行が当たったか外れたか判定
```
my_idx = observation['agentIndex']
        my_last_action = observation['lastActions'][my_idx]
        op_last_action = observation['lastActions'][1-my_idx]
        my_action_list.append(my_last_action)
        op_action_list.append(op_last_action)
```
自分と相手の選択を抜き出す。そしてリストに保存する。
```
   if 0 < last_reward:
            bandit_dict[my_last_action]['win'] = bandit_dict[my_last_action]['win'] +1
        else:
            bandit_dict[my_last_action]['loss'] = bandit_dict[my_last_action]['loss'] +1
```
ここでは、last rewardの値に応じて、bandit_dictを更新する。
~~例えば{...1235:{},1236:{},1237{win:1,loss:0,...},1238:{},1239:{}...}
が負けると{...1235:{},1236:{},1237{win:1,loss:1,...},1238:{},1239:{}...}になる????~~
→ 少し違っていた。辞書のキーは試行回数ではなく自動販売機のインデックスであった。つまり
    {0:{}...14:{},15:{},16{win:1,loss:0,...},17:{},18:{}...100:{}}　が、
    {0:{}...14:{},15:{},16{win:1,loss:1,...},17:{},18:{}...100:{}}　になるというのが正しい
```
        if observation['step'] >= 3:
            if my_action_list[-1] == my_action_list[-2]:
                bandit_dict[my_last_action]['my_continue'] += 1
            else:
                bandit_dict[my_last_action]['my_continue'] = 0
            if op_action_list[-1] == op_action_list[-2]:
                bandit_dict[op_last_action]['op_continue'] += 1
            else:
                bandit_dict[op_last_action]['op_continue'] = 0
```
ここだけ違う???get_next_bandintでop_continueが使われていた。
ここだけ自分で変えたら面白いかも...
アイデアとしては、continue_list(辞書だとやりにくいので)を作り、相手が2

少し寝たらだいぶ対戦が進んでいた、それぞれのsubの対戦結果は以下の通り(落ち着いてきた)

| 名前 | 係数 | スコア |
|----|----|----|
|1st commit|0.97|1024.5|
|3rd commit|0.96|1134.6|
|4th commit|0.98|924.3|
|5th commit|0.95|1107.5|

実験的には0.95~0.97の間にスコアを最大化する点がある？(上に凸の二次関数的な)
ということで頭悪いけど、間の値でローラー作戦を行う。

math.pow(0.97,〜)→math.pow(0.955,〜) **6th commit**<br>
math.pow(0.97,〜)→math.pow(0.965,〜) **7th commit**<br>

上手くいくか???

#### 結果
6th commit　デフォルトカーネルよりは強い
7th commit　commit5のほうが強そう

現段階での格付けは　commit4(0.98)＜commit1(デフォルト,0.97)＜commit6(0.955)＜commit7(0.965)＜commit5(0.95)＜commit3(0.96)　である。<br>
0.96台をローラーする？これで今の所銅メダルは守れそう
math.powの意味だが、指数部部分は(bandit_dict[bnd]['win'] + bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp'])で、今まで引かれた回数を指している。
つまり、初期の確率と比べて、どれだけ減衰しているかを指している。つまり、底の部分を0.97よりも小さくするということは、減衰のスピードを早める。つまり、多数選択されているものを、より選びにくくなるということである。→これによってスコアがデフォルトカーネルのそれよりも向上しているということは示唆的である。

このコンペのジレンマは、ずっと同じものを選び続けていては、確率がどんどん下がってしまう。しかしながら、ランダムに飛び回っていたとしたら、高い確率を持つ自動販売機にとどまる時間が短くなってしまう。いい塩梅を狙わないといけない。

デフォルトカーネルを読み終わった。構造としては`get_next_bandit()`は、**ある評価式**に従って、最も期待できる自動販売機のインデックスを返す。`multi_armed_probabilities(observation, configuration)`は、評価式で利用する、インデックス毎の情報(win:自分の当たり回数,loss:自分のハズレ回数,opp:相手が選んだ回数,my_continue:自分が連続で選んだ回数の合計,op_continue:相手が連続で選んだ回数の合計)を更新する機能と、次のターン自分が引く自動販売機を決定するという機能を持っている。重要なのは太字の評価式の部分と、最後の次のターン自分が引く自動販売機を決定するという部分だろう。これ以外は基本的にいじる必要はないだろう。

#### 評価式
```
(bandit_dict[bnd]['win'] - bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp'] - (bandit_dict[bnd]['opp']>0)*1.5+ bandit_dict[bnd]['op_continue']) \
    / (bandit_dict[bnd]['win'] + bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp']) \
    * math.pow(0.965, bandit_dict[bnd]['win'] + bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp'])
```
<!-- \frac{win-loss+opp-f(opp)\times1.5)}{win+loss+opp} \times (0.95\sim 0.97)^{win+loss+opp} -->
<img src="https://latex.codecogs.com/gif.latex?\bg_white&space;\frac{win-loss&plus;opp-f(opp)\times1.5&plus;opcontinue}{win&plus;loss&plus;opp}&space;\times&space;(0.95\sim&space;0.97)^{win&plus;loss&plus;opp}" />
<img src="https://latex.codecogs.com/gif.latex?\bg_white&space;opp=0:&space;f(opp)=0&space;\quad&space;opp\neq0:f(opp)=1">

f(opp)の存在意義が少し不明な感じがするが、相手が一回選んですぐ選ぶのをやめたものをよりえらびにくくする意味があるのだろうと推測する。
#### 決定アルゴリズム
```
 if observation['step'] >= 4:
    if (my_action_list[-1] == my_action_list[-2]) and (my_action_list[-1] == my_action_list[-3]):
        if random.random() < 0.5:
            my_pull = my_action_list[-1]#前の3つが同じだったら、50%の確率で同じのに固執し続ける。
        else:
            my_pull = get_next_bandit()
    else:
        my_pull = get_next_bandit()
else:
    my_pull = get_next_bandit()
```
そこまで言及すべきことはないかな?stepが4以下で例外処理をするのは2行目のエラーを防ぐためだろう。
#### アイデア
分子のlossを消す。すなわち、ハズレのときのペナルティーは無しにする(別に当たったときに補正かけるだけでいい気がする)→失敗<br>
前の3つが同じだったら、50%の確率で同じのに固執し続ける。というのを、少し変える。例えば80%にしたい。

| 係数 | ハズレペナあり | ハズレペナ無し |
|----|----|----|
|0.950|commit5|commit8|
|0.955|commit6|commit10|
|0.960|commit3|commit9|
|0.965|commit7|commit11|

完全に失敗、よく考えたら、ペナルティがないと相手の行動次第でずっと同じものと選んでしまうだろう。<br>しかし、これにより、逆にペナルティを強くするという可能性を思いついた。

| 係数(ペナあり) | 固執50% | 固執80% |
|----|----|----|
|0.950|commit5|commit12|
|0.955|commit6|commit13|
|0.960|commit3|commit14|
|0.965|commit7|commit15|

| 係数(ペナなし) | 固執50% | 固執80% |
|----|----|----|
|0.950|commit8|commit16|
|0.955|commit10|commit17|
|0.960|commit9|commit18|
|0.965|commit11|commit19|

1/15,1/16,1,17のsubを使って実験。
####　結果
| 名前 | score |
|----|----|
|commit8|500|
|commit10|500|
|commit9|500|
|commit11|中止|
|commit12|490|
|commit13|560|
|commit14|中止|
|commit15|中止|
|commit16|中止|
|commit17|中止|
|commit18|中止|
|commit19|中止|


次は係数をかけたペナルティをかける方式で実験する。ていうか自分のNotebookの中で対戦できることに今気づいた、遅すぎ。

### 1/16
起きたら興味深いことが起こっていた。commit7がcommit3を上回っていた。<br>
commit12,13をsubしたが、カススコア、どちらの実験もうまく行っていない。そんなもんよ。

commit7がcommit3を上回っていたので、推定最強係数は今の所0.965ということになる。が、0.95もそこそこ強いので、一意の値に確定することはできない？
>次は係数をかけたペナルティをかける方式で実験する。

これを実行する、まずは0.8,0.9,1.1,1.2倍に補正を行う。まず自分の環境内で実験を行ってからsubする。

####　仮説　書き下し
1.1　序盤強い??
oppの係数も変えたほうがいい?

とりあえずペナルティの係数変化を継続する。

| 係数=0.965 | 名前 | 結果 |
|----|----|----|
|ペナ=0.8|中止  |無し|
|ペナ=0.9|commit22|370|
|ペナ=1.1|commit20|830|
|ペナ=1.2|commit21|740|

あまり変えないほうがいいのか?しかし、係数1が最適解であるのは決して自明ではない。一方でスコアが激下がりしているのも確かである。もう少し刻めば良いのか？

| 係数=0.965 | 名前 | 結果 |
|----|----|----|
|ペナ=0.99|中止||
|ペナ=1.01|commit23||
|ペナ=1.02|commit24||
|ペナ=1.03|中止||
そこまで弱くはなさそうだが、めちゃくちゃ強くもない、しかし、これらは自分最強のエージェントであるcommit7(今までcommitと読んでいたが、versionと呼ぶべきか?)に近いパラメータを有しているので、近いスコアを示すと期待されるが、大きくスコアが変わるようだったら、それはそれで興味深い結果ではある。

### 1/17
銅圏で安定してホッとしている自分がいるが、強気で銀を狙いに行かないと殺されてしまう。世の中そういうもん。
ここでヤバいミスに気づく、本当はversion7との対照実験をしているつもりであったが、固執係数が0.8のままであった(強いわけがないのだ!)。version20から対照実験やり直しです。はい（泣）。

0.9から1.2まで、0.5刻みでoppの係数を変化させる。

| 係数=0.965 | 名前 | 結果 |
|----|----|----|
|ペナ=0.90|version25|1033|
|ペナ=0.95|version26|1112|
|ペナ=1.00|version7 |1123|
|ペナ=1.05|version27|980|
|ペナ=1.10|version28|1015,1020|
|ペナ=1.15|version29|中止|
|ペナ=1.20|version30|中止|

version26もそこそこ強そう。もう万策尽きたら0.95~1.01の間を0.1刻みでローラーしよう。

次は、op_continueの補正について実験を行う。係数が大きいと、相手がいい自動販売機を見つけた際に、横取りしやすくなる。0だと、それが一切なくなる(連続は全く考慮に入れないというだけであって、選んだ回数自体は重要視する。)また、`(bandit_dict[bnd]['opp']>0)*1.5`の意味についてだが、相手が一回選んですぐに捨てたやつを拾いにくくする効果がある。
ふと思いついたのだが、私が今使っているカーネルは、公開されたカーネルであり、多くの人が使用していると考えられる。そうでないとしても、op_continueのアイデアは使っている人は一定数いるのではないか?だとしたら、こちらがあえてノイズを入れる、つまり、いい自動販売機をみつけてもあえて連続ではえらばないように改造すれば、相手の戦略を台無しにすることができるかもしれない。しかし、実装が少し大変であるし、効果が限定的であるので、諸刃の剣になるだろう。さされば嬉しいが。

さて、今はop_continueの話であった、まず、係数を大きくすることで、良い自動販売機を積極的に奪いに行く手を考える。係数を1.4~2.0で0.2刻みにする線形的な考えと2^xのオーダーで指数的に増加させる考えの両方を使っていきたい。

| 係数=0.965 ペナ1.00 | 名前 | 結果 |
|----|----|----|
|1.4|version31|強くない|
|1.6|version32|強くない|
|1.8|version33|中止|
|2.0|version34|中止|
|2.2|version35|中止|

次に、べき乗則で係数が増えていくバージョン、式としては'n^(op_continue)'　で、nが係数に相当する。1.4ならn=0,1,2,3に対して、1.0,1.4,1,96,2.744と増えていく。<br>アイデアは悪くなさそうだが、パラメータの調整が難航しそう。

| 係数=0.965 ペナ1.00 | 名前 | 結果 |
|----|----|----|
|1.4|version36|中止|
|1.5|version37|中止|
|1.6|version38|中止|

ちょっと混乱してきた。op_continueの挙動が自分の想定と違っていた。`last_opponent_list[-2]`としないと、望む挙動は得られない???0.001*bndで重複を防ぐ？

別のNotebookを作って、色々実験を行った。
alpha_version1:つよそう

しかし、自分の手に縛られて、相手のを積極的に奪いに行けていない。もっと高確率で相手の手を奪いに行く。
beta_version1:
beta_version2:
beta_version3:
序盤弱い

係数を増やしても弱い。3,7,26が強い

紆余曲折あり、結局op_continueはいらないのではないかと考え始めた。

version22の実験がミスってることに気がついた。*が**になっていなのでペナ0.9が激弱だった件は、なかったことになる。

ver36:op_continueを消し、係数を0.5にしている。現最強エージェントのver26にはめっぽう強かったが...?
ver37:op_continueを消し、係数を1.0にしている。
ver38:op_continueを消し、係数を0.8にしている。
ver39:op_continueを消し、係数を0.9にしている。
ver40:op_continueを消し、係数を1.1にしている。

最近思ったのは、終盤戦では、すべての自動販売機の当たり確率が均質化されて、ほぼrandomに選んでも性能はそこまで変化しないのではないか。そうだとしたら、op_continueは、特に終盤に関しては、何も意味がなかったことになる。つまり、op_continueに関する仮説が大外れだったことになる。(時間返せ)
