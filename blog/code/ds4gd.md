## 生命動態のデータサイエンス(DS2)
（春学期 火曜日３時限）
慶應義塾大学　政策・メディア研究科　後期博士課程　教育体験　講義資料
（Last updated on June 1st by Daniel Evans-Yamamoto）



内容：COVID19の原因ウイルスSARS-CoV-2 (Severe acute respiratory syndrome coronavirus 2)について、公開データをもとにコドン使用バイアス、変異情報の解析を行う（[第1回](https://github.com/DanYamamotoEvans/DanYamamotoEvans.github.io/blob/master/blog/code/ds4gd.md#教育体験第1回202062)）。[第2回](https://github.com/DanYamamotoEvans/DanYamamotoEvans.github.io/blob/master/blog/code/ds4gd.md#教育体験第2回202069)では、タンパク質の立体構造データを[Protein Data Bank (PDB)](https://www.rcsb.org)から取得し、変異情報を立体構造に描画する。最後に、[第3回](https://github.com/DanYamamotoEvans/DanYamamotoEvans.github.io/blob/master/blog/code/ds4gd.md#教育体験第3回2020616)では、ウイルスのタンパク質とホストのタンパク質の結合面を確認し、変異がそのようなドメインに濃縮しているのか、ヒトとSARS-CoV-2のの遺伝的多様性を確認する。
　評価は、各回に貸される課題に基づいて行う。


## 教育体験　第1回　2020/6/2
- Introduction
- SARS-CoV-2のゲノムをダウンロード
- ORF領域を抽出、コドン使用頻度を確認
- SARS-CoV-2株間の変異情報の確認
- タンパク質配列に影響を与えるのはどれくらいの頻度か？
- SARS-CoV-2には変異のホットスポットはあるのか？

## 教育体験　第1回　2020/6/9 (予定)
- タンパク質の構造をPDBからダウンロード
- Rpdb を用いて描画
- String からPPI 情報、PPI interface情報を取得
- 変異情報をタンパク質立体構造に描画

##  教育体験　第1回　2020/6/16 (予定)
- ウイルス側の情報で得たことを、ホストの関連タンパク質で行う
- 機能（ドメイン）と、Enrichmentに関係があるのかを考察
- PPI のco-evolutionについて学ぶ


## memo 
install.packages("bio3d")
library(bio3d)



x <- read.pdb(system.file("examples/PCBM_ODCB.pdb",package="Rpdb"))

- 各変異がタンパク質の構造にマッピング

- タンパク質相互作用に重要なドメインに当たるかを確認
- PPIドッキングの情報と照らし合わせる
- 各変異の影響を考察　(レポート)
