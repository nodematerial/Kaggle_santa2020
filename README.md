# Santa2020competition
Kaggle diary for Santa 2020 - The Candy Cane Contest

このリポジトリはSantaコンペ2020のKaggle日記です。

## コンペ情報

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
Lindadaさんの公開カーネル：pull_vegas_slot_machines add weaken rate continue5　をそのまま提出。<br>
https://www.kaggle.com/a763337092/pull-vegas-slot-machines-add-weaken-rate-continue5<br>
 →結果、レート1000、順位200位ぐらいに落ち着いた。
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
|金|1250|
|銀|1150|
|銅|1100|
となっている。明らかに運ではないだろう。

hansung.devさんのカーネルhttps://www.kaggle.com/hansungdev/santa-2020-beginner-w-a-simple-bandit

#### わかったこと
>未知の報酬活動は0.97の割合で減少する。
見落としていた。
>上記の最適化問題は、古典的な多腕バンディット（MAB）問題です。強化学習の第一段階でよく説明されているので、以下に抜粋してみました。
>
>強化学習の最も単純な形は、n本の手（＝腕）を持つバンディット、つまり多腕バンディットです。バンディットをn個のハンドルを持つスロットマシンと考えるのは簡単です。
>各ハンドルは異なる確率で報酬を提供します。エージェントの目標は、最も高い報酬を提供するハンドルを見つけ、常にそれを選択することによって、時間の経過とともにリターンの報酬>を最大化することである。
>
>n個のスロットマシンが強化学習に最初に降りてくるときの良い出発点である理由は、時間の依存性や状態の依存性を心配する必要がないからです。n個の手を持つスロットマシンで考えな>ければならないのは、どの行動にどのような報酬が関連しているのか、そして最善の行動を選択するようにすることだけです。

MAB問題について知る必要がありそうだ。

幸いQiitaに記事があったので、読んでみることにしよう。

強化学習入門：多腕バンディット問題：https://qiita.com/tsugar/items/b809f8d6399cc988aa69
