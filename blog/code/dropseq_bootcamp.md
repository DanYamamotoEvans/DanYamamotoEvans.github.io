## Command for Drop-seq analysis using TTCK server (King)


### Preparation

#### Login to TTCK server

```unix
ssh USER_NAME@cs0.bioinfo.ttck.keio.ac.jp
```

#### Login to cluster server (king)

```unix
ssh king
```

```unix
```


#### 1. Dropseq toolsのインプット用にファイル形式を変換（fastq to bam) !!TEST!!
```unix
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

