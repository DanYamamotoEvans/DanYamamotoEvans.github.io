## 生命動態のデータサイエンス[DS2]
（春学期 火曜日３時限）



担当回について、以下のことをJupyter notebook＋Collaborator (https://colab.research.google.com/notebooks/intro.ipynb)で行うことを考えています。

内容：

- SARS-CoV-19のゲノムをダウンロード
- ORF領域を抽出、確認
- タンパク質の構造をPDBからダウンロード
- Rpdb を用いて描画

install.packages("bio3d")
library(bio3d)



x <- read.pdb(system.file("examples/PCBM_ODCB.pdb",package="Rpdb"))

- 各変異がタンパク質の構造にマッピング

- タンパク質相互作用に重要なドメインに当たるかを確認
- PPIドッキングの情報と照らし合わせる
- 各変異の影響を考察　(レポート)
