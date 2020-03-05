## Commands for Drop-seq analysis using TTCK server (King)


### Dropseq-toolsでの処理の流れ

fastqファイルから細胞ごとの遺伝子発現量のテーブル(DGE : degital gene expression file)を作成する。
- Cell barcodeとUMIの抜き出し
- リードのトリミング
- ゲノムへのマッピング(STAR)
- マッピング結果にバーコード情報を付与
- 遺伝子情報の付与
- バーコードの修正
- 細胞の抽出
- 種（mouse, human)ごとの細胞数のカウント


その後、Seuratを用いて発現量の表からクラスタリングや各クラスターを代表する遺伝子を探す。




### Preparation

#### Login to TTCK server

```unix
ssh USER_NAME@cs0.bioinfo.ttck.keio.ac.jp
```

#### Login to cluster server (king)

```unix
ssh king
```

シークエンスファイル（Fastq）は下記のディレクトリにアップロードしています。
```unix
/home/daney/projects/Drop_seq/data
```

Drop-seqの解析ツールはあらかじめ下記のディレクトリにあります。
```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp
```

#### Make project directory
```unix
#ディレクトリを作成する
mkdir project
mkdir project/dropseq_bootcamp

#作成したディレクトリに移動する
cd project/dropseq_bootcamp

#現在の位置を確認する
pwd

# /home/あなたのusername/project/dropseq_bootcamp のように出てくればok。

#今後は基本的にこのディレクトリで作業します。
```







### Drop-seq toolsでデータを処理する

#### 1. Dropseq toolsのインプット用にファイル形式を変換（fastq to bam) !!TEST->ok!!
```unix
java -jar /home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/jar/lib/picard-2.18.14.jar  FastqToSam F1=/home/daney/projects/Drop_seq/data/09202019-yachielab-dan_S3_L001_R1_001.fastq  F2=/home/daney/projects/Drop_seq/data/09202019-yachielab-dan_S3_L001_R2_001.fastq  O=work.bam SM=example
```

#### 2. Cell barcodeを抜き出してタグ付け !!TEST!!
```unix
```

#### 3. UMIを抜き出してタグ付け !!TEST!!
```unix
```

#### 4. クオリティが低いリードをフィルタリング !!TEST!!
```unix
```

#### 5. SMART adapter 配列の除去（Start site trimming) !!TEST!!
```unix
```


#### 6. Poly Aをトリミング !!TEST!!
```unix
PolyATrimmer INPUT=work.cb.UMI.filtered.trimS.bam OUTPUT=work.cb.UMI.filtered.trimS.trimA.bam OUTPUT_SUMMARY= work.cb.UMI.filtered.trimS.trimA_report.txt MISMATCHES=0 NUM_BASES=6 USE_NEW_TRIMMER=true
```



#### 7. STARでアライメントするためファイル形式を変換(bam to fastq)

```unix
java -jar for_dropseq_bootcamp/Drop-seq_tools-2.3.0/jar/lib/picard-2.18.14.jar SamToFastq INPUT=work.cb.UMI.filtered.trimS.trimA.bam FASTQ=work.cb.UMI.filtered.trimS.trimA.fastq
```

