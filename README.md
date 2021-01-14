# Santa2020competition
Kaggle diary for Santa 2020 - The Candy Cane Contest

このリポジトリはSantaコンペ2020のKaggle日記です。

## timeline
1/14　joined<br>
2/2   deadline

### 1/14
Lindadaさんの公開カーネル：pull_vegas_slot_machines add weaken rate continue5　をそのまま提出。
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
