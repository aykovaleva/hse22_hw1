# hse22_hw1
de novo genome aasembly homework

#comands on server
ssh ayukovaleva_1@92.242.58.92 -p 5222 -i mykey

ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq                                                     
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq                                                     
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq                                       
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq                                       
htop                                                                                                         
top                                                                                                                                                                         
tmux                                     



seqtk sample -s1001 oil_R1.fastq 5000000 > sub1_paired.fq
seqtk sample -s1001 oil_R2.fastq 5000000 > sub2_paired.fq
seqtk sample -s1001 oilMP_S4_L001_R1_001.fastq 1500000 > sub1_mate_p.fq
seqtk sample -s1001 oilMP_S4_L001_R2_001.fastq 1500000 > sub2_mate_p.fq

mkdir fastqc
mkdir multiqc
ls *.fq | xargs -P 4 -tI{} fastqc -o fastqc {}
multiqc -o multiqc fastqc

platanus_trim sub1_paired.fq sub2_paired.fq 
platanus_internal_trim sub1_mate_p.fq sub2_mate_p.fq

rm *fq

mkdir trimmed
mv *trimmed ./trimmed

mkdir fastqc_trimmed
ls trimmed/* | xargs -P 4 -tI{} fastqc -o fastqc_trimmed {}
mkdir multiqc_trimmed
multiqc -o multiqc_trimmed fastqc_trimmed

time platanus assemble -o Poil -t 8 -m 28 -f trimmed/sub1_paired.fq.trimmed trimmed/sub2_paired.fq.trimmed 2> assemble_paired.log
head Poil_contig.fa 
time platanus scaffold -o Poil -t 8 -c Poil_contig.fa -IP1 trimmed/sub1_paired.fq.trimmed trimmed/sub2_paired.fq.trimmed -OP2 trimmed/sub1_mate_p.fq.int_trimmed trimmed/sub2_mate_p.fq.int_trimmed  2> scaffold.log
time platanus gap_close -o Poil -t 8 -c Poil_scaffold.fa -IP1 trimmed/sub1_paired.fq.trimmed trimmed/sub2_paired.fq.trimmed -OP2 trimmed/sub1_mate_p.fq.int_trimmed trimmed/sub2_mate_p.fq.int_trimmed  2> gapclose.log

rm -r trimmed/

mkdir big_sample
mv ./* big_sample
mkdir small_sample
cd small_sample/

mv ../big_sample/oilMP_S4_L001_R* ../
mv ../big_sample/oil_R* ../

cd small_sample/
seqtk sample -s1001 oil_R1.fastq 3000000 > sub1_paired.fq

seqtk sample -s1001 ../oil_R1.fastq 3000000 > sub1_paired.fq
seqtk sample -s1001 ../oil_R2.fastq 3000000 > sub2_paired.fq

seqtk sample -s1001 ../oilMP_S4_L001_R1_001.fastq 1000000 > sub1_mate_p.fq
seqtk sample -s1001 ../oilMP_S4_L001_R2_001.fastq 1000000 > sub2_mate_p.fq
mkdir fastqc
mkdir multiqc
ls *.fq | xargs -P 4 -tI{} fastqc -o fastqc {}
exit
