# Strain Tracking in the Microbiome

## Data download

1. Download read data
2. Download assembly data
3. Download E. coli reference data

## Introductory lecture

Bacterial species and strains

1. Define species ANI, 95% threshold
2. Universal bacteria genes, species core genome, species accessory genome
3. Introduce key species: E. coli, E. faecalis, K. pneumoniae, M. gnavus, B. longum, V. parvula

The infant growth and microbiome (IGRAM) study

1. Original idea: obesity and weight gain
2. Meconium paper
3. Other papers: antibiotics, C. difficile
4. Introduce subjects 104 and 285

## Read-based approaches to strain tracking

### Introduce E. coli species-specific marker genes from StrainPhlAn

```bash
less reference_data/ecoli_marker_ids.txt
```

```bash
wc -l reference_data/ecoli_marker_ids.txt
```

Try looking up a marker in UniRef. Go to [uniprot.org](https://www.uniprot.org/). Choose the UniRef database and paste in the part of the identifier before the first pipe, for example "UniRef90_P33924".

Try searching for the marker nucleotide sequence on [NCBI BLAST](https://blast.ncbi.nlm.nih.gov/).

### Align reads to StrainPhlAn E. coli marker genes

Index the E. coli marker genes for search.

```bash
bwa-mem2 index ecoli_db/ecoli_markers.fna
```

Align 1d samples to E. coli genes with bwa-mem

```bash
bwa-mem2 mem ecoli_db/ecoli_markers.fna igram1_reads/s104.STL.V01.1.4d_1.fastq.gz > s104.STL.V01.1.4d_1.sam
```

Inspect SAM file and look for matches. Explain fields for edit distance (NM) and mismatched bases (MD).

Count reads aligning to each E. coli marker gene

```bash
cat s104.STL.V01.1.4d_1.sam | grep -v "^@" | cut -f 3 | grep -vF '*' | sort | uniq -c | sort -nr > s104.STL.V01.1.4d_1_ecoli_cts.txt
```

Compare the set of E. coli genes in each sample. Have audience members paste gene list into spreadsheet and find overlap between samples.

Investigate SNVs in E. coli gene alignments. Have audience members find a SNV in a gene with high coverage and verify that the SNP appears in multiple read alignments.

## Assembly-based approaches to strain tracking

1. Review E. coli genome (e.g., [K12](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000005845.2/)). Genome size (~5 Mbases), GC content (~50%), number of genes (~5k).
2. Tour of contigs and annotation files. Contig length (.fna), gene location on contigs (inference.tsv), gene annotations (.ffn).
3. Taxonomic annotation of contigs (Kraken assignment file). Look at contig assignments from 1d and 1m samples. Look at species-level summary.
5. Use vsearch to align ORFs between samples.

The E. coli strain is retained in infant s104.

```bash
vsearch --usearch_global igram1_assembly/s104.STL.V01.1.4d.ffn --db igram1_assembly/s104.STL.V02.1mo.ffn --blast6out vsearch_s104.V01_s104.V02_ffn.txt --id 0.8
```

Contrast with alignment to the 1d sample from infant s285.

```bash
vsearch --usearch_global igram1_assembly/s104.STL.V01.1.4d.ffn --db igram1_assembly/s285.STL.V01.1.4d.ffn --blast6out vsearch_s104.V01_s285.V01_ffn.txt --id 0.8
```

6. Use vsearch to align ORFs to Single Copy Core Genes (SCCGs).

```bash
vsearch --usearch_global igram1_assembly/s104.STL.V01.1.4d.ffn --db ecoli_db/ecoli_sccg.ffn --blast6out s104.STL.V01.1.4d_ecoli_sccg.txt --id 0.9
```

Count hits to SCCGs

```bash
cat s104.STL.V01.1.4d_ecoli_sccg.txt | cut -f 2 | sort | uniq -c
```

We notice that one gene, NP_417757.1, was hit twice. One of the hits does not cover the full gene, and corresponds to KELACCIFFN_254 Alpha operon ribosome binding site. We can see that th other hits cover the full length of the target genes.

7. Use vsearch to align ORFs to StrainPhlAn marker genes.

```bash
vsearch --usearch_global igram1_assembly/s104.STL.V01.1.4d.ffn --db ecoli_db/ecoli_strainphlan.ffn --blast6out vsearch_s104.V01_ecoli_strainplhan.txt --id 0.8
```

```bash
cat vsearch_s104.V01_ecoli_strainplhan.txt| cut -f 2 | sort
```

Compare to the marker genes found by alignment of reads.

8. BONUS: Point out phage proteins

```bash
less -i igram1_assembly/s104.STL.V01.1.4d.ffn
```

End with infant virome paper.
