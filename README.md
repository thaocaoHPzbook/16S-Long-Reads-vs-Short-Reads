# Full-length 16S rRNA gene sequencing improves taxonomic resolution
## Publication 
https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-024-10213-5
![image](https://github.com/user-attachments/assets/efb19b5b-0250-4512-a54e-826da2da901a)
## Collecdata from SRA toolkit    
**1. Trace NCBI**  https://www.ncbi.nlm.nih.gov/bioproject/PRJNA933120    
**2. Create the directory to contain long and short reads data**
`mkdir -p 16S_analysis/input/Pacbio 16S_analysis/input/illumina`

**3. Download data**    
- 4 Ilumina feaces samples SRR23380954, SRR23380955, SRR23380956, SRR23380957    
- 4 Pacbio feaces samplesSRR23380883, SRR23380890, SRR23380891, SRR23380892    
`prefetch SRR23380954 SRR23380955 SRR23380956 SRR23380957 SRR23380883 SRR23380890 SRR23380891 SRR23380892`
**4. Convert .SRA file into fastq flie
`fasterq-dump --outdir home/hp/-16S_analysis/input/Illumina -split-files SRR23380954
fasterq-dump --outdir home/hp/-16S_analysis/input/Illumina --split-files SRR23380955
fasterq-dump --outdir home/hp/-16S_analysis/input/Illumina --split-files SRR23380956
fasterq-dump --outdir home/hp/-16S_analysis/input/Illumina --split-files SRR23380957
fasterq-dump --outdir home/hp/-16S_analysis/input/Pacbio --split-files SRR23380883
fasterq-dump --outdir home/hp/-16S_analysis/input/Pacbio --split-files SRR23380890
fasterq-dump --outdir home/hp/-16S_analysis/input/Pacbio --split-files SRR23380891
fasterq-dump --outdir home/hp/-16S_analysis/input/Pacbio --split-files SRR23380892`

