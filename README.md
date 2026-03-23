# rm_dup
Removal of haplotypic and redundant contigs using alignment and gene-based information

## Overview
RMdup identifies and removes duplicated contigs from draft genome assemblies by integrating self-alignment and BUSCO gene annotations.

## Features
- Alignment-based duplication detection
- BUSCO-guided filtering
- Supports diploid and polyploid genomes
- Prevents overpurging

## Installation
conda env create -f environment.yml

## Usage
python rmdup_main.py \
    --paf input.paf \
    --busco full_table.tsv \
    --fai genome.fai \
    --output result/

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
