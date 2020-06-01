## 生命動態のデータサイエンス[DS2]
（春学期 火曜日３時限）
慶應義塾大学　政策・メディア研究科　後期博士課程　教育体験　講義資料
（Last updated on June 1st by Daniel Evans-Yamamoto）



内容：COVID19の原因ウイルスSARS-CoV-2 (Severe acute respiratory syndrome coronavirus 2)の配列解析を

第1回
- Introduction
- SARS-CoV-2のゲノムをダウンロード
- ORF領域を抽出、コドン使用頻度を確認
- タンパク質の構造をPDBからダウンロード
- Rpdb を用いて描画

第2回 (予定)
- String からPPI 情報、PPI interface情報を取得
- 変異情報をタンパク質立体構造に描画
- 変異頻度をタンパク質立体構造に描画
- 機能（ドメイン）と、Enrichmentに関係があるのかを考察

第3回 (予定)
- ウイルス側の情報で得たことを、ホストの関連タンパク質で行う
- PPI のco-evolutionについて学ぶ



## memo 
install.packages("bio3d")
library(bio3d)



x <- read.pdb(system.file("examples/PCBM_ODCB.pdb",package="Rpdb"))

- 各変異がタンパク質の構造にマッピング

- タンパク質相互作用に重要なドメインに当たるかを確認
- PPIドッキングの情報と照らし合わせる
- 各変異の影響を考察　(レポート)
