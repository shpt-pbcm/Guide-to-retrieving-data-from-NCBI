# Guide to Download Raw Data (SRR) from NCBI BioProject

This repository provides a step-by-step guide to retrieve raw sequencing data (SRR) from an NCBI BioProject using **EDirect** and **SRA Toolkit** on Ubuntu/Linux systems.

---

## I. Download Raw Data (SRR) from a BioProject

### 1. Retrieve Run (SRR) list using EDirect

#### 1.1 Install EDirect
```bash
sudo apt-get update
sudo apt-get install -y edirect```

1.2 Export runinfo.csv from SRA
bash
Sao chép mã
esearch -db sra -query [BioProject_number] | efetch -format runinfo > runinfo.csv
Example

bash
Sao chép mã
esearch -db sra -query PRJNA719415 | efetch -format runinfo > runinfo_PRJNA719415.csv
1.3 Extract SRR accessions
bash
Sao chép mã
cut -d',' -f1 runinfo.csv | tail -n +2 > runs.txt
head runs.txt
wc -l runs.txt
Example

bash
Sao chép mã
cut -d',' -f1 runinfo_PRJNA719415.csv | tail -n +2 > runs.txt
runs.txt contains the list of SRR accessions for download.

2. Download raw data using SRA Toolkit
2.1 Install SRA Toolkit
bash
Sao chép mã
conda install -c bioconda sra-tools
Verify installation:

bash
Sao chép mã
prefetch -h | head
fasterq-dump -h | head
2.2 Create working directories
bash
Sao chép mã
mkdir sra_download fastq
3. Download .sra files and convert to FASTQ
3.1 Download .sra files
bash
Sao chép mã
prefetch --option-file runs.txt --output-directory sra_download
Check downloaded files:

bash
Sao chép mã
find sra_download -name "*.sra" | head
3.2 Convert .sra to FASTQ
bash
Sao chép mã
fasterq-dump --split-files --threads 8 --outdir fastq $(cat runs.txt)
II. Common Issues and Troubleshooting
1. BioProject returns no Run data
Symptoms:

runinfo.csv is empty or contains only a header

runs.txt has 0 lines

1.1 Inspect runinfo.csv
bash
Sao chép mã
ls -lh runinfo.csv
head -n 5 runinfo.csv
tail -n 5 runinfo.csv
If the file contains errors, HTML, or no data → EDirect query failed.

1.2 Verify the Run column position
bash
Sao chép mã
head -n 1 runinfo.csv
Automatically extract the correct Run column:

bash
Sao chép mã
awk -F',' '
NR==1{
  for(i=1;i<=NF;i++) if($i=="Run") col=i
  if(!col){print "Run column not found"; exit}
  next
}
NR>1 && $col!="" {print $col}
' runinfo.csv > runs.txt
1.3 Fix Windows CRLF line endings
bash
Sao chép mã
sed -i 's/\r$//' runinfo.csv
1.4 Force BioProject-specific query
bash
Sao chép mã
esearch -db sra -query "PRJNA719415[BioProject]" | efetch -format runinfo > runinfo.csv
1.5 Retrieve SRR accessions without runinfo
bash
Sao chép mã
esearch -db sra -query "PRJNA719415[BioProject]" \
| esummary \
| xtract -pattern DocumentSummary -element Runs \
| tr ',' '\n' \
| sed 's/^[[:space:]]*//;s/[[:space:]]*$//' \
| grep '^SRR' > runs.txt
2. Check whether a BioProject contains raw SRA data
2.1 Check SRA availability
bash
Sao chép mã
esearch -db sra -query "PRJNA719415[BioProject]" | wc -l
0 → No SRA data

>0 → SRA data available

2.2 Check BioProject–SRA linkage
bash
Sao chép mã
esearch -db bioproject -query PRJNA719415 | elink -target sra | wc -l
2.3 Check for genome assemblies
bash
Sao chép mã
esearch -db assembly -query PRJNA719415 \
| esummary \
| xtract -pattern DocumentSummary -element AssemblyAccession
If output contains GCA_XXXXXXXX.X → Assembly data exists.
