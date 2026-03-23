# rm_dup
Removal of haplotypic and redundant contigs using alignment and gene-based information

## Dependency
Python 3.x.x

Python packages:
- pandas

External tools:
- bedtools
- samtools
- seqkit



## Overview
RMdup identifies and removes duplicated contigs from draft genome assemblies by integrating self-alignment and BUSCO gene annotations.

## Installation
git clone ???
chmod 755 *

## Run
### Step 0. make input file
Before running rmdup, three input files are required:
genome index file (.fai), Minimap2 self-alignment file (.paf), and BUSCO result file (full_table.tsv).
The following commands can be used to generate these three files.
#### genome index file
```
samtools faidx $assembly
```

#### minimap2

```
minimap2 -xasm5 -DP -t 48 $assembly $assembly | gzip -c - > $assembly.paf.gz
```
#### busco

```
busco -i $assembly -l $odb -o $assembly_busco -m genome -c 36
```

### Step 1. Preprocessing
In Step 0, the three prepared input files are used to run rmdup_prepare.
```
./rmdup_prepare -fai {index file} -b {full_table.tsv} -m {selfalign.paf.gz} -p {output prefix}
Option:
 -fai, --fai FILE       genome index file.
 -b, --busco FILE       full_table.tsv from busco result.
 -m, --mapping FILE     gzip paf file from minimap2 result.
 -p, --prefix STR       output prefix [default: rm_dup].
 -v, --version          show version.
 -h, --help             This message.
```

#### run example command
```
./rmdup_prepare -fai test.fasta.fai -b full_table.tsv -m test.fasta.paf.gz
```
#### output format
##### rm_dup.sorted.paf

##### rm_dup_sorted_full_busco.tsv
##### sorted_filtered_judaica.fasta.fai

### Step 2. Identify duplicated site

```
./rmdup_main -sm {sorted.paf} -sb {sorted_full_busco.tsv} -sfai {sorted.fai} -p {prefix}

Option:
 -sm, --sorted-paf FILE          simplified paf from rmdup_prepare (*.sorted.paf)
 -sb, --sorted-busco FILE        sorted busco table from rmdup_prepare (*_sorted_full_busco.tsv)
 -sfai, --sorted-fai FILE        sorted fai (sorted_*.fai)
 -t1, --type1 INT        type1 coverage cutoff [default: 70]
 -t2, --type2 INT        type2 coverage cutoff [default: 50]
 -t0, --type0 INT        type0 coverage cutoff [default: 50]
 -r, --repeat INT        repeat/noBUSCO overlap cutoff [default: 50]
 -p, --prefix STR        output prefix [default: rm_dup]
 -v, --version           show version
 -h, --help              This message


```


### Step 3. Identify keep site

```
rmdup_makebed -i temp.bed [-p prefix]

Option:
 -i, --input FILE      input temp bed file (ex. prefix.temp.bed)
 -p, --prefix STR      output prefix (output = {prefix}_{input_basename})
                      [default output: rm_sup_{input_basename}]
 -v, --version         show version
 -h, --help            This message

```


### Step 4 Extract final contig

```
rmdup_getseq -tB {temp_keepbed} -g {genome} -fai {genome_index} -l {filterLen} -p {prefix}

Option:
 -tB, --tempBed         .temp.bed from step3_keep bed result.
 -g, --genome            raw genome assembly fasta.
 -fai, --fai FILE       genome index file.
 -l, --filterLen FILE   filter length [default: 50000].
 -p, --prefix STR       output prefix [default: rm_dup].
 -v, --version          show version.
 -h, --help             This message.
```








## Input
- PAF (minimap2)
- BUSCO full_table.tsv
- genome.fai

## Output
- BED file of duplicated regions
- filtered assembly

## Example
cd example
bash run.sh

## Citation
(논문 넣기)
