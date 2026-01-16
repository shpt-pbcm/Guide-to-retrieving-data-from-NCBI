# Guide to Download Raw Data (SRR) from NCBI BioProject

This repository provides a step-by-step guide to retrieve raw sequencing data (SRR) from an NCBI BioProject using **EDirect** and **SRA Toolkit** on Ubuntu/Linux systems.

---

## I. Download Raw Data (SRR) from a BioProject

### 1. Retrieve Run (SRR) list using EDirect

#### 1.1 Install EDirect
```bash
sudo apt-get update
sudo apt-get install -y edirect
```

#### 1.2 Export runinfo.csv from SRA
```bash
esearch -db sra -query [BioProject_number] | efetch -format runinfo > [file.tsv/csv]
```

Example
```bash
esearch -db sra -query PRJNA719415 | efetch -format runinfo > runinfo_PRJNA719415.csv
```

#### 1.3 Extract SRR accessions
```bash
cut -d',' -f1 [file.tsv/csv]| tail -n +2 > [file.txt]
head [file.txt]
wc -l [file.txt]
```

Example

```bash
cut -d',' -f1 runinfo_PRJNA719415.csv | tail -n +2 > runs.txt
runs.txt contains the list of SRR accessions for download.
```

### 2. Download raw data using SRA Toolkit
#### 2.1 Install SRA Toolkit
```bash
conda install -c bioconda sra-tools
Verify installation:
prefetch -h | head
fasterq-dump -h | head
```

#### 2.2 Create working directories
```bash
mkdir [folder]
```

Example
```bash
mkdir sra_download
```

### 3. Download .sra files and convert to FASTQ
#### 3.1 Download .sra files
```bash
prefetch --option-file [file.txt] --output-directory [folder]
```

Check downloaded files:
```bash
find [folder] -name "*.sra" | head
```
Example
```bash
prefetch --option-file runs.txt --output-directory sra_download
find sra_download -name "*.sra" | head
```
#### 3.2 Convert .sra to FASTQ
```bash
fasterq-dump --split-files --threads [number]--outdir fastq $([file.txt])
```

Example
```bash
fasterq-dump --split-files --threads 8 --outdir fastq $(cat runs.txt)
```

## II. Common Issues and Troubleshooting
### 1. BioProject returns no Run data
Symptoms:
runinfo.csv is empty or contains only a header
runs.txt has 0 lines
#### 1.1 Inspect runinfo.csv
```bash
ls -lh [file.tsv/csv]
head -n 5 [file.tsv/csv]
tail -n 5 [file.tsv/csv]
```

Example:
```bash
ls -lh runinfo.csv
head -n 5 runinfo.csv
tail -n 5 runinfo.csv
```

If the file contains errors, HTML, or no data → EDirect query failed.

#### 1.2 Verify the Run column position
```bash
head -n 1 [file.tsv/csv]
```

Example:
```bash
head -n 1 runinfo.csv
```

Automatically extract the correct Run column:

```bash
awk -F',' '
NR==1{
  for(i=1;i<=NF;i++) if($i=="Run") col=i
  if(!col){print "Run column not found"; exit}
  next
}
NR>1 && $col!="" {print $col}
' [file.tsv/csv] > [file.txt]
```

Example:
```bash
awk -F',' '
NR==1{
  for(i=1;i<=NF;i++) if($i=="Run") col=i
  if(!col){print "Run column not found"; exit}
  next
}
NR>1 && $col!="" {print $col}
' runinfo.csv > runs.txt
```

#### 1.3 Fix Windows CRLF line endings
```bash
sed -i 's/\r$//' [file.tsv/csv]
```

Example:
```bash
sed -i 's/\r$//' runinfo.csv
```

#### 1.4 Force BioProject-specific query
```bash
esearch -db sra -query "[BioProject_number][BioProject]" | efetch -format runinfo > [file.tsv/csv]
```

Example:
```bash
esearch -db sra -query "PRJNA719415[BioProject]" | efetch -format runinfo > runinfo.csv
```

#### 1.5 Retrieve SRR accessions without runinfo
```bash
esearch -db sra -query "[BioProject_number][BioProject]" \
| esummary \
| xtract -pattern DocumentSummary -element Runs \
| tr ',' '\n' \
| sed 's/^[[:space:]]*//;s/[[:space:]]*$//' \
| grep '^SRR' > [file.txt]
```

Example:
```bash
esearch -db sra -query "PRJNA719415[BioProject]" \
| esummary \
| xtract -pattern DocumentSummary -element Runs \
| tr ',' '\n' \
| sed 's/^[[:space:]]*//;s/[[:space:]]*$//' \
| grep '^SRR' > runs.txt
```

### 2. Check whether a BioProject contains raw SRA data
#### 2.1 Check SRA availability
```bash
esearch -db sra -query "[BioProject_number][BioProject]" | wc -l
```
Example:
```bash
esearch -db sra -query "PRJNA719415[BioProject]" | wc -l
```

0 → No SRA data
>0 → SRA data available

#### 2.2 Check BioProject–SRA linkage
```bash
esearch -db bioproject -query [BioProject_number]| elink -target sra | wc -l
```

Example:
```bash
esearch -db bioproject -query PRJNA719415 | elink -target sra | wc -l
```

#### 2.3 Check for genome assemblies
```bash
esearch -db assembly -query [BioProject_number] \
| esummary \
| xtract -pattern DocumentSummary -element AssemblyAccession
```
Example:
```bash
esearch -db assembly -query PRJNA719415 \
| esummary \
| xtract -pattern DocumentSummary -element AssemblyAccession
```

If output contains GCA_XXXXXXXX.X → Assembly data exists.
