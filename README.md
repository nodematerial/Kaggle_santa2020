# Santa2020competition
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
らしい、



## timeline
1/14　joined<br>
2/2   deadline

## 目標
金　上位11
銀　上位50
銅　上位100

銅メダル、できれば銀メダル
### 1/14
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
bandint dict={  0: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1},<br>
                1: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1},<br>
                2: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1},<br>
                3: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1},<br>
                4: {'loss': 0, 'my_continue': 0, 'op_continue': 0, 'opp': 0, 'win': 1}}<br>
```
というような形。辞書の中に辞書が入っている
### 1/15
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

今回のコンペで厄介な点は、同じ自動販売機を選び続けたら当たりの確率が小さくなることと、相手の情報の一部を使えるという点だろうか???そのため単純な上のアルゴリズムはそのまで機能しないということだろうか、うーん、確率が時間変動するので当然といえば当然であるが

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
(bandit_dict[bnd]['win'] - bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp'] - (bandit_dict[bnd]['opp']>0)*1.5) \
    / (bandit_dict[bnd]['win'] + bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp']) \
    * math.pow(0.965, bandit_dict[bnd]['win'] + bandit_dict[bnd]['loss'] + bandit_dict[bnd]['opp'])
```
<img src="https://latex.codecogs.com/gif.latex?\bg_white&space;a&plus;b&plus;1=2" />
#### 決定アルゴリズム
```
aaa
```