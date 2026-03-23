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
<img width="1500" height="1051" alt="image" src="https://github.com/user-attachments/assets/9b14847b-6652-456d-8ea9-4e5919b34dd6" />

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

#### busco
```
busco -i $assembly -l $odb -o $assembly_busco -m genome -c 36
```

#### minimap2
```
minimap2 -xasm5 -DP -t 48 $assembly $assembly | gzip -c - > $assembly.paf.gz
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
##### sorted_test.fasta.fai

```
#Contig length OFFSET LINEBASES LINEWIDTH 
ptg000002l      70232259        54520495        60      61
ptg000011l      64666834        409237564       60      61
ptg000006l      59361535        241215658       60      61
ptg000004l      53872932        145655011       60      61
ptg000001l      53626692        12      60      61
ptg000017l      41640878        595927592       60      61
ptg000005l      40121122        200425838       60      61

```
##### rm_dup_sorted_full_busco.tsv

```
#Contig Busco_gene_name Start End Busco_type
ptg000002l      56324at3193     1621864 1628371 Duplicated
ptg000002l      29166at3193     2632932 2635358 Duplicated
ptg000002l      195796at3193    3009596 3010207 Complete
ptg000002l      1578at3193      5430076 5496512 Complete
ptg000002l      189624at3193    7195531 7204424 Complete
ptg000002l      176959at3193    9197011 9218946 Complete
ptg000002l      141488at3193    10011232        10024614        Complete
ptg000002l      105405at3193    11065195        11080395        Complete
ptg000002l      159493at3193    11344311        11347408        Complete
ptg000002l      169448at3193    16546874        16571658        Complete

```

##### rm_dup.sorted.paf

Query coverage =round(matchbp/query length*100,2)

Subject coverage =round(matchbp/subject length*100,2)

```
#qContig qStart qEnd sContig sStart sEnd qCoverage sCoverage pseudo-duplciateContig 
ptg000002l      64423652        68723853        ptg000018l      15      4295174 5.66    68.4    ptg000018l
ptg000002l      27573476        27980392        ptg000010l      13634105        13999609        0.08    0.15    ptg000010l
ptg000002l      27404417        27911445        ptg000013l      18745016        19339462        0.1     0.29    ptg000013l
ptg000002l      63987419        64054842        ptg000011l      25125753        25213541        0.05    0.05    equal
ptg000002l      27566731        27879995        ptg000014l      17573965        17943355        0.07    0.19    ptg000014l
ptg000002l      27573476        27979937        ptg000015l      7060425 7455515 0.08    0.18    ptg000015l
ptg000002l      53859635        53882931        ptg000009l      8895293 8927631 0.03    0.21    ptg000009l
ptg000002l      27573476        27981886        ptg000008l      26094399        26485938        0.07    0.17    ptg000008l
ptg000002l      27573476        27865837        ptg000004l      32073724        32486284        0.08    0.1     ptg000004l
ptg000002l      27688531        27974166        ptg000006l      26732782        26979669        0.05    0.06    ptg000006l
ptg000002l      67882182        67968354        ptg000019l      160013  234368  0.03    2.29    ptg000019l
ptg000002l      27555701        27775688        ptg000005l      26844139        27123973        0.04    0.07    ptg000005l
ptg000002l      67891992        67968396        ptg000030l      1491830 1564574 0.02    0.48    ptg000030l
ptg000002l      25475840        25498058        ptg000024l      15298   54905   0.02    7.47    ptg000024l
ptg000002l      25471015        25498058        ptg000025l      303252  358645  0.02    3.92    ptg000025l
ptg000002l      25471015        25485128        ptg000023l      126859  140199  0.01    4.6     ptg000023l
ptg000002l      27859520        27891898        ptg000021l      5442    27500   0.02    22.52   ptg000021l

```


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

### run command
```
./rmdup_mainV4 -sm ../1.rmdup_prepare/*.sorted.paf -sb ../1.rmdup_prepare/*_sorted_full_busco.tsv -sfai ../1.rmdup_prepare/sorted*.fai -p sample2 -t2 10 -r 30
```

#### output 
rm_dup.temp.bed
busco based: haplotig, overlap, unique, issubset, repeat
minimap2 based: minimap2

type1_haplotig
type1_issubset
type1_minimap2
type1_repeat
type1_unique
type2_issubset
type2_minimap2
type2_overlap
type2_repeat
type2_unique


```
ptg000294l      temp    0       36320   type1   type1_repeat
ptg000086l      temp    0       34388   type1   type1_repeat
ptg000272l      temp    0       41406   type1   type1_repeat
ptg000008l      temp    86      29012907        type1   type1_minimap2
ptg000008l      54811at3193     61131   75070   type1   type1_haplotig
ptg000008l      216839at3193    107555  162673  type1   type1_unique

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

#### run command
```
./rmdup_bedV4 -i ../2.rmdup_main/rm_dup.temp.bed
```

#### output
rm_dup_rm_dup.temp.bed
type1_haplotig
type1_issubset
type1_minimap2
type1_repeat
type2_issubset
type2_minimap2
type2_overlap
type2_repeat

```
ptg000001l      0       53626692        type2_issubset
ptg000003l      0       19408224        type1_issubset
ptg000003l      0       19408224        type1_issubset
ptg000007l      0       29832611        type2_issubset
ptg000008l      86      29012907        type1_minimap2
ptg000008l      61131   75070   type1_haplotig
ptg000008l      204945  27314041        type1_haplotig
ptg000009l      0       9416874 type1_issubset

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

#### run command
```
./rmdup_getseq -tB ../3.rmdup_keepbed/rm_dup_rm_dup.temp.bed -g ../1.rmdup_prepare/filtered.fasta -fai ../1.rmdup_prepare/filtered.fasta.fai
```

#### output

-rw-rw-r-- 1 ahchoi123 ahchoi123 432M Mar 23 20:50 rm_dup_rm_dup_50000.fa
-rw-rw-r-- 1 ahchoi123 ahchoi123 427M Mar 23 20:50 rm_dup_rm_dup.fa
-rw-rw-r-- 1 ahchoi123 ahchoi123 2.8K Mar 23 20:49 rm_dup_rename.bed

```
ptg000002l      0       70232259        ptg000002l_1
ptg000004l      0       53872932        ptg000004l_1
ptg000005l      0       40121122        ptg000005l_1
ptg000006l      0       59361535        ptg000006l_1
ptg000008l      0       86      ptg000008l_1
ptg000008l      29012907        29014243        ptg000008l_2
ptg000010l      0       37642125        ptg000010l_1
ptg000011l      0       64666834        ptg000011l_1
ptg000013l      0       9       ptg000013l_1

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
