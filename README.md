# Step by step instructions to benchmark baseline (minimap2) and OpenOmics minimap2 (mm2-fast) on c5.12xlarge and m6i.16xlarge instances of AWS

## Step 1: Download datasets
Download reference genome
```sh
wget https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/references/GRCh38/GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta.gz
```
Link to download HG002 ONT Guppy 3.6.0 dataset:
https://precision.fda.gov/challenges/10/view \
File name: HG002_GM24385_1_2_3_Guppy_3.6.0_prom.fastq.gz

Link to download HG002 HiFi 14kb-15kb dataset:
https://precision.fda.gov/challenges/10/view \
File name: HG002_35x_PacBio_14kb-15kb.fastq.gz

Download HG002 CLR dataset from 
s3://giab/data_indexes/AshkenazimTrio/sequence.index.AJtrio_PacBio_MtSinai_NIST_subreads_fasta_10082018

Download hap2 assembly dataset- 
```sh
wget https://zenodo.org/record/4393631/files/NA24385.HiFi.hifiasm-0.12.hap2.fa.gz
```

## Step 2: Download and compile baseline (minimap2 v0.2.22)
```sh
git clone https://github.com/lh3/minimap2.git -b v2.22
cd minimap2 && make
```

## Step 3: Run baseline minimap2
```sh
./minimap2 -ax [preset] [ref-seq] [read-seq] -t 48 > minimap2output
```
Example command for ONT HG002 dataset:
```sh
./minimap2 -ax map-ont  GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta.gz HG002_ONT.fastq -t 48 > minimap2output
```

## Step 3: Download and compile OpenOmics minimap2 (mm2-fast)
```sh
git clone --recursive https://github.com/lh3/minimap2.git -b fast-contrib-v2.22 mm2-fast-contrib
cd mm2-fast-contrib && make multi
```

## Step 4: Create index for OpenOmics minimap2
```sh
./build rmi.sh path-to-ref-seq <preset flags>
<preset flags> are as follows:
ONT: map-ont
HiFi: map-hifi
CLR: map-pb
Assembly: asm5
```
Example: Create OpenOmics minimap2 index for ONT datasets for GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta.gz
```sh
./build rmi.sh GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta.gz map-ont
```

## Step 5: Run OpenOmics minimap2
```sh
./mm2-fast -ax [preset] [ref-seq] [read-seq] -t [num_threads] > mm2-fastoutput
```
Example command to run HG002 ONT dataset on c5.12xlarge
```sh
./mm2-fast -ax map-ont GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta.gz HG002_ONT.fastq -t 48 > mm2-fastoutput
```
Example command to run HG002 ONT dataset on m6i.16xlarge
```sh
./mm2-fast -ax map-ont GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta.gz HG002_ONT.fastq -t 64 > mm2-fastoutput
```
