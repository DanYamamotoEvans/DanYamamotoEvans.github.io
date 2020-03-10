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
/home/daney/projects/Drop_seq/data/20200308MicroMiSeq
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
mkdir project/dropseq_bootcamp/data

#作成したディレクトリに移動する
cd project/dropseq_bootcamp

#現在の位置を確認する
pwd
# /home/あなたのusername/project/dropseq_bootcamp のように出てくればok。
#今後は基本的にこのディレクトリで作業します。




#シーケンスデータをコピーする　
cp /home/daney/projects/Drop_seq/data/20200308MicroMiSeq/03072020-Ikko* data/.

#Drop seq tools のパスを通す
export PATH=/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0:/home/daney/projects/Drop_seq/for_dropseq_bootcamp/STAR-2.7.3a/STAR/source:$PATH
```





### Drop-seq toolsでデータを処理する

#### 1. Dropseq toolsのインプット用にファイル形式を変換（fastq to bam) 
```unix
java -jar /home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/jar/lib/picard-2.18.14.jar  FastqToSam F1=data/09202019-yachielab-dan_S3_L001_R1_001.fastq  F2=data/09202019-yachielab-dan_S3_L001_R2_001.fastq  O=work.bam SM=example
```





#### 2. Cell barcodeを抜き出してタグ付け 
```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/TagBamWithReadSequenceExtended INPUT=work.bam OUTPUT=work.cb.bam SUMMARY=work.cb.bam_summary.txt BASE_RANGE=1-12 BASE_QUALITY=10 BARCODED_READ=1 DISCARD_READ=FALSE TAG_NAME=XC NUM_BASES_BELOW_QUALITY=1
```





#### 3. UMIを抜き出してタグ付け 
```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/TagBamWithReadSequenceExtended INPUT=work.cb.bam OUTPUT=work.cb.UMI.bam SUMMARY=work.cb.UMI.bam_summary.txt BASE_RANGE=13-20 BASE_QUALITY=10 BARCODED_READ=1 DISCARD_READ=TRUE TAG_NAME=XM NUM_BASES_BELOW_QUALITY=1
```





#### 4. クオリティが低いリードをフィルタリング 
```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/FilterBam TAG_REJECT=XQ INPUT=work.cb.UMI.bam OUTPUT=work.cb.UMI.filtered.bam
```





#### 5. SMART adapter 配列の除去（Start site trimming) 
```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/TrimStartingSequence INPUT=work.cb.UMI.filtered.bam OUTPUT=work.cb.UMI.filtered.trimS.bam OUTPUT_SUMMARY= work.cb.UMI.filtered.trimS_report.txt SEQUENCE= AAGCAGTGGTATCAACGCAGAGTGAATGGG MISMATCHES=0 NUM_BASES=5
```






#### 6. Poly Aをトリミング 
```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/PolyATrimmer INPUT=work.cb.UMI.filtered.trimS.bam OUTPUT=work.cb.UMI.filtered.trimS.trimA.bam OUTPUT_SUMMARY= work.cb.UMI.filtered.trimS.trimA_report.txt MISMATCHES=0 NUM_BASES=6 USE_NEW_TRIMMER=true
```







#### 7. STARでアライメントするためファイル形式を変換(bam to fastq)

```unix
java -jar /home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/jar/lib/picard-2.18.14.jar SamToFastq INPUT=work.cb.UMI.filtered.trimS.trimA.bam FASTQ=work.cb.UMI.filtered.trimS.trimA.fastq
```





#### 8. STARでアライメント

```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/STAR-2.7.3a/source/STAR --runThreadN 1 --genomeDir /home/daney/projects/Drop_seq/for_dropseq_bootcamp/STAR_database/ --readFilesIn work.cb.UMI.filtered.trimS.trimA.fastq --outFileNamePrefix star
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
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/TagReadWithGeneFunction I= merged.bam O=star_gene_exon_tagged.bam ANNOTATIONS_FILE=/home/daney/projects/Drop_seq/for_dropseq_bootcamp/metadata_dropseq_hg19mm10/genes.refFlat
```





#### 12.ビーズのエラー（置換）を修正 

```unix

mkdir tmp

/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/DetectBeadSubstitutionErrors I=star_gene_exon_tagged.bam O=my_clean_substitution.bam OUTPUT_REPORT=my_clean.substitution_report.txt TMP_DIR= /home/[ユーザー名]/project/dropseq_bootcamp/tmp
```





#### 13. ビーズのエラー（合成）を修正

```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/DetectBeadSynthesisErrors I=my_clean_substitution.bam O=my_clean.bam REPORT=my_clean.indel_report.txt OUTPUT_STATS=my.synthesis_stats.txt SUMMARY=my.synthesis_stats.summary.txt PRIMER_SEQUENCE=AAGCAGTGGTATCAACGCAGAGTAC TMP_DIR= /home/[ユーザー名]/project/dropseq_bootcamp/tmp

# my.synthesis_stats.summary.txtの中身を確認する
less my.synthesis_stats.summary.txt

#my.synthesis_stats.summary.txtに記載されているNO_ERRORの数を参照して、メモする。次のステップで値を使います。


```





#### 14. 発現量等の表を作成

```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/DigitalExpression I= my_clean.bam O= my_clean.dge.txt.gz SUMMARY= my_clean.dge.summary.txt NUM_CORE_BARCODES=XXXX
#my.synthesis_stats.summary.txtに記載されているNO_ERRORの数を参照して、抜き出すバーコード数（NUM_CORE_BARCODESで指定する値）を決定

```





#### 15. Read数の変化から発現している細胞数を見積もる

```unix
# knee pointの出し方の一例（自作スクリプト使用）
perl /home/daney/projects/Drop_seq/for_dropseq_bootcamp/convert_for_detect_knee_point.pl my_clean.dge.summary.txt
python /home/daney/projects/Drop_seq/for_dropseq_bootcamp/detect_knee_point.py for_detect_knee_x.txt for_detect_knee_y.txt           

#次のステップで使用するknee pointの値が出力されるので、メモする。
```





#### 16. 細胞として抽出したバーコードの発現量表を作成

```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/DigitalExpression I=my_clean.bam O=my_clean.dge.extracted.txt.gz SUMMARY=my_clean.dge.extracted.summary.txt  NUM_CORE_BARCODES=XXX
#NUM_CORE_BARCODESにKnee pointを指定
```





#### 17. 細胞を種ごとにカウントする(1)

```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/FilterBam INPUT=my_clean.bam OUTPUT=human.bam REF_SOFT_MATCHED_RETAINED=hg19

/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/FilterBam INPUT=my_clean.bam OUTPUT=mouse.bam REF_SOFT_MATCHED_RETAINED=mm10
```





#### 17. 細胞を種ごとにカウントする(2)

```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/DigitalExpression I=my_clean.bam O=my_clean.dge.extracted.txt.gz SUMMARY=my_clean.dge.extracted.summary.txt TMP_DIR= TMP_DIR= /home/[ユーザー名]/project/dropseq_bootcamp/tmp NUM_CORE_BARCODES=XXX
#NUM_CORE_BARCODESにKnee pointを指定
```





#### 18. 細胞として抽出したバーコードの発現量表を作成

細胞として抽出されたバーコードのリストを作成
```unix
awk -F "\t" '{print $1}' my_clean.dge.extracted.summary.txt > barcode_list.txt
#headerなどはいらないのでbarcode_list.txtファイル中から削除

emacs barcode_list.txt
#消去するところは僕が指示するので、ここまでできたら教えてください。


```
種ごとに遺伝子発現量のテーブルを作成
```unix
/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/DigitalExpression I=human.bam O=human.dge.txt.gz SUMMARY=human.dge.summary.txt CELL_BC_FILE=barcode_list.txt TMP_DIR= /home/[ユーザー名]project/dropseq_bootcamp/tmp

/home/daney/projects/Drop_seq/for_dropseq_bootcamp/Drop-seq_tools-2.3.0/DigitalExpression I=mouse.bam O=mouse.dge.txt.gz SUMMARY=mouse.dge.summary.txt CELL_BC_FILE=barcode_list.txt TMP_DIR= /home/[ユーザー名]project/dropseq_bootcamp/tmp
```






#### 19. 細胞を種ごとにカウントする(3)

```unix
#まずはファイルの位置を確認します
pwd
#この場所をメモります。

#次に　Command + T を押し、新しいタブを開いてください。
#以下のようにDesktopに移動します。
cd Desktop

#次にDropseq_bootcampというディレクトリを作ります
mkdir Dropseq_bootcamp

#いま作成したディレクトリに入ります
cd Dropseq_bootcamp

#先ほどのサーバー上の位置から、ファイルをダウンロードします。
scp [ユーザー名]@cs0.bioinfo.ttck.keio.ac.jp:[先ほどのpwdで得られたパス]/"*.txt.gz" .


#メールにて送ったRスクリプトも同じディレクトリに入れましょう。
```


#### 20. Rを用いた解析(1) データ前処理
```unix

#パッケージのダウンロード
library(Seurat)
library(ggplot2)
library(Matrix)
library(tidyverse)


#ない場合はインストールし、再度実行する
#install.packages("パッケージ")
#library(パッケージ)

setwd("~/Desktop/Dropseq_bootcamp/")

## For Dropseq data, read the expression matrix and transform into the sparse matrix format

exp <- read_tsv("my_clean.dge.txt.gz",col_names = TRUE)
exp <- column_to_rownames(exp,var="GENE")
exp.mat <- Matrix(as.matrix(exp))

## Build a Seurat object
## min.cells: Include features detected in at least N cells
## min.features: Include cells where at least N features are detected
seu <- CreateSeuratObject(exp.mat,min.cells = 3,min.features = 200)

## Remove low quality cells which have high mitochondrial gene expresison
mito_pattern <- "^hg19-MT-|^mm10-mt-" #Reagular expression which indicates mitochondrial gene name
rownames(seu)[grep(mito_pattern,rownames(seu))] #Check it
seu[["percent.mito"]] <- PercentageFeatureSet(seu, pattern = mito_pattern)
VlnPlot(seu, features = c("nFeature_RNA", "nCount_RNA", "percent.mito"), ncol = 3)
seu <- subset(seu, subset = nFeature_RNA > 200 & nFeature_RNA < 100000 & percent.mito < 20)

## Human-Mouse plot
exp.count.mat <- seu@assays$RNA@counts
exp.human.mat <- exp.count.mat[grep("^hg19",rownames(exp.count.mat)),]
exp.mouse.mat <- exp.count.mat[grep("^mm10",rownames(exp.count.mat)),]
exp.human.sum <- colSums(exp.human.mat)
exp.mouse.sum <- colSums(exp.mouse.mat)
human.mouse.mat <- data.frame(exp_human=exp.human.sum,exp_mouse=exp.mouse.sum)
ggplot(human.mouse.mat,aes(x=exp_human,y=exp_mouse))+
  geom_point(col="orange",size=1.5)+
  theme_classic()

#Save plot
ggsave("human-mouse.pdf")

## save the object
saveRDS(seu,"step1_seurat_object.rds")

```

##### ここで得られる描画一覧

各セルバーコードに見られるRNAの種類、総数、ミトコンドリア由来の割合
```unix
VlnPlot(seu, features = c("nFeature_RNA", "nCount_RNA", "percent.mito"), ncol = 3)
```
![photo](https://danyamamotoevans.github.io/materials/mito_violin.pdf)


各セルバーコードにおいてHumanとMouseにそれぞれマップした総リード数
 ```unix
ggplot(human.mouse.mat,aes(x=exp_human,y=exp_mouse))+
  geom_point(col="orange",size=1.5)+
  theme_classic()
```
![photo](https://danyamamotoevans.github.io/materials/human-mouse.pdf)




#### 20. Rを用いた解析(2) PCA解析
```unix

## Load the saved object
seu <- readRDS("step1_seurat_object.rds")

## Normalize the expression data
seu <- NormalizeData(seu,normalization.method = "LogNormalize", scale.factor = 10000,verbose = FALSE)

## Find top 3000 highly variable genes that is expected to be useful for cell population clustering
seu <- FindVariableFeatures(seu,selection.method="vst",nfeatures = 3000,verbose = FALSE)
head(VariableFeatures(seu),10) #see top 10 highly variable genes

## Centering and scaling the data so that the variance across cells is 1 using linear transformation
seu <- ScaleData(seu,verbose=FALSE)

## Run principal component analysis to reduce the dimention
seu <- RunPCA(seu,verbose = FALSE)

## PCA is enough to cluster the cells
DimPlot(seu)
FeaturePlot(seu,features = c("mm10-Pclaf","mm10-Prc1"))

saveRDS(seu,"step2_seurat_object.rds")

```

##### ここで得られる描画一覧

PCA解析の出力
 ```unix
DimPlot(seu)DimPlot(seu)
```
![photo](https://danyamamotoevans.github.io/materials/PCA.pdf)


PCA解析の出力に、head(VariableFeatures(seu),10) で見られたTop2の遺伝子をマップしたもの
 ```unix
FeaturePlot(seu,features = c("mm10-Pclaf","mm10-Prc1"))
```
![photo](https://danyamamotoevans.github.io/materials/PCA-feature.pdf)





#### 20. Rを用いた解析(3) UMAP 
```unix

## Load the saved object
seu <- readRDS("step2_seurat_object.rds")

## Detect the informative PC used for dimention reduction
ElbowPlot(seu)
dimlimit <- 8 #Using PC1-8 seems to be fine

## Find k-neighbor cells
seu <- FindNeighbors(seu, reduction = "pca", dims = 1:dimlimit,verbose=FALSE)

## detect clusters with unsupervised way
seu <- FindClusters(seu, resolution = 0.3,verbose=FALSE)

## UMAP
seu <- RunUMAP(seu, reduction = "pca", dims = 1:dimlimit,verbose=FALSE)
DimPlot(seu,reduction = "umap")

## Map human or mouse gene expression on UMAP
norm.count <- seu@assays$RNA@data
norm.count.human <- norm.count[grep("^hg19",rownames(norm.count)),]
norm.count.mouse <- norm.count[grep("^mm10",rownames(norm.count)),]
exp.human.sum <- colSums(norm.count.human)
exp.mouse.sum <- colSums(norm.count.mouse)

seu$huamn_gene_count <- exp.human.sum
seu$mouse_gene_count <- exp.mouse.sum

FeaturePlot(seu,
            features = c("huamn_gene_count","mouse_gene_count"),
            cols = c("grey80","deeppink2"),
            pt.size = 1.5)
            
#Save plot
ggsave("UMAP.pdf")
```

##### ここで得られる描画一覧

次元圧縮に寄与している遺伝子を上から並べ、それぞれどれくらいのばらつきがあるかを示す
 ```unix
ElbowPlot(seu)
```
![photo](https://danyamamotoevans.github.io/materials/PCA-component_std.pdf)



次元圧縮したデータを、UMAPを用いて描画する
 ```unix
DimPlot(seu,reduction = "umap")
```
![photo](https://danyamamotoevans.github.io/materials/umap.pdf)


UMAPの描画にHumanとMouseのgeneカウントをマップする
 ```unix
FeaturePlot(seu,
            features = c("huamn_gene_count","mouse_gene_count"),
            cols = c("grey80","deeppink2"),
            pt.size = 1.5)

```
![photo](https://danyamamotoevans.github.io/materials/umap_human_mouse.pdf)


