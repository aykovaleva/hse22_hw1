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

# multiqc stats
см. в файлах big_sample and small_sample для основного и дополнительного задания соотверственно

# jupyter code for stats
см. файл contigs_stats_script в основной директории

## comparison of scaffolding results for bigger and smaller sample
# contig stats
общее количество, общая длина, длина самого большого контига
общее количество равно для большой и маленько выборки: 618
суммарная длина контигов для большей выборки больше, но не значительно: 3925834 и 3919421
длина самого большого контига: 179307 и 226585. Неожиданным образом это значение больше для меньшей выборки
# N50
Это значение тоже почему-то для меньше выборки ботше: 47611, 73934

# scaffold stats
общее количество, общая длина, длина самого большого контига
общее количество не равно для большой и маленько выборки: 69 и 75
суммарная длина скаффолдов для большей выборки больше, но не значительно: 3875730 и 3873363.
длина самого большого скаффолда (и Т50): 3834319 и 2754567.Ожиданно это значение больше для большей выборки
Можно сделать вывод, что использование меньшей выборки (3000000 парноконцевых и 1500000 mate pairs) быстрее, но значительно ухудшает результаты

# gaps in scaffolds
количество, суммарная длина
62 и 51 - большая и маленькая выбрка
6825 и 4813 - большая и маленькая выбрка
Вывод: Хотя обычно считается, что чем меньше гэпов, тем лучше, количество гепов плохой индикатор качесва сборки, 
так как он слишком сильно зависит от объема выборки 

# gaps in optimised scaffolds
количество, суммарная длина
10 и 6 - большая и маленькая выбрка
1838 и 1533 - большая и маленькая выбрка
