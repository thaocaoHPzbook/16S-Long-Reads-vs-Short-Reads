# Full-length 16S rRNA gene sequencing improves taxonomic resolution
## Publication 
https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-024-10213-5
![image](https://github.com/user-attachments/assets/efb19b5b-0250-4512-a54e-826da2da901a)
## Collecdata from SRA toolkit
1. Trace NCBI https://www.ncbi.nlm.nih.gov/bioproject/PRJNA933120
2. #create the directory to contain long and short reads data
`mkdir -p 16S_analysis/input/Pacbio 16S_analysis/input/illumina`

3. Download data
- 4 Ilumina feaces samples SRR23380954, SRR23380955, SRR23380956, SRR23380957
- 4 Pacbio feaces samplesSRR23380883, SRR23380890, SRR23380891, SRR23380892
`prefetch SRR23380954 SRR23380955 SRR23380956 SRR23380957 SRR23380883 SRR23380890 SRR23380891 SRR23380892`
