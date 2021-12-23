## Annotation

In this file tree overview of the annotation results, log files are ignored. 

`results/annotation/${sample}/`  
`├── final_contigs.cmscan`  
`├── final_contigs.faa`  
`├── final_contigs.features.gff`: Position of Open Reading Frames (ORFs), which were predicted by Prodigal.  
`├── final_contigs.ffn`: Nucleotide sequences of ORFs.   
`├── final_contigs.gff`  
`│`  
`├── gene_counts.tsv`:   Counts of ORFs, also in other samples.  
`│`  
`├── ${sample}.emapper.annotations`: Predicted functional groups to which a query (in this case, an ORFs) correspond.  
`│                                `  Fields include query (= predicted ORF), eggNOG\_OGs, COG\_category, GOs, multiple KEGG categories and Pfam.  
`├── ${sample}.emapper.seed_orthologs`  
`├── ${sample}.pfam.out`  
`├── infernal.log`  
`│`  
`│   `The following glob patterns build filenames together:   
`├── enzymes.*`  
`├── kos.*`  
`├── modules.*`  
`├── pathways.*`  
`│   `Files corresponding to these first four patterns are based upon the KEGG database.  
`│`  
`├── pfam.*`  
`│`  
`├──          *.parsed.tsv`: Coupling of ORFs to KEGG database or protein families.  
`├──          *.parsed.counts.tsv`: Raw counts of KEGG entries or protein families.  
`│`  
`│            `Normalized counts, performed by edgeR:  
`├──          *.parsed.TMM.tsv`: edgeR's native normalization method  
`├──          *.parsed.RLE.tsv`: based upon DESeq  
`└──          *.parsed.CSS.tsv`: based upon metagenomeSeq  
