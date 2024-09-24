# Comparison of taxonomic resolution ability between Full-length 16S rRNA gene (long reads) and 16S V3V4 gene (short reads)    
## Publication 
https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-024-10213-5
![image](https://github.com/user-attachments/assets/efb19b5b-0250-4512-a54e-826da2da901a)
## Collecdata from SRA toolkit    
**1. Trace NCBI**  https://www.ncbi.nlm.nih.gov/bioproject/PRJNA933120    
**2. Create the directory contents long and short reads data**
```bash
mkdir -p 16S_analysis/input/Pacbio 16S_analysis/input/illumina    
```



**3. Download data**    
- 4 Ilumina feaces samples SRR23380954, SRR23380955, SRR23380956, SRR23380957    
- 4 Pacbio feaces samplesSRR23380883, SRR23380890, SRR23380891, SRR23380892
```bash
prefetch SRR23380954 SRR23380955 SRR23380956 SRR23380957 SRR23380883 SRR23380890 SRR23380891 SRR23380892
```

**4. Convert .SRA file into fastq flie**    
```bash
fasterq-dump --outdir /home/hp/16S_analysis/input/Illumina --split-files SRR23380954
fasterq-dump --outdir /home/hp/16S_analysis/input/Illumina --split-files SRR23380955
fasterq-dump --outdir /home/hp/16S_analysis/input/Illumina --split-files SRR23380956
fasterq-dump --outdir /home/hp/16S_analysis/input/Illumina --split-files SRR23380957
fasterq-dump --outdir /home/hp/16S_analysis/input/Pacbio --split-files SRR23380883
fasterq-dump --outdir /home/hp/16S_analysis/input/Pacbio --split-files SRR23380890
fasterq-dump --outdir /home/hp/16S_analysis/input/Pacbio --split-files SRR23380891
fasterq-dump --outdir /home/hp/16S_analysis/input/Pacbio --split-files SRR23380892
````

## QC downloaded samples and statistic
**1. Install Seqkit**  
```bash
conda install -c bioconda seqkit
``` 
**2. Install csvtk**    
```bash
conda install -c bioconda csvtk    
```
**3. Process QC and statistics for Illumina samples**    
```bash
for file in /home/hp/16S_analysis/input/illumina/*.fastq; do
    sampleID=$(basename "$file" .fastq)
    seqkit fx2tab -j 8 -q --gc -l -H -n -i "$file" | \
    svtk mutate2 -t -n sample -e "\"$sampleID\"" > "/home/hp/16S_analysis/fastqc/illumina/${sampleID}.seqkit.readstats.tsv"
    seqkit stats -T -j 8 -a "$file" | \
    csvtk mutate2 -t -n sample -e "\"$sampleID\"" > "/home/hp/16S_analysis/fastqc/illumina/${sampleID}.seqkit.summarystats.tsv"
done
```
**4. Process QC and statistics for Pacbio samples**   
```bash
for file in /home/hp/16S_analysis/input/pacbio/*.fastq; do
    sampleID=$(basename "$file" .fastq)
    
    # Xử lý thống kê read với seqkit và csvtk
    seqkit fx2tab -j 8 -q --gc -l -H -n -i "$file" | \
    csvtk mutate2 -t -n sample -e "\"$sampleID\"" > "/home/hp/16S_analysis/fastqc/pacbio/${sampleID}.seqkit.readstats.tsv"
    # Xử lý thống kê tổng hợp với seqkit và csvtk
    seqkit stats -T -j 8 -a "$file" | \
    csvtk mutate2 -t -n sample -e "\"$sampleID\"" > "/home/hp/16S_analysis/fastqc/pacbio/${sampleID}.seqkit.summarystats.tsv"
done
```
## Long Reads 16S hifi Pacbio Data processing   
### Import to QIIME 2
**1. Generate manifest file***
```bash
nano manifest1.csv
```
*plaintext of manifest.csv*
```bash
sample-id,absolute-filepath,direction
SRR23380883,/home/hp/16S_analysis/input/pacbio/processed_data/SRR23380883.fastq,forward
SRR23380890,/home/hp/16S_analysis/input/pacbio/processed_data/SRR23380890.fastq,forward
SRR23380891,/home/hp/16S_analysis/input/pacbio/processed_data/SRR23380891.fastq,forward
SRR23380892,/home/hp/16S_analysis/input/pacbio/processed_data/SRR23380892.fastq,forward
````

**2. Import data to QIIME2**
```bash
conda activate qiime2-amplicon-2024.5
qiime tools import \
  --type 'SampleData[SequencesWithQuality]' \
  --input-path manifest.csv \
  --output-path long_reads_demux1.qza \
  --input-format SingleEndFastqManifestPhred33
```

***3. Inspect the imported data**   
```bash
qiime demux summarize \
  --i-data long_reads_demux1.qza \
  --o-visualization long_reads_demux1.qzv
```

### Denoising with DADA2 plugin
```bash
qiime dada2 denoise-ccs \
  --i-demultiplexed-seqs long_reads_demux1.qza \
  --o-table dada2-ccs_table.qza \
  --o-representative-sequences dada2-ccs_rep.qza \
  --o-denoising-stats dada2-ccs_stats.qza \
  --p-min-len 1000 \
  --p-max-len 1600 \
  --p-max-ee 3 \
  --p-front 'AGRGTTYGATYMTGGCTCAG' \
  --p-adapter 'RGYTACCTTGTTACGACTT' \
  --p-n-threads 8
```

### Remove chimeric ASVs
```bash
qiime vsearch uchime-denovo \
  --i-table dada2-ccs_table.qza \
  --i-sequences dada2-ccs_rep.qza \
  --o-chimeras chimeras.qza \
  --o-nonchimeras nonchimeras.qza \
  --o-stats chimera-stats.qza
```

### Generate SILVA database   
**1. Download SILVA database**
```bash
wget https://data.qiime2.org/2022.2/common/silva-138-99-tax.qza
wget https://data.qiime2.org/2022.2/common/silva-138-99-seqs.qza
```

**Update scikit-learn to Match Classifier Version**
```bash
pip install scikit-learn==0.24.1
```
**2. Filtering Chimeric ASVs**
```bash
qiime feature-classifier classify-sklearn \
  --i-classifier silva-138-99-nb-classifier.qza \
  --i-reads nonchimeras.qza \
  --p-confidence 0.97 \
  --o-classification taxonomy.vsearch.qza
```

**Visualize taxonomy table**
```bash
qiime metadata tabulate \
  --m-input-file taxonomy.vsearch.qza \
  --o-visualization taxonomy.vsearch.qzv
````

**3. Update feature table**
```bash
qiime feature-table filter-features \
  --i-table dada2-ccs_table.qza \
  --m-metadata-file taxonomy.vsearch.qza \
  --o-filtered-table filtered_table.qza
```

**4. Generate taxa barplot**
**4.1. Generate metadata file**
```bash
nano metadata1.tsv
```
*plaintext*
```bash
sample-id       group   read_type
SRR23380883     pacbio  long
SRR23380890     pacbio  long
SRR23380891     pacbio  long
SRR23380892     pacbio  long
```

**4.2. Generate taxa barplot**
```bash
qiime taxa barplot \
  --i-table filtered_table.qza \
  --i-taxonomy taxonomy.vsearch.qza \
  --m-metadata-file metadata1.tsv \
  --o-visualization long_reads_taxa-bar-plots.qzv
```

### Generate phylo tree
**1. Filtering non chimeric ASV**
```bash
qiime feature-table filter-seqs \
  --i-data dada2-ccs_rep.qza \
  --i-table long_reads_filtered_table.qza \
  --o-filtered-data no-chimera-rep-seqs.qza
```

**2. Generate phylo tree**
```bash
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences no-chimera-rep-seqs.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
```

**3. Visualize phylo tree
```qiime tools export \
  --input-path rooted-tree.qza \
  --output-path exported-rooted-tree
```
*View with iTOL*


### Generate rarefaction curve
```bash
qiime diversity alpha-rarefaction \
  --i-table long_reads_filtered_table.qza \
  --i-phylogeny rooted-tree.qza \
  --p-max-depth 8500 \
  --m-metadata-file metadata1.tsv \
  --o-visualization long_reads_alpha-rarefaction.qzv
```





