## 生命動態のデータサイエンス(DS2)
（春学期 火曜日３時限）
慶應義塾大学　政策・メディア研究科　後期博士課程　教育体験　講義資料
（Last updated on June 1st by Daniel Evans-Yamamoto）



内容：COVID19の原因ウイルスSARS-CoV-2 (Severe acute respiratory syndrome coronavirus 2)について、公開データをもとにコドン使用バイアス、変異情報の解析を行う（[第1回](https://github.com/DanYamamotoEvans/DanYamamotoEvans.github.io/blob/master/blog/code/ds4gd.md#教育体験第1回202062)）。[第2回](https://github.com/DanYamamotoEvans/DanYamamotoEvans.github.io/blob/master/blog/code/ds4gd.md#教育体験第2回202069)では、タンパク質の立体構造データを[Protein Data Bank (PDB)](https://www.rcsb.org)から取得し、変異情報を立体構造に描画する。最後に、[第3回](https://github.com/DanYamamotoEvans/DanYamamotoEvans.github.io/blob/master/blog/code/ds4gd.md#教育体験第3回2020616)では、ウイルスのタンパク質とホストのタンパク質の結合面を確認し、変異がそのようなドメインに濃縮しているのかや、ヒトとSARS-CoV-2の遺伝的多様性を確認する。

　評価は、各回に課される課題に基づいて行う。


## 教育体験　第1回　2020/6/2
#### 目次
1. Introduction
2. SARS-CoV-2のゲノムをダウンロード
3. ORF領域を抽出、翻訳
4. SARS-CoV-2株間の変異情報の確認
5. SARS-CoV-2には変異のホットスポットはあるのか？

#### 1.1 Introduction

#### 1.2. SARS-CoV-2のゲノムをダウンロード

NCBIからDNA配列を取得する。SARS-CoV-2のNCBI accessionは武漢で単離された[NC_045512](https://www.ncbi.nlm.nih.gov/nuccore/NC_045512)で、これはRefSeqと言われるレファレンスに指定されている。[NCBIのSARS-CoV-2ページ](https://www.ncbi.nlm.nih.gov/genbank/sars-cov-2-seqs/)にはこれに加えて世界中で採取された5,000近い配列が登録されている。


ここでは先週行った、Rのseqinrパッケージを用いたゲノムのダウンロードを行う。
    
    library(seqinr)
    ACCESSION <- "NC_045512" # SARS-CoV-2
    
    filename <- paste0("https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=",ACCESSION,"&rettype=fasta&retmode=text")

    seqs <- read.fasta(file=filename, seqtype="DNA", strip.desc=TRUE)
    sarscov2 <- seqs[[1]]
    

#### 1.3. ORF領域を抽出、翻訳

SARS-CoV-2ゲノム中には、14の機能的タンパク質がコードされていることが推測されている。今回はSARS-CoV-2の中でも、ヒト細胞に感染する際に重要なSpike proteinに着目する。 タンパク質がコードされている配列位置情報はNCBIのWEBサイト(https://www.ncbi.nlm.nih.gov/nuccore/NC_045512)で確認できる。

    #SARS-CoV-2のSpike Protein はゲノムの21563bpから25384bpの位置にコードされているため、これを抽出する
    spike_cds = sarscov2[21563:25384]


DNAは生体内でRNAに翻訳され、その後遺伝暗号に従いアミノ酸配列に翻訳される。その対応表を下に示す。

![photo](https://blog.addgene.org/hs-fs/hubfs/7_18_to_9_18/codonUsageBias_TJF_2018_9_20/Codon%20Chart.png?width=800&name=Codon%20Chart.png)


ここでは、遺伝暗号表（コドンテーブル）をもとにseqinr内のgetTrans関数で核酸配列をアミノ酸（タンパク質）配列に翻訳する。
    
    spike_AA  = getTrans(spike_cds)
  
    #プリントしてみよう 1文字目は必ずメチオニンを表すMとなるはず
    spike_AA
    

#### 1.4. SARS-CoV-2株間の変異情報の確認

[公開されたCOVID-19のゲノム配列にもとづく疫学データ](https://nextstrain.org/ncov)を見てみると、ウイルス株の変異パターンから流行の過程を推測することができる。そのような変異はゲノム上のどのような位置にあるのだろうか？


これを踏まえて、「変異している＝より感染力が強い」と結論付けられるのだろうか？

進化とは、この多様性の獲得(変異；variation)と選択 (淘汰；selection)が行われることで、次第に環境に適応した個体が残る。選択のステップで獲得された変異が利益（あるいは）不利益を被る場合はそのような進化は助長されるが、中にはあってもなくても変わらない（中立的）変異も存在する。したがって、変異があるからと言ってそれがすなわち危ないということではない。

SARS-CoV-2では、多くのsynonymous, non-synonymous

頻度　（positionを選んで計算してね）




#### 1.6. SARS-CoV-2の変異を実際に（演習課題）

[疫学データ](https://nextstrain.org/ncov)をもとに、ゲノム上の変異位置の一つに着目しよう。

そこで、
- 選んだ変異がどこでサンプリングされたか
- どのタンパク質コード領域か
- タンパク質を
- ゲノム上の変異の位置（Nulceotide position）コード
- ゲノム上の変異の位置（Nulceotide position）
- ゲノム上の変異の位置（Nulceotide position）
を確認しよう。

確認されたサンプリングポイントのゲノムを[NCBIのSARS-CoV-2ページ](https://www.ncbi.nlm.nih.gov/genbank/sars-cov-2-seqs/)で見つけ、IDを控えよう。

1.2 ~ 1.3の


頻度　（positionを選んで計算してね）








## 教育体験　第2回　2020/6/9 (予定)
#### 目次
1. タンパク質の構造をPDBからダウンロード
2. Rpdb を用いて描画
3. String からPPI 情報、PPI interface情報を取得
4. 変異情報をタンパク質立体構造に描画

##  教育体験　第3回　2020/6/16 (予定)
1. ウイルス側の情報で得たことを、ホストの関連タンパク質で行う
2. 機能（ドメイン）と、Enrichmentに関係があるのかを考察
3. PPI のco-evolutionについて学ぶ


## memo 
install.packages("bio3d")
library(bio3d)



x <- read.pdb(system.file("examples/PCBM_ODCB.pdb",package="Rpdb"))

- 各変異がタンパク質の構造にマッピング

- タンパク質相互作用に重要なドメインに当たるかを確認
- PPIドッキングの情報と照らし合わせる
- 各変異の影響を考察　(レポート)
