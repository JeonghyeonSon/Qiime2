# Using VCL computer in NCSU

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


## 5. Upload files from my computer to VCL (using your windown terminal/ not your ubuntu environment)
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



## 9. Impoort the data
```
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path manifest.tsv \
  --output-path paired-end-demux.qza \
  --input-format PairedEndFastqManifestPhred33V2
```

## 10. 
