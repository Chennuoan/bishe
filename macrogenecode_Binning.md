#载入环境 export PATH="/root/binning/software/BWA/bwa-0.7.17:$PATH"
export PATH="/root/binning/software/samtools/samtools-1.13:$PATH" export
PATH="/root/binning/software/metabat2/metabat:$PATH"
export PATH="/root/software/miniconda3/bin:$PATH"

#創建自己工作目錄 cd /root/WORKSPACE_Stu/09.Binning mkdir \$(whoami) cd
$(whoami)
mkdir clean_data
ln -fs /root/WORKSPACE/09.Binning/clean_data/* /root/WORKSPACE_Stu/09.Binning/$(whoami)/clean_data/.
mkdir workspace cd workspace #bwa index bwa index -a is
../clean_data/final_Scaftigs.fasta -p index #bwa mem bwa mem -t 10 -a
index ../clean_data/S.clean_R1.fastq.gz
../clean_data/S.clean_R2.fastq.gz \> S_final.sam bwa mem -t 10 -a index
../clean_data/B.clean_R1.fastq.gz ../clean_data/B.clean_R2.fastq.gz \>
B_final.sam

补充-复制已跑结果： (1) ln -fs
/root/WORKSPACE_Stu/09.Binning/root/workspace/S_final.sam
/root/WORKSPACE_Stu/09.Binning/$(whoami)/workspace/S_final.sam
(2) ln -fs /root/WORKSPACE_Stu/09.Binning/root/workspace/B_final.sam  /root/WORKSPACE_Stu/09.Binning/$(whoami)/workspace/B_final.sam

# 格式转换：SAM \> BAM

time samtools view -@ 16 -b -S S_final.sam -o S_final.bam time samtools
view -@ 16 -b -S B_final.sam -o B_final.bam rm S_final.sam B_final.sam

补充-复制已跑结果： (1) ln -fs
/root/WORKSPACE_Stu/09.Binning/root/workspace/S_final.bam
/root/WORKSPACE_Stu/09.Binning/$(whoami)/workspace/S_final.bam
(2) ln -fs /root/WORKSPACE_Stu/09.Binning/root/workspace/B_final.bam  /root/WORKSPACE_Stu/09.Binning/$(whoami)/workspace/B_final.bam

#BAM排序 samtools sort -@ 16 -l 9 -O BAM S_final.bam -o
S_final.sorted.bam samtools sort -@ 16 -l 9 -O BAM B_final.bam -o
B_final.sorted.bam

补充-复制已跑结果： (1) ln -fs
/root/WORKSPACE_Stu/09.Binning/root/workspace/S_final.sorted.bam
/root/WORKSPACE_Stu/09.Binning/$(whoami)/workspace/S_final.sorted.bam
(2) ln -fs /root/WORKSPACE_Stu/09.Binning/root/workspace/B_final.sorted.bam /root/WORKSPACE_Stu/09.Binning/$(whoami)/workspace/B_final.sorted.bam

#計算Contig深度 time jgi_summarize_bam_contig_depths --outputDepth
final.depth.txt \*final.sorted.bam \# metabat2 mkdir bins metabat2 -i
../clean_data/final_Scaftigs.fasta -o bins/metabat2 --numThreads 15
--unbinned -a final.depth.txt --sensitive

#checkm source /root/software/miniconda3/etc/profile.d/conda.sh conda
activate binning checkm lineage_wf -t 12 -x fa --nt --tab_table -f
metabat.lineage_wf.txt bins check_temp_out --reduced_tree

补充-复制已跑结果： (1) ln -fs
/root/WORKSPACE_Stu/09.Binning/root/workspace/metabat.lineage_wf.txt
/root/WORKSPACE_Stu/09.Binning/\$(whoami)/workspace/metabat.lineage_wf.txt

#good bins python3 /root/binning/software/filter_bins_by_checkM.py -d
bins -o good_bins -i metabat.lineage_wf.txt -x ".fa" -c 50 -n 20 -s 0
