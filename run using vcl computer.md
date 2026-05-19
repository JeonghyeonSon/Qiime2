# Using VCL computer in NCSU
You can use a GUI environment on a VCL computer via Remote Desktop Connection (Windows)


## 1. "Miniforge" installation file download

```
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh"
```

## 2. install Miniforge

```
bash Miniforge3-Linux-x86_64.sh -b
```

## 3. Initialize and activate Conda

```
~/miniforge3/bin/conda init
source ~/.bashrc
```


## 4. Search for ‘QIIME 2 download’ on Google, and follow the instructions to install QIIME 2 within a Conda environment.
Please visit qiime 2 website and check the most recent version of qiime2 "https://docs.qiime2.org/2024.10/install/native/"
```
conda env create -n qiime2-amplicon-2024.10 --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml
```


## 5. Upload files from my computer to VCL (using your ubuntu terminal in windows/ not your vcl environment)
Please double check exact location (below code is an example)

```
scp -r "/mnt/d/OneDrive/qiime/yesid_raw data/zr25695.rawdata.250827" json7@vclvm178-152.vcl.ncsu.edu:~/
```
When prompted, type "yes" and enter your portal password - then the files will be uploaded.


## 6. Activate the QIMME2 Environment
```
conda activate qiime2-amplicon-2024.10
```

## 7. Navigate to the data directory (set working directory)
```
cd /home/json7/zr25695.rawdata.250827
```

## 8. Create a Manifest file
```
( \
  echo -e "sample-id\tforward-absolute-filepath\treverse-absolute-filepath"; \
  paste \
    <(ls *R1.fastq.gz | sort -V) \
    <(ls *R2.fastq.gz | sort -V) \
  | awk -v path="$(pwd)/" '{OFS="\t"; print "sample-"NR, path$1, path$2}' \
) > manifest.tsv
```
```manifest.tsv``` should be generated. This file indicates the information of raw sequence files (including location, file name, and mapping forward and reverse read)


## 9. Impoort the data
```
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path manifest.tsv \
  --output-path paired-end-demux.qza \
  --input-format PairedEndFastqManifestPhred33V2
```
```paired-end-demux.qza``` should be generated. This file means your files ```fastq``` are successfully imported in your qiime2 environment.

```qza``` file is a **QIIME Zipped Artifact** and is your data.
```qzv``` file is a  **QIIME Zipped Visualization** and is as visual representation of your data.


## 10. Quality Control
```
qiime demux summarize \
  --i-data paired-end-demux.qza \
  --o-visualization demux.qzv
```
Based on these results, we will determine how many sequences to remove during the **denoising process**.
**How to check the result**:```.qzv``` file can be opened using **QIIME 2 View website** https://view.qiime2.org/
When the report opens, you will see several tabs. Click on the **Interactive Quality Plot** tab.
This plot shows the quality scores (y-axis) at each position along the sequence length (x-axis). The key question you need to address is:

At what position do the quality scores for the **Forward reads (R1)** and **Reverse reads (R2)** begins to drop significantly?
Generally, you should trim the sequences at the position where the median quality score (Phred score) drops below 20-25. Data beyond this point is considered less reliable. Again, if the score is lower than 20-25, that sucks.

## 11. Denoising using DADA2
Step 11 takes **a lot of time**.
(less than 1 hour, it depends on hardware condition)


```bash
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs paired-end-demux.qza \
  --p-trim-left-f [#] \
  --p-trim-left-r [#] \
  --p-trunc-len-f 280 \
  --p-trunc-len-r 280 \
  --o-table table.qza \
  --o-representative-sequences rep-seqs.qza \
  --o-denoising-stats denoising-stats.qza \
  --p-n-threads 0
```

```--p-trim-left-f [#]```: Replace ```[#]``` with your forward primer's length. 17

```--p-trim-left-r [#]```: Replace ```[#]``` with your reverse primer's length. 24

```--p-trunc-len-f 280```: Use the full length of the forward read.

```--p-trunc-len-r 280```: Use the full length of the reverse read.

```--p-n-threads 0```: Use all available computer cores to speed up the process.

**After running this step**, we may have ```table.qza```, ```rep-seqs.qza```, and ```denoising-stats.qza```.
```table.qza``` is an **ASV table**. It is the main output, containing the counts of each microbe (ASV) across all your samples. All downstream diversity analyses are based on this file.

```rep-seqs.qza``` contains the **Representative sequences**. It holds the unique DNA sequence for each ASV present in your feature table. This file is used to assign taxonomy (i.e., to identify the names of the microbes, such as Lactobacillus).

```denoising-stats.qza``` provides the **DADA2 Denoising stats**. It's a report that shows how many sequences from each sample passed through the filtering and denoising steps to become a final ASV. This is very useful for quality control and identifying any problematic or contaminated samples.


## 12. Download Classifier
Visit this link: https://library.qiime2.org/data-resources#naive-bayes-classifiers
Download latest classifier **Silva 138 99% OTUs full-length sequences**

## 13. Generate taxonomy.qza file (Taxonomy assignments)
```
qiime feature-classifier classify-sklearn \
  --i-classifier silva-138-99-nb-classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza \
  --p-n-jobs 0
```
Now, we have two files ```table.qza``` and ```taxonomy.qza```

## 14. Get ASV values at species level
Here is the example for species level. If you need other levels, you can proceed with the codes in 14-1.
First, we need to organize data at species level.
```
qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 7 \
  --o-collapsed-table species-table.qza
```
Second, export data as biom
```
qiime tools export \
  --input-path species-table.qza \
  --output-path exported-species-table
```

Third, convert biom to tsv (final output)

```
biom convert \
  -i exported-species-table/feature-table.biom \
  -o species-table.tsv \
  --to-tsv
```

## 14-1. Get ASV values at each taxonomy level
Phylum (level 2)

# 1. Collapse taxonomy at Phylum level
qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 2 \
  --o-collapsed-table phylum-table.qza

# 2. Export data as BIOM format
qiime tools export \
  --input-path phylum-table.qza \
  --output-path exported-phylum-table

# 3. Convert BIOM to TSV (Final output for Excel/SAS)
biom convert \
  -i exported-phylum-table/feature-table.biom \
  -o phylum-table.tsv \
  --to-tsv

Family (level 5)
# 1. Collapse taxonomy at Family level
qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 5 \
  --o-collapsed-table family-table.qza

# 2. Export data as BIOM format
qiime tools export \
  --input-path family-table.qza \
  --output-path exported-family-table

# 3. Convert BIOM to TSV (Final output for Excel/SAS)
biom convert \
  -i exported-family-table/feature-table.biom \
  -o family-table.tsv \
  --to-tsv

Genus level (level 6)
# 1. Collapse taxonomy at Genus level
qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 6 \
  --o-collapsed-table genus-table.qza

# 2. Export data as BIOM format
qiime tools export \
  --input-path genus-table.qza \
  --output-path exported-genus-table

# 3. Convert BIOM to TSV (Final output for Excel/SAS)
biom convert \
  -i exported-genus-table/feature-table.biom \
  -o genus-table.tsv \
  --to-tsv

Species level (level 7)
# 1. Collapse taxonomy at Species level
qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 7 \
  --o-collapsed-table species-table.qza

# 2. Export data as BIOM format
qiime tools export \
  --input-path species-table.qza \
  --output-path exported-species-table

# 3. Convert BIOM to TSV (Final output for Excel/SAS)
biom convert \
  -i exported-species-table/feature-table.biom \
  -o species-table.tsv \
  --to-tsv


 
