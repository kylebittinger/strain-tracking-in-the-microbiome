# Strain Tracking in the Microbiome

[Workshop slides](https://docs.google.com/presentation/d/1cl7J-YjPcem-VkbAChEYUusJHhUyyIUVhV91fQ0ArF4/edit?usp=sharing)

[Google Cloud Instructions](https://docs.google.com/presentation/d/1qVzn-xnHzUcfUSqcxrDEyWHAzq5GcjFEH5H0r3qz4Aw/edit?usp=sharing)

## Set up Miniconda enviornment

Installation instructions for [MacOS](https://www.anaconda.com/docs/getting-started/miniconda/install#macos-terminal-installer) or [Linux/WSL](https://www.anaconda.com/docs/getting-started/miniconda/install#linux-terminal-installer)

```bash
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh
```

```bash
bash ~/Miniconda3-latest-MacOSX-arm64.sh
```

Agree to the license, the installation location, and to modifying your login
files.

**Close and re-open your terminal.** When you re-open the terminal, you should
see "(base)" on the left side of your prompt.

```bash
conda list
```

Now, we are going to make a conda environment for our workshop.

```bash
conda create --name strain-tracking
```

You may have to agree to some license terms during the creation of your first
environment.

```bash
conda activate strain-tracking
```

You should now see "(strain-tracking)" on the left side of your prompt.

We will use `wget` to download files from the web.

```bash
conda install wget
```

We will use `bwa-mem2` to align short reads to a reference database.

```bash
conda install -c bioconda bwa-mem2
```

We will use `vsearch` to align gene or contig sequences.

```bash
conda install -c bioconda vsearch
```

## Data download

First, let's make a project directory.

```bash
mkdir workshop-project
```

Now, we're going to move into this directory and **stay there for the rest of
the workshop**.

```bash
cd workshop-project
```

Now we're ready to download the data. First, download the assembly output.

```bash
wget 'https://zenodo.org/records/17781459/files/data_assembly.tar.gz?download=1' -O data_assembly.tar.gz
```

Extract the assembly data from the tar.gz file.

```bash
tar xvzf data_assembly.tar.gz
```

Next, download the *E. coli* reference data.

```bash
wget 'https://zenodo.org/records/17781459/files/refdb.tar.gz?download=1' -O refdb.tar.gz
```

Extract from tar.gz file.

```bash
tar xvzf refdb.tar.gz
```

Download sequence read data. This will take a few minutes.

```bash
wget 'https://zenodo.org/records/17781459/files/s104.STL.V01.1d_1.fastq.gz?download=1' -O s104.STL.V01.1d_1.fastq.gz
wget 'https://zenodo.org/records/17781459/files/s104.STL.V02.1m_1.fastq.gz?download=1' -O s104.STL.V02.1m_1.fastq.gz
wget 'https://zenodo.org/records/17781459/files/s285.STL.V01.1d_1.fastq.gz?download=1' -O s285.STL.V01.1d_1.fastq.gz
wget 'https://zenodo.org/records/17781459/files/s285.STL.V02.1m_1.fastq.gz?download=1' -O s285.STL.V02.1m_1.fastq.gz
```

## Read-based approaches to strain tracking

### Read-by-read alignment

The sequence read files have already been trimmed to remove low-quality
sequences and filtered to remove human DNA.

Select a few reads from one of the files and BLAST at NCBI.

```bash
gunzip -c s285.STL.V01.1d_1.fastq.gz | less
```

Try selecting the option "Sequences from type material" in the box titled
"Choose search set".

Discuss species-level similarity and global vs. local sequence alignment.

### Species-specific marker genes from StrainPhlAn

```bash
less refdb/ecoli_strainphlan.ffn
```

```bash
cat refdb/ecoli_strainphlan.ffn | grep "^>" | wc -l
```

Try looking up a marker in UniRef. Go to [uniprot.org](https://www.uniprot.org/).
Choose the UniRef database in the menu next to the search box. Paste in the
part of the identifier before the first "|" character, for example
"UniRef90_P33924".

Try searching for the marker nucleotide sequence on
[NCBI BLAST](https://blast.ncbi.nlm.nih.gov/).

### Align reads to StrainPhlAn E. coli marker genes

Index the E. coli marker genes for search.

```bash
bwa-mem2 index refdb/ecoli_strainphlan.ffn
```

Align day 1 samples from subject 104 to E. coli marker genes with bwa-mem.

```bash
bwa-mem2 mem refdb/ecoli_strainphlan.ffn s104.STL.V01.1d_1.fastq.gz > s104.V01_strainphlan.sam
```

Inspect SAM file and look for matches. Explain fields for edit distance (NM)
and mismatched bases (MD). Explain bitwise flag for pos/neg strand (16).

```bash
less s104.V01_strainphlan.sam
```

Count reads aligning to each E. coli marker gene.

```bash
cat s104.V01_strainphlan.sam | grep -v "^@" | cut -f 3 | grep -vF '*' | sort | uniq -c | sort -nr > s104.V01_strainphlan_cts.txt
```

Compare the set of E. coli genes in each sample. Paste gene list into
spreadsheet and find overlap between samples.

Investigate single nucleotide varants (SNVs) in E. coli gene alignments. Pick
one SNV in a gene with high coverage and verify that the SNV appears in
multiple read alignments.

## Assembly-based approaches to strain tracking

1. Review E. coli genome (e.g. [K12](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000005845.2/)).
   Genome size (~5 Mbases), GC content (~50%), number of genes (~5k).
2. Tour of contigs and annotation files. Contigs (.fna), gene location on
   contigs (inference.tsv), open reading frames (.ffn), and protein sequences
   (.faa).
3. Taxonomic annotation of contigs (.taxa.tsv). Look at contig assignments from
   day 1 and month 1 samples. Look at species-level summary.

### Align ORFs between samples

The E. coli strain is retained in infant s104.

```bash
vsearch --usearch_global data_assembly/s104.STL.V01.1d.ffn --db data_assembly/s104.STL.V02.1m.ffn --blast6out vsearch_s104.V01_s104.V02_ffn.txt --id 0.8
```

Contrast with alignment to the 1d sample from infant s285.

```bash
vsearch --usearch_global data_assembly/s104.STL.V01.1d.ffn --db data_assembly/s285.STL.V01.1d.ffn --blast6out vsearch_s104.V01_s285.V01_ffn.txt --id 0.8
```

### Align ORFs to Single Copy Core Genes (SCCGs).

```bash
vsearch --usearch_global data_assembly/s104.STL.V01.1d.ffn --db refdb/ecoli_k12_sccg.ffn --blast6out vsearch_s104.V01_ecoli_sccg.txt --id 0.8
```

Count hits to SCCGs

```bash
cat vsearch_s104.V01_ecoli_sccg.txt | cut -f 2 | sort | uniq -c
```

We notice that one gene, NP_417757.1, was hit twice. One of the hits does not
cover the full gene, and corresponds to KELACCIFFN_254 Alpha operon ribosome
binding site. We can see that th other hits cover the full length of the target
genes.

### Align ORFs to StrainPhlAn marker genes.

```bash
vsearch --usearch_global data_assembly/s104.STL.V01.1d.ffn --db refdb/ecoli_strainphlan.ffn --blast6out vsearch_s104.V01_ecoli_strainplhan.txt --id 0.8
```

```bash
cat vsearch_s104.V01_ecoli_strainplhan.txt | cut -f 2 | sort
```

Compare to the marker genes found by alignment of reads.

### BONUS: Point out phage proteins

```bash
less -i igram1_assembly/s104.STL.V01.1d.ffn
```

End with infant virome paper.
