# ------------------------------------------------- init 0 切换至工作目录/加载环境 ---------------------------------------------------

# 每次登入服务器后，需要重新执行第0步

# 0.1 创建项目路径

cd /root/WORKSPACE_Stu/03_Metagenomics 
mkdir -p $(whoami) 
cd $(whoami)

# 0.2 激活操作环境变量

source /root/.bashrc

# -------------------------------------------------- step 1 质控 -------------------------------------------------------------------

# 1.1 低质量碱基过滤

mkdir -p 01.QC/01-trim

trimmomatic PE /db/metagenome/rawfq/S1.R1.fq.gz
/db/metagenome/rawfq/S1.R2.fq.gz\
01.QC/01-trim/S1.pe.R1.fastq 01.QC/01-trim/S1.unpe.R1.fastq\
01.QC/01-trim/S1.pe.R2.fastq 01.QC/01-trim/S1.unpe.R2.fastq\
LEADING:3 TRAILING:3 SLIDINGWINDOW:5:20 MINLEN:50\
-phred33 ILLUMINACLIP:/db/metagenome/rawfq/TruSeq3-PE.fa:2:30:10

trimmomatic PE /db/metagenome/rawfq/S2.R1.fq.gz
/db/metagenome/rawfq/S2.R2.fq.gz\
01.QC/01-trim/S2.pe.R1.fastq 01.QC/01-trim/S2.unpe.R1.fastq\
01.QC/01-trim/S2.pe.R2.fastq 01.QC/01-trim/S2.unpe.R2.fastq\
LEADING:3 TRAILING:3 SLIDINGWINDOW:5:20 MINLEN:50\
-phred33 ILLUMINACLIP:/db/metagenome/rawfq/TruSeq3-PE.fa:2:30:10

# 1.2 去除PCR重复

mkdir -p 01.QC/02-uniq

ls 01.QC/01-trim/S1.pe\* \> 01.QC/02-uniq/S1.input.fastuniq ls
01.QC/01-trim/S2.pe\* \> 01.QC/02-uniq/S2.input.fastuniq

fastuniq -i 01.QC/02-uniq/S1.input.fastuniq -t q\
-o 01.QC/02-uniq/S1.clean.R1.fq\
-p 01.QC/02-uniq/S1.clean.R2.fq\
-c 0

fastuniq -i 01.QC/02-uniq/S2.input.fastuniq -t q\
-o 01.QC/02-uniq/S2.clean.R1.fq\
-p 01.QC/02-uniq/S2.clean.R2.fq\
-c 0

# 1.3 序列与碱基数量统计

mkdir -p 01.QC/03-count

fqBaseSum 01.QC/02-uniq/S1.clean.R1.fq \> 01.QC/03-count/fq.stat.txt
fqBaseSum 01.QC/02-uniq/S1.clean.R2.fq \>\> 01.QC/03-count/fq.stat.txt
fqBaseSum 01.QC/02-uniq/S2.clean.R1.fq \>\> 01.QC/03-count/fq.stat.txt
fqBaseSum 01.QC/02-uniq/S2.clean.R2.fq \>\> 01.QC/03-count/fq.stat.txt
#cat看结果

# 1.4 fastqc 质量报告

mkdir -p 01.QC/04-report/S1.fastqc.out mkdir -p
01.QC/04-report/S2.fastqc.out

fastqc -o 01.QC/04-report/S1.fastqc.out 01.QC/02-uniq/S1.clean.R1.fq
01.QC/02-uniq/S1.clean.R2.fq

fastqc -o 01.QC/04-report/S2.fastqc.out 01.QC/02-uniq/S2.clean.R1.fq
01.QC/02-uniq/S2.clean.R2.fq

# -------------------------------------------------- 基于reads的物种注释 -----------------------------------------------------------

# 1.5 基于reads的注释

mkdir -p 05.Annotation/01-metaphlan

metaphlan 01.QC/02-uniq/S1.clean.R1.fq,01.QC/02-uniq/S1.clean.R2.fq\
--bowtie2out 05.Annotation/01-metaphlan/S1.bowtie2.bz2\
--input_type fastq\
-o 05.Annotation/01-metaphlan/S1.taxa.profile.txt

metaphlan 01.QC/02-uniq/S2.clean.R1.fq,01.QC/02-uniq/S2.clean.R2.fq\
--bowtie2out 05.Annotation/01-metaphlan/S2.bowtie2.bz2\
--input_type fastq\
-o 05.Annotation/01-metaphlan/S2.taxa.profile.txt \# 整合各个样品的结果
merge_metaphlan_tables.py 05.Annotation/01-metaphlan/\*.taxa.profile.txt
\> 05.Annotation/01-metaphlan/merged_taxa_profile.txt

# -------------------------------------------------- step 2 环境的搭建 --------------------------------------------------------------

# 2.2 下载conda安装包

wget -c --no-check-certificate
https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \#
如下载失败可以执行下面的命令 \# cp
/root/WORKSPACE_Stu/03_Metagenomics/Miniconda3-latest-Linux-x86_64.sh .
sh Miniconda3-latest-Linux-x86_64.sh \# yes \# yes source
/home/\$(whoami)/.bashrc

# 2.3 修改\~/.condarc

vim \~/.condarc \# 复制以下5行内容到 condarc, 并使用:wq 命令保存
channels: - r - conda-forge - bioconda - defaults
------------------------------ \#
创建环境，两种方式，指定名称、或者指定路径 conda create -n
`<环境名称>`{=html} \# conda create --prefix `<环境路径>`{=html}

# 使用环境，使用前需要激活

## 激活环境

conda activate `<环境名称>`{=html} \## 退出环境 conda deactivate

#查找想要安装的软件(以 bwa 为例) conda search bwa conda activate
`<环境名>`{=html} conda install bwa=0.7.17

# 2.4 使用源代码安装软件

mkdir -p \~/src && cd \~/src git clone https://github.com/lh3/bwa.git \#
下载失败的话执行：cp -r /root/WORKSPACE_Stu/03_Metagenomics/bwa . cd bwa
make

# -------------------------------------------------- step 3 组装 -------------------------------------------------------------------

# 3.1 megahit 组装

mkdir -p 02.Assembly/01-sample

megahit -t 1 -1 01.QC/02-uniq/S1.clean.R1.fq -2
01.QC/02-uniq/S1.clean.R2.fq\
--out-dir 02.Assembly/01-sample/S1.assembly\
--k-min 35 --k-max 95 --k-step 20 --min-contig-len 500

megahit -t 1 -1 01.QC/02-uniq/S2.clean.R1.fq -2
01.QC/02-uniq/S2.clean.R2.fq\
--out-dir 02.Assembly/01-sample/S2.assembly\
--k-min 35 --k-max 95 --k-step 20 --min-contig-len 500

# 修改下 序列的名称

cp 02.Assembly/01-sample/S1.assembly/final.contigs.fa
02.Assembly/01-sample/S1.final.contigs.fa cp
02.Assembly/01-sample/S2.assembly/final.contigs.fa
02.Assembly/01-sample/S2.final.contigs.fa sed -i 's/\>/\>S1\_/g'
02.Assembly/01-sample/S1.final.contigs.fa sed -i 's/\>/\>S2\_/g'
02.Assembly/01-sample/S2.final.contigs.fa

# 3.2 样品中未利用的reads 进行混合组装

mkdir -p 02.Assembly/02-unmap \# 构建索引 bwa index
02.Assembly/01-sample/S1.final.contigs.fa bwa index
02.Assembly/01-sample/S2.final.contigs.fa

# 比对

bwa mem -k 19 02.Assembly/01-sample/S1.final.contigs.fa\
01.QC/02-uniq/S1.clean.R1.fq 01.QC/02-uniq/S1.clean.R2.fq \>
02.Assembly/02-unmap/S1.unused.sam

bwa mem -k 19 02.Assembly/01-sample/S2.final.contigs.fa\
01.QC/02-uniq/S2.clean.R1.fq 01.QC/02-uniq/S2.clean.R2.fq \>
02.Assembly/02-unmap/S2.unused.sam

# 根据比对结果提取未比对上的序列

awk -F '`\t'`{=tex}'\$3=="\*"{print \$1}'
02.Assembly/02-unmap/S1.unused.sam\|sort\|uniq\|\
seqtk subseq 01.QC/02-uniq/S1.clean.R1.fq - \>
02.Assembly/02-unmap/S1.unmap.R1.fq

awk -F '`\t'`{=tex}'\$3=="\*"{print \$1}'
02.Assembly/02-unmap/S1.unused.sam\|sort\|uniq\|\
seqtk subseq 01.QC/02-uniq/S1.clean.R2.fq - \>
02.Assembly/02-unmap/S1.unmap.R2.fq

awk -F '`\t'`{=tex}'\$3=="\*"{print \$1}'
02.Assembly/02-unmap/S2.unused.sam\|sort\|uniq\|\
seqtk subseq 01.QC/02-uniq/S2.clean.R1.fq - \>
02.Assembly/02-unmap/S2.unmap.R1.fq

awk -F '`\t'`{=tex}'\$3=="\*"{print \$1}'
02.Assembly/02-unmap/S2.unused.sam\|sort\|uniq\|\
seqtk subseq 01.QC/02-uniq/S2.clean.R2.fq - \>
02.Assembly/02-unmap/S2.unmap.R2.fq

# 混合组装

megahit -t 1 -1
02.Assembly/02-unmap/S1.unmap.R1.fq,02.Assembly/02-unmap/S2.unmap.R1.fq\
-2
02.Assembly/02-unmap/S1.unmap.R2.fq,02.Assembly/02-unmap/S2.unmap.R2.fq\
--out-dir 02.Assembly/02-unmap/unmap.assembly\
--k-min 35 --k-max 95 --k-step 20 --min-contig-len 500

cp 02.Assembly/02-unmap/unmap.assembly/final.contigs.fa
02.Assembly/02-unmap/unmap.final.contigs.fa sed -i 's/\>/\>unmap\_/g'
02.Assembly/02-unmap/unmap.final.contigs.fa

# 此软件需要装java

# 3.3 组装结果统计

mkdir -p 02.Assembly/03-stat

pileup.sh in=02.Assembly/02-unmap/S1.unused.sam\
ref=02.Assembly/01-sample/S1.assembly/final.contigs.fa\
out=02.Assembly/03-stat/S1.covstats.txt overwrite=true

pileup.sh in=02.Assembly/02-unmap/S2.unused.sam\
ref=02.Assembly/01-sample/S2.assembly/final.contigs.fa\
out=02.Assembly/03-stat/S2.covstats.txt overwrite=true

# 删除sam文件

rm 02.Assembly/02-unmap/\*.unused.sam

# 统计N50

seqkit stat -a 02.Assembly/01-sample/S1.final.contigs.fa seqkit stat -a
02.Assembly/01-sample/S2.final.contigs.fa

# -------------------------------------------------- step 4 预测/聚类 -------------------------------------------------------------------

# 4.1

mkdir -p 03.Predict/01-orf \# 合并所有的组装结果 cat
02.Assembly/01-sample/\*.final.contigs.fa
02.Assembly/02-unmap/unmap.final.contigs.fa \>
02.Assembly/all.final.contigs.fa

# 4.2 预测基因

prodigal -i 02.Assembly/all.final.contigs.fa\
-o 03.Predict/01-orf/all.gene.gff\
-a 03.Predict/01-orf/all.gene.faa.tmp\
-d 03.Predict/01-orf/all.gene.fna.tmp\
-f gff -g 11 -p meta

# 4.3 Antifam 数据库，过滤假基因

mkdir -p 03.Predict/03-psudogene

hmmsearch --domtblout 03.Predict/03-psudogene/all.antifam.out\
--domE 1e-10\
/db/metagenome/antifam/AntiFam.hmm\
03.Predict/01-orf/all.gene.faa.tmp

awk '{print \$1}' 03.Predict/03-psudogene/all.antifam.out \|grep -v '\#'
\| sort \| uniq \> 03.Predict/03-psudogene/pseudogene_ids.txt seqkit
grep -v -f 03.Predict/03-psudogene/pseudogene_ids.txt
03.Predict/01-orf/all.gene.faa.tmp -o 03.Predict/01-orf/all.gene.faa
seqkit grep -v -f 03.Predict/03-psudogene/pseudogene_ids.txt
03.Predict/01-orf/all.gene.fna.tmp -o 03.Predict/01-orf/all.gene.fna rm
03.Predict/01-orf/\*tmp

# 4.4 基因聚类

mkdir -p 03.Predict/04-clust

cp 03.Predict/01-orf/all.gene.fna 03.Predict/04-clust/all.gene.fna.tmp
cp 03.Predict/01-orf/all.gene.faa 03.Predict/04-clust/all.gene.faa.tmp

awk '{if (\$1\~"\>") { n+=1 print "\>unigene\_"n } else { print } }'
03.Predict/04-clust/all.gene.fna.tmp \>\
03.Predict/04-clust/all.gene.fna

awk '{if (\$1\~"\>") { n+=1 print "\>unigene\_"n } else { print } }'
03.Predict/04-clust/all.gene.faa.tmp \>\
03.Predict/04-clust/all.gene.faa

# 使用mmseq 对基因的核酸序列去冗余

mmseqs createdb 03.Predict/04-clust/all.gene.fna 03.Predict/04-clust/DB

mmseqs linclust 03.Predict/04-clust/DB\
03.Predict/04-clust/DB_clu\
03.Predict/04-clust/tmp\
-k 0 -e 0.001 --min-seq-id 0.95 -c 0.9 --cluster-mode 0

mmseqs createsubdb 03.Predict/04-clust/DB_clu\
03.Predict/04-clust/DB\
03.Predict/04-clust/DB_clu_rep

mmseqs convert2fasta 03.Predict/04-clust/DB_clu_rep\
03.Predict/04-clust/DB_clu_rep.fasta

mmseqs createtsv 03.Predict/04-clust/DB 03.Predict/04-clust/DB
03.Predict/04-clust/DB_clu
03.Predict/04-clust/query_linclust_cluster.tsv cp
03.Predict/04-clust/DB_clu_rep.fasta
03.Predict/04-clust/gene_catalogue.fna

seqkit seq -n 03.Predict/04-clust/gene_catalogue.fna\|seqkit grep -f -
03.Predict/04-clust/all.gene.faa -o
03.Predict/04-clust/gene_catalogue.faa

# -------------------------------------------------- step 5 定量 -------------------------------------------------------------------------

# 5 比对定量

mkdir -p 04.Abundance/align

bwa index 03.Predict/04-clust/gene_catalogue.fna

bwa mem -k 19 03.Predict/04-clust/gene_catalogue.fna\
01.QC/02-uniq/S1.clean.R1.fq 01.QC/02-uniq/S1.clean.R2.fq \>\
04.Abundance/align/S1.sam

bwa mem -k 19 03.Predict/04-clust/gene_catalogue.fna\
01.QC/02-uniq/S2.clean.R1.fq 01.QC/02-uniq/S2.clean.R2.fq \>\
04.Abundance/align/S2.sam

pileup.sh in=04.Abundance/align/S1.sam\
ref=03.Predict/04-clust/gene_catalogue.fna\
out=04.Abundance/S1.covstats.txt overwrite=true

pileup.sh in=04.Abundance/align/S2.sam\
ref=03.Predict/04-clust/gene_catalogue.fna\
out=04.Abundance/S2.covstats.txt overwrite=true

# 丰度计算, ipython代码：

# #--------------------------------------

import pandas as pd df =
pd.read_table('04.Abundance/S1.covstats.txt',index_col=0) sigma =
((df\['Plus_reads'\] + df\['Minus_reads'\])/df\['Length'\]).sum()
abundance = (df\['Plus_reads'\] + df\['Minus_reads'\])/df\['Length'\] /
sigma abundance.to_csv(''04.Abundance/S1.abundance.txt', sep =
'`\t'`{=tex}) \# #--------------------------------------

ipython /root/WORKSPACE_Stu/03_Metagenomics/scripts/cal_abudance.py
04.Abundance 04.Abundance/Abundance.txt

# -------------------------------------------------- step 6 注释 --------------------------------------------------------------------------

# 6.1基于contigs的注释：

# NR物种注释

mkdir 05.Annotation/02-NR diamond blastp -q
03.Predict/04-clust/gene_catalogue.faa -d /db/metagenome/miniNR/nr.dmnd
-o 05.Annotation/02-NR/diamond_results.m8 -e 1e-5 -k 5 --threads 1
blast2lca\
-i 05.Annotation/02-NR/diamond_results.m8\
-o 05.Annotation/02-NR/megan.out\
-f BlastTab -mdb /db/metagenome/megan/megan-map-Feb2022.db

ipython /root/WORKSPACE_Stu/03_Metagenomics/scripts/fix_megan_lca.py
05.Annotation/02-NR/megan.out 05.Annotation/02-NR/gene2tax.txt

# eggNOG功能注释

conda activate eggnog mkdir 05.Annotation/03-eggNOG emapper.py --cpu 1
-i 03.Predict/04-clust/gene_catalogue.faa --output
05.Annotation/03-eggNOG/diamond_eggnog --data_dir /db/metagenome/eggNOG
-m diamond
