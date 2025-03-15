# **QIIME2 Workshop: Analysis of 16S rRNA Amplicon Sequences**

## **1. Introduction**

In this workshop, we will learn how to analyze 16S rRNA sequencing data using **QIIME2**, a bioinformatics pipeline designed for microbiome studies. We will work with amplicon data generated from an **Illumina MiSeq** sequencing platform. QIIME2 allows us to process raw sequencing reads, detect amplicon sequence variants (ASVs), and generate feature tables for further analysis.

---

## **2. Installing QIIME2 using Conda Environment**

QIIME2 can be installed using a **Conda environment**, which allows us to manage dependencies efficiently. We will use a pre-configured **Virtual Machine (VM)** for this workshop, which includes all required software and tools.

### **Installation Steps**:

1. **Check QIIME2 installation documentation:** [QIIME2 Installation Guide](https://docs.qiime2.org/2024.5/install/native/)
2. **Miniconda Installation** *(already installed within our VM)*
3. **Activate the QIIME2 environment in the terminal:**
   ```bash
   qiime2-activate
   ```

---

## **3. Preparing the Required Files**

Before running any analysis, ensure all necessary files are available.

### **Navigate to the working directory:**

```bash
cd /home/vmuser/Desktop/QIIME2
```

### **List the available sequencing files:**

```bash
ls
ls -lh *.fastq.gz  # Lists all FASTQ files with detailed information
```

---

## **4. Importing Sequence Data into QIIME2**

### **Understanding QIIME2 Data Structure**

QIIME2 uses **artifacts** (.qza) to store intermediate results and metadata. To import our sequencing data, we must reference the original **FASTQ** files using a **manifest file** (CSV format), which lists the paths of our raw sequence files.

### **Check the manifest file format:**

```bash
cat manifest.csv
```

### **Import paired-end sequence data:**

```bash
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' \
 --input-path manifest.csv \
 --input-format PairedEndFastqManifestPhred33 \
 --output-path demux.qza
```

### **Visualizing the imported sequences:**

```bash
qiime demux summarize --i-data demux.qza --o-visualization demux.qzv
```

To view the visualization in your browser:

```bash
qiime tools view demux.qzv
```

---

## **5. Denoising and ASV Generation with DADA2**

The **DADA2** algorithm is used for:

- Filtering low-quality sequences
- Correcting sequencing errors
- Removing chimeric sequences
- Merging paired-end reads
- Generating unique **amplicon sequence variants (ASVs)**

### **Run DADA2 for ASV generation:**

```bash
qiime dada2 denoise-paired --i-demultiplexed-seqs demux.qza \
 --p-n-threads 2 \
 --p-trunc-len-f 280 --p-trunc-len-r 220 \
 --o-table table.qza \
 --o-representative-sequences rep-seqs.qza \
 --o-denoising-stats denoising-stats.qza \
 --p-trim-left-f 19 --p-trim-left-r 22
```

### **Explanation of Parameters:**

- `--p-trunc-len-f 280` & `--p-trunc-len-r 220`: Truncate forward (R1) and reverse (R2) reads to remove low-quality bases.
- `--p-trim-left-f 19` & `--p-trim-left-r 22`: Remove primers (19 bp from forward and 22 bp from reverse reads).

### **Output Files:**

- **Denoising statistics**: `denoising-stats.qza`
- **Representative ASVs sequences**: `rep-seqs.qza`
- **Feature table (ASVs count per sample)**: `table.qza`

---

## **6. Exporting and Analyzing Results**

### **Tabulating Denoising Statistics**

```bash
qiime metadata tabulate --m-input-file denoising-stats.qza --o-visualization denoising-stats.qzv
qiime tools view denoising-stats.qzv
```

### **Export statistics in CSV format**

```bash
mkdir denoising-stats
qiime tools export --output-path denoising-stats --input-path denoising-stats.qza
cd denoising-stats
less stats.tsv
```

### **Extract Representative ASVs Sequences**

```bash
mkdir rep_seqs
qiime tools export --output-path rep_seqs --input-path rep-seqs.qza
cd rep_seqs
less dna-sequences.fasta
```

---

## **7. Generating a Readable Feature Table**

Convert the `table.qza` file into a **CSV format** for further analysis.

### **Export Feature Table and Convert to CSV**

```bash
mkdir table_otus
qiime tools export --output-path table_otus --input-path table.qza
biom convert -i table_otus/feature-table.biom -o table_otus.csv --to-tsv
```

Now, the ASV abundance table is available in a **CSV file**, which can be opened in Excel or other programs for further analysis.

---

## **8. Renaming ASV Sequences for Better Readability**

To simplify sequence headers in the **FASTA file**, rename ASVs:

```bash
awk '/^>/ {print ">ASV_" sprintf("%05d", ++i); next} {print}' dna-sequences.fasta > dna-sequences-rename.fasta
```

To rename ASVs in the **feature table CSV file**, use:

```bash
awk 'NR<=2 {print; next} {print "ASV_" sprintf("%05d", ++i) "\t" $2 "\t" $3 "\t" $4 "\t" $5}' table_otus.csv > table_otus_rename.csv
```

---

## **9. Summary**

In this workshop, we have:

1. **Installed and activated QIIME2**
2. **Prepared sequence data and imported it into QIIME2**
3. **Denoised sequences and generated ASVs using DADA2**
4. **Exported and visualized the ASV feature table and representative sequences**
5. **Renamed ASVs for readability**

These steps provide the foundation for downstream microbial community analysis, such as **taxonomic classification**, **diversity analysis**, and **functional annotation**.

For further QIIME2 documentation, visit: [QIIME2 Official Documentation](https://docs.qiime2.org)


---

You have obtained the **representative sequences for each ASV**, along with a table showing the number of sequences clustered under each ASV across your samples.

**You are ready to proceed with analyze representative sequence using the OPU (Operational Phylogenetic Units) approach!**


