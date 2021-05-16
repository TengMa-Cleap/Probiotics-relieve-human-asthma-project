# Probiotics-relieve-human-asthma-project

## For quality control and de_hosting: KneadData and Bowtie2
### klab_metaqc qc -s data.list -t human -j 10 -o qc_result_folder_rmhuman -f # klab_metaqc is a simple process combining the above two tools

## For assembling: Megahit
### ls -d *| parallel -j 5 megahit -m 0.5 --min-contig-len 200 -t 10 --out-dir {}_Output_ass --out-prefix {} -1 {}/{}.rmhost.r1.fq.gz -2 {}/{}.rmhost.r2.fq.gz ::: * # parallel code

## For genome Binning: MaxBin2, MetaBAT2 and Concoct
### ls -d Sample* | parallel -j 3 metawrap binning -a Assembly_result/{}_Output_ass/{}_Output_contigs_min2000.fasta -o {}/{}.bins -t 16 --metabat2 --maxbin2 --concoct -l 2000 {}/{}.rmhost_1.fastq {}/{}.rmhost_2.fastq # parallel code

## For bin_refinement: MetaWRAP
### ls -d *bin | parallel -j 5 metawrap bin_refinement -t 5 -m 200 -c 50 -x 10 -A {}/maxbin2_bins -B {}/metabat2_bins -o {}/metawrap # parallel code

## For genome de_replicate: dRep
### cat bins_80_5_checkm.summary | awk 'BEGIN{OFS=",";}{print $1 ".fa", $2, $3}' > ./bins_80_5_checkm_info
sed 's/bin.fa,/genome,/g' bins_80_5_checkm_info > bins_80_5_checkm_info.csv
dRep dereplicate dereplicate_result -pa 0.95 -sa 0.95 -g ./bins_80_5/* -p 32 --genomeInfo bins_80_5_checkm_info.csv
