# strain-tracking-in-the-microbiome

Prerequisite

1. Define species ANI, 95% threshold
2. Universal bacteria genes, species core genome, species accessory genome
3. Introduce key species: E. coli, E. faecalis, K. pneumoniae, M. gnavus, B. longum, V. parvula

Introduce study

1. Original idea: obesity and weight gain
2. Meconium paper
3. Other papers: antibiotics, C. difficile
4. Introduce subjects 104 and 285

Read-based approaches

1. Introduce E. coli species-specific marker genes from StrainPhlAn
2. Index E. coli genes with bwa-mem
3. Align 1d samples to E. coli genes with bwa-mem
4. Compare the set of . coli genes in each sample
5. Investigate SNVs in E. coli gene alignments

Assembly-based approaches

1. Review E. coli genome. Genome size, GC content, number of genes.
2. Tour of contigs and annotation files. Contig length (.fna), gene location on contigs (inference.tsv), gene annotations (.ffn).
3. Taxonomic annotation of contigs (Kraken assignment file). Look at species from 1d and 1m samples.
4. Use vsearch to align ORFs between samples.
5. BONUS: Point out phage tail proteins
