---
title: "八木作戦 — 二射"
topics: ["RadioWave", "MobileNetwork", "YagiAnntena", "Telephony", "Band"]
type: "tech"
emoji: "🐚"
published: true
---
# 八木作戦 — 二射

電波を集めて圏外を殲滅せよ

---

# 🌒️ 序

前回、八木作戦 — 一射 ( [Zenn](https://zenn.dev/nosaki/articles/70621_operation_yagi_docomo-27870), [Qiita](https://qiita.com/nyosaki/items/353fa12f7d1ba58121ef) ) では、 docomo band 42 向けに設計した八木アンテナを試した。今回は、 au band 18 向けを試す。

# 🌕️ 破

## 材料

    - 白木材 ¥148 (910×12×18mm)
    - アルミ ¥256 (995×3mm, 2本)
  - 合計 ¥404 (含む消費税等 ¥36)

ホームルーターの外部アンテナは、台座に磁石がついている。それを使って固定するために、鉄製の小さなプレートを探してみたのだが、良いものが無い。最終的に 100均でおもちゃのような蝶番を買った。平面でない部分はゴムを埋めて固定する。

    - 蝶番 ¥110 (16×22mm, 4個)
  - 合計 ¥110 (含む消費税等 ¥10)

![hinge.png](https://nyosak.github.io/article-base-doc/media/70628_operation_yagi_au_hinge.png)

## 装備強化

前回の反省会を踏まえ、新たな装備を2点調達した。

    - 3.1mmφドリル刃 ¥548
    - 10mmタガネ ¥398
  - 合計 ¥946 (含む消費税等 ¥86)

キリでつけた小さな穴だと、ドリル刃の食いつきが悪くズレる原因になるので、タガネで大きめの凹みを作る作戦だ。

![tagane.png](https://nyosak.github.io/article-base-doc/media/70628_operation_yagi_au_tagane.png)

左から、今回追加したタガネ、今回追加したドリル刃、前回使用したキリ。

## 加工

簡易図面を用意する

![au_band18.png](https://nyosak.github.io/article-base-doc/media/70628_operation_yagi_au_au_band18.png)

![au_band18_realized.png](https://nyosak.github.io/article-base-doc/media/70628_operation_yagi_au_au_band18_realized.png)

前回の反省会を踏まえ、木は 910mm 全長をそのまま使い、両端の遊びを増やす。

アルミ棒を切断するのだが、前回の余りが 560mm あるので、そちらを活用する。無駄なく切り出すために木取りの試算をする。

![book_match.png](https://nyosak.github.io/article-base-doc/media/70628_operation_yagi_au_book_match.png)

無駄なく配置でき、1本まるごと余った。買い物より先に木取りしておけば良かったかも。

前回の反省会を踏まえ、穴あけのズレ対策として、次を行った。

- 事前練習1回
- 12mm 厚の補助板を用意し、ガイドを棒と平行に目線の正面に置く
- 穴ごとに棒の固定位置をずらして、常に同じ立ち位置と方向で穴をあける

![wood.png](https://nyosak.github.io/article-base-doc/media/70628_operation_yagi_au_wood.png)

いい感じ。

3.1mmφであけると、穴がちょうどの大きさなので、輪ゴムの固定が不要になった。

![done.png](https://nyosak.github.io/article-base-doc/media/70628_operation_yagi_au_done.png)

組み上がった。 docomo band 42 向けと比べると、やはり大きい。

![base.png](https://nyosak.github.io/article-base-doc/media/70628_operation_yagi_au_base.png)

![done2.png](https://nyosak.github.io/article-base-doc/media/70628_operation_yagi_au_done2.png)

アンテナの土台をつける。アンテナ側に他の素子を寄せる。

## 実戦

確認は窓際の、 docomo 基地局が目視できる室内で行った。立地的に、 au も含め全キャリアがそこに基地局を有する可能性が高い。

ホームルーターにはディスプレイが無いので、📶のような情報が取れない。その代わりに、設定用の web 画面で信号強度 (dBm) を見ることができる。

- 室内: 八木アンテナ無し: -85
- 室内: 八木アンテナ有り: -85

変わらない。

だと⁉

弐号機、完全に沈黙…

## 反省会

明確な信号の増幅は確認できなかった。その原因になりそうなものを列挙する。

- 基地局の方角が違う
- 外部アンテナの接続不良
- 外部アンテナのサイズで給電素子を固定した設計をしていない
- web 画面の信号強度表示が信頼できない

今回は、アンテナサイズが大きく、また電源コード縛りもあるので、室内の固定位置でしか試せなかった。室外を含めた状況確認が必要だと思われる。

# 🌖️ 急

二部完結の予定だったが、どうやら第三部に続きそうだ。アルミ棒が余ったのはフラグだったか。

