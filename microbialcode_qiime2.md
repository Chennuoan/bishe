```
##====数据准备及环境加载==== 
> source /root/software/miniconda3/etc/profile.d/conda.sh 
> conda activate Meta
```

```
#創建自己工作目錄 
> cd /root/WORKSPACE_Stu/11.Meta/qiime2 
> mkdir $(whoami) 
> cd $(whoami)
```

```
#获取原始数据 
> pwd
注：显示/root/WORKSPACE_Stu/11.Meta/qiime2/\[自己帐号ID\]为正确 
> mkdir raw_data 
> ln -fs /root/WORKSPACE/11.Meta/raw_data/* ./raw_data/.
```

```
##====Fastp(質控)==== #质控（各样本） 
> pwd
注：显示/root/WORKSPACE_Stu/11.Meta/qiime2/\[自己帐号ID\]为正确 
> mkdir clean_data


> fastp -i ./raw_data/A1.R1.fq.gz -W 4 -M 20 -5 -o ./clean_data/A1.R1.clean.fq -I ./raw_data/A1.R2.fq.gz -O ./clean_data/A1.R2.clean.fq -h ./clean_data/A1.reads.fastp.html
> fastp -i ./raw_data/A2.R1.fq.gz -W 4 -M 20 -5 -o ./clean_data/A2.R1.clean.fq -I ./raw_data/A2.R2.fq.gz -O ./clean_data/A2.R2.clean.fq -h ./clean_data/A2.reads.fastp.html
> fastp -i ./raw_data/A3.R1.fq.gz -W 4 -M 20 -5 -o ./clean_data/A3.R1.clean.fq -I ./raw_data/A3.R2.fq.gz -O ./clean_data/A3.R2.clean.fq -h ./clean_data/A3.reads.fastp.html
> fastp -i ./raw_data/B1.R1.fq.gz -W 4 -M 20 -5 -o ./clean_data/B1.R1.clean.fq -I ./raw_data/B1.R2.fq.gz -O ./clean_data/B1.R2.clean.fq -h ./clean_data/B1.reads.fastp.html
> fastp -i ./raw_data/B2.R1.fq.gz -W 4 -M 20 -5 -o ./clean_data/B2.R1.clean.fq -I ./raw_data/B2.R2.fq.gz -O ./clean_data/B2.R2.clean.fq -h ./clean_data/B2.reads.fastp.html
> fastp -i ./raw_data/B3.R1.fq.gz -W 4 -M 20 -5 -o ./clean_data/B3.R1.clean.fq -I ./raw_data/B3.R2.fq.gz -O ./clean_data/B3.R2.clean.fq -h ./clean_data/B3.reads.fastp.html
```

```
##====拼接数据==== #去引物 
> pwd
注：显示/root/WORKSPACE_Stu/11.Meta/qiime2/\[自己帐号ID\]为正确 
> mkdir dePrimer
```
#創建sample.list, 寫入當前目錄
sample-id	forward-absolute-filepath	reverse-absolute-filepath
A1	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/A1.R1.clean.fq	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/A1.R2.clean.fq
A2	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/A2.R1.clean.fq	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/A2.R2.clean.fq
A3	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/A3.R1.clean.fq	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/A3.R2.clean.fq
B1	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/B1.R1.clean.fq	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/B1.R2.clean.fq
B2	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/B2.R1.clean.fq	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/B2.R2.clean.fq
B3	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/B3.R1.clean.fq	/root/WORKSPACE_Stu/11.Meta/qiime2/Stu51/clean_data/B3.R2.clean.fq


> qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path sample.list --output-path ./dePrimer/paired-end-demux.qza --input-format PairedEndFastqManifestPhred33V2              #paired-end 双端
> qiime cutadapt trim-paired --i-demultiplexed-sequences ./dePrimer/paired-end-demux.qza --p-cores 2 --p-front-f GTGCCAGCMGCCGCGGTAA --p-front-r CCGTCAATTCMTTTRAGTTT --o-trimmed-sequences ./dePrimer/paired-end-trimmed-seqs.qza   #切除引物
	
> qiime demux summarize --i-data ./dePrimer/paired-end-demux.qza --o-visualization ./dePrimer/paired-end-demux.qzv
> qiime demux summarize --i-data ./dePrimer/paired-end-trimmed-seqs.qza --o-visualization ./dePrimer/paired-end-trimmed-seqs.qzv
#qzv文件，先用xftp导出，再通过[qii](https://view.qiime2.org/) 查看结果
```

```
##====代表序列数据==== ##質控、去冗余 
> pwd
注：显示/root/WORKSPACE_Stu/11.Meta/qiime2/\[自己帐号ID\]为正确 
> mkdir dada2

> qiime dada2 denoise-paired --i-demultiplexed-seqs ./dePrimer/paired-end-trimmed-seqs.qza --p-n-threads 2 --p-trim-left-f 0 --p-trim-left-r 0 --p-trunc-len-f 0 --p-trunc-len-r 0 --o-table ./dada2/feature_table.qza --o-representative-sequences ./dada2/rep-seqs.qza --o-denoising-stats ./dada2/stats.qza    #"0",前面质控过，这里设置为0   "feature_table" qiime2的叫法

> qiime tools export --input-path ./dada2/stats.qza --output-path ./dada2/stats     #下面操作将qza转换，使可以阅读
> qiime tools export --input-path ./dada2/feature_table.qza --output-path ./dada2/feature_table
> qiime tools export --input-path ./dada2/rep-seqs.qza --output-path ./dada2/rep-seqs

> biom convert -i ./dada2/feature_table/feature-table.biom -o ./dada2/otu_table.txt --to-tsv

```

```
##====OTU物种丰度表==== #物种注释 
> pwd
注：显示/root/WORKSPACE_Stu/11.Meta/qiime2/\[自己帐号ID\]为正确 
> mkdir sintax_16S

> qiime feature-classifier classify-sklearn --i-classifier /root/WORKSPACE/11.Meta/database/classifier_16S_greengenes_v13.8.qza --i-reads ./dada2/rep-seqs.qza --o-classification ./sintax_16S/sintax_16S.qza --p-confidence 0.8       #confidence 置信度
> qiime tools export --input-path ./sintax_16S//sintax_16S.qza --output-path ./sintax_16S/sintax_16S

```

```
#OTU物种丰度表 
> pwd
注：显示/root/WORKSPACE_Stu/11.Meta/qiime2/\[自己帐号ID\]为正确

> tail -n +2 ./dada2/otu_table.txt > ./sintax_16S/otu_temp.txt #拼接信息
> cut -f 2 ./sintax_16S/sintax_16S/taxonomy.tsv > ./sintax_16S/tax_temp.txt
> paste ./sintax_16S/otu_temp.txt ./sintax_16S/tax_temp.txt > ./sintax_16S/Table.otu.raw.txt
```