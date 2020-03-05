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

#Login to node
qsub -I -l nodes=1:ppn=32
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
TagBamWithReadSequenceExtended INPUT=work.bam OUTPUT=work.cb.bam SUMMARY=work.cb.bam_summary.txt BASE_RANGE=1-12 BASE_QUALITY=10 BARCODED_READ=1 DISCARD_READ=FALSE TAG_NAME=XC NUM_BASES_BELOW_QUALITY=1
```

#### 3. UMIを抜き出してタグ付け !!TEST!!
```unix
TagBamWithReadSequenceExtended INPUT=work.cb.bam OUTPUT=work.cb.UMI.bam SUMMARY=work.cb.UMI.bam_summary.txt BASE_RANGE=13-20 BASE_QUALITY=10 BARCODED_READ=1 DISCARD_READ=TRUE TAG_NAME=XM NUM_BASES_BELOW_QUALITY=1
```

#### 4. クオリティが低いリードをフィルタリング !!TEST!!
```unix
FilterBam TAG_REJECT=XQ INPUT=work.cb.UMI.bam OUTPUT=work.cb.UMI.filtered.bam
```

#### 5. SMART adapter 配列の除去（Start site trimming) !!TEST!!
```unix
TrimStartingSequence INPUT=work.cb.UMI.filtered.bam OUTPUT=work.cb.UMI.filtered.trimS.bam OUTPUT_SUMMARY= work.cb.UMI.filtered.trimS_report.txt SEQUENCE= AAGCAGTGGTATCAACGCAGAGTGAATGGG MISMATCHES=0 NUM_BASES=5
```


#### 6. Poly Aをトリミング !!TEST!!
```unix
PolyATrimmer INPUT=work.cb.UMI.filtered.trimS.bam OUTPUT=work.cb.UMI.filtered.trimS.trimA.bam OUTPUT_SUMMARY= work.cb.UMI.filtered.trimS.trimA_report.txt MISMATCHES=0 NUM_BASES=6 USE_NEW_TRIMMER=true
```



#### 7. STARでアライメントするためファイル形式を変換(bam to fastq)

```unix
java -jar /home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/jar/lib/picard-2.18.14.jar SamToFastq INPUT=work.cb.UMI.filtered.trimS.trimA.bam FASTQ=work.cb.UMI.filtered.trimS.trimA.fastq
```

#### 8. STARでアライメント

```unix
STAR --runThreadN 1 --genomeDir /home/daney/projects/Drop_seq/for_dropseq_bootcamp/STAR_database/ --readFilesIn work.cb.UMI.filtered.trimS.trimA.fastq --outFileNamePrefix star
```

#### 9. アライメント結果をソート

```unix
java -jar /home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/jar/lib/picard-2.18.14.jar SortSam I=starAligned.out.sam O=aligned.sorted.bam SO=queryname
```

#### 10. Cell barcode, UMI情報と統合

```unix
java -jar /home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/jar/lib/picard-2.18.14.jar MergeBamAlignment ALIGNED_BAM=aligned.sorted.bam UNMAPPED_BAM=work.cb.UMI.filtered.trimS.trimA.bam OUTPUT=merged.bam REFERENCE_SEQUENCE=/home/daney/projects/Drop_seq/for_dropseq_bootcamp/metadata_dropseq_hg19mm10/genome.fa INCLUDE_SECONDARY_ALIGNMENTS=false PAIRED_RUN=false
```

#### 11. gene情報を付加

```unix
TagReadWithGeneFunction I= merged.bam O=star_gene_exon_tagged.bam ANNOTATIONS_FILE=/home/daney/projects/Drop_seq/for_dropseq_bootcamp/metadata_dropseq_hg19mm10/genes.refFlat
```

#### 12.ビーズのエラー（置換）を修正 

```unix
DetectBeadSubstitutionErrors I= star_gene_exon_tagged.bam O=my_clean_substitution.bam OUTPUT_REPORT=my_clean.substitution_report.txt
```

#### 13. ビーズのエラー（合成）を修正

```unix
DetectBeadSynthesisErrors I=my_clean_substitution.bam O=my_clean.bam REPORT=my_clean.indel_report.txt OUTPUT_STATS=my.synthesis_stats.txt SUMMARY=my.synthesis_stats.summary.txt PRIMER_SEQUENCE=AAGCAGTGGTATCAACGCAGAGTAC
```

#### 14. 発現量等の表を作成

```unix
DigitalExpression I= my_clean.bam O= my_clean.dge.txt.gz SUMMARY= my_clean.dge.summary.txt NUM_CORE_BARCODES=XXXX
#my.synthesis_stats.summary.txtに記載されているNO_ERRORの数を参照して、抜き出すバーコード数（NUM_CORE_BARCODESで指定する値）を決定

```

#### 15. Read数の変化から発現している細胞数を見積もる

```unix
# knee pointの出し方の一例（自作スクリプト使用）
perl convert_detection_knee_point_data.pl my_clean.dge.summary.txt
```

#### 16. 細胞として抽出したバーコードの発現量表を作成

```unix
DigitalExpression I=my_clean.bam O=my_clean.dge.extracted.txt.gz SUMMARY=my_clean.dge.extracted.summary.txt  NUM_CORE_BARCODES=XXX
#NUM_CORE_BARCODESにKnee pointを指定
```

#### 17. 細胞を種ごとにカウントする(1)

```unix
FilterBam INPUT=my_clean.bam OUTPUT=human.bam REF_SOFT_MATCHED_RETAINED=hg19

FilterBam INPUT=my_clean.bam OUTPUT=mouse.bam REF_SOFT_MATCHED_RETAINED=mm10
```

#### 17. 細胞を種ごとにカウントする(2)

```unix
DigitalExpression I=my_clean.bam O=my_clean.dge.extracted.txt.gz SUMMARY=my_clean.dge.extracted.summary.txt  NUM_CORE_BARCODES=XXX
#NUM_CORE_BARCODESにKnee pointを指定
```

#### 16. 細胞として抽出したバーコードの発現量表を作成

細胞として抽出されたバーコードのリストを作成
```unix
awk -F "\t" '{print $1}' my_clean.dge.extracted.summary.txt > barcode_list.txt
#headerなどはいらないのでbarcode_list.txtファイル中から削除
```
種ごとに遺伝子発現量のテーブルを作成
```unix
DigitalExpression I=human.bam O=human.dge.txt.gz SUMMARY=human.dge.summary.txt CELL_BC_FILE=barcode_list.txt

DigitalExpression I=mouse.bam O=mouse.dge.txt.gz SUMMARY=mouse.dge.summary.txt CELL_BC_FILE=barcode_list.txt
```


#### 17. 細胞を種ごとにカウントする(3)

```unix
```
