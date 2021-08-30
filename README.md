# Probiotics-relieve-human-asthma-project

## 1. For quality control and de_hosting: KneadData and Bowtie2
### klab_metaqc qc -s data.list -t human -j 10 -o qc_result_folder_rmhuman -f # klab_metaqc is a simple process combining the above two tools

## 2. For assembling: Megahit
### ls -d *| parallel -j 5 megahit -m 0.5 --min-contig-len 200 -t 10 --out-dir {}_Output_ass --out-prefix {} -1 {}/{}.rmhost.r1.fq.gz -2 {}/{}.rmhost.r2.fq.gz ::: * # parallel code

## 3. For genome Binning: MaxBin2, MetaBAT2 and Concoct
### ls -d Sample* | parallel -j 3 metawrap binning -a Assembly_result/{}_Output_ass/{}_Output_contigs_min2000.fasta -o {}/{}.bins -t 16 --metabat2 --maxbin2 --concoct -l 2000 {}/{}.rmhost_1.fastq {}/{}.rmhost_2.fastq # parallel code

## 4. For bin_refinement: MetaWRAP
### ls -d *bin | parallel -j 5 metawrap bin_refinement -t 5 -m 200 -c 50 -x 10 -A {}/maxbin2_bins -B {}/metabat2_bins -o {}/metawrap # parallel code

## 5. For genome de_replicate: dRep
### cat bins_80_5_checkm.summary | awk 'BEGIN{OFS=",";}{print $1 ".fa", $2, $3}' > ./bins_80_5_checkm_info
sed 's/bin.fa,/genome,/g' bins_80_5_checkm_info > bins_80_5_checkm_info.csv
dRep dereplicate dereplicate_result -pa 0.95 -sa 0.95 -g ./bins_80_5/* -p 32 --genomeInfo bins_80_5_checkm_info.csv

## 6. For gene prediction: prodigal 
### ls -d all_rename_SGBs/*fa | parallel --plus -j 32 prodigal -i {} -a all_rename_SGBs_faa/{/.}.faa -f gff -o all_rename_SGBs_faa/{/.}.gff -q -d all_rename_SGBs_faa/{/.}.ffn

## 7. For species annotation
### cat all_rename_SGBs_faa/*faa > all_combine.faa
diamond blastp --threads 32 --max-target-seqs 10 --db  /nvmessdnode3/opt/database/uniport/uniprot_trembl_sport.dmnd --query all_combine.faa --outfmt 6 qseqid sseqid stitle pident qlen slen length mismatch gapopen qstart qend sstart send evalue bitscore --out all_combine.dia
mkdir all_rename_SGBs_speci_out
ls -d all_rename_SGBs_faa/*faa | parallel -j 30 specI.py {.}.ffn {} 2 {/.} all_rename_SGBs_speci_out
cat all_rename_SGBs_speci_out/S*/*results | cut -f 1,4,5 > all_rename_SGBs_speci_out.summary

## 8. For reads mapping
### cat sample_names | parallel -j 10 bbmap.sh ref=all_rename_bins.fa in1=./all_sample/{}/{}_paired_1.fastq.gz in2=./all_sample/{}/{}_paired_2.fastq.gz -Xmx50g threads=10 unpigz=T out=bbmap_out/{}.sam minid=0.95 idfilter=0.95


## 9. Phage assembly: VIBRANT and DeepVirFinder
### ls -d *fasta | parallel -j 10 VIBRANT_run.py -i {} -t 20 
### parallel -j 10 python /userdata/data_jinh/.method/script.links/181120rename_seq_inputname_retain_rawnames.py {}/{/}_Output_contigs_min2000.fasta {}/{/}.rename.fasta {/}
