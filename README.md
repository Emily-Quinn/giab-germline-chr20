# GIAB HG002 chr20 Germline Variant Calling

A germline variant-calling pipeline (BWA-MEM → GATK HaplotypeCaller) run on chromosome 20 of the Genome in a Bottle sample HG002 (NA24385), benchmarked against the GIAB truth set with hap.py. This mirrors how a clinical lab validates an NGS test: run a reference sample with a known answer, measure sensitivity and precision, and document the results.

## Status

- [x] Repo and pinned conda environment
- [x] Truth set and reference downloaded and verified
- [ ] Read dataset chosen; chr20 reads extracted
- [ ] Alignment and BAM processing
- [ ] Variant calling and filtering
- [ ] Benchmarking with hap.py
- [ ] Error review and write-up

## Data

**Sample:** HG002 (NA24385), GIAB Ashkenazi Trio son

**Truth set:** GIAB HG002 v5.0q small-variant benchmark, GRCh38
- `HG002_GRCh38_v5.0q_smvar.vcf.gz` and `HG002_GRCh38_v5.0q_smvar.benchmark.bed`
- Source: https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG002_NA24385_son/v5.0q/
- Chosen because NIST lists v4.2.1 as deprecated for HG002 in favor of v5.0q. A comparison against v4.2.1 is planned.

**Reference:** GRCh38 no-alt analysis set (`GCA_000001405.15_GRCh38_no_alt_analysis_set.fna`)
- Source: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/001/405/GCA_000001405.15_GRCh38/seqs_for_alignment_pipelines.ucsc_ids/
- No alt contigs, so standard BWA-MEM can be used without ALT-aware mode.

**Reads:** TBD (dataset name, platform, read length, coverage)

All downloads were checked against the provider's md5 checksums. Large files (FASTQ, BAM, VCF, reference) are not stored in this repo.

## Tools

Pinned in `environment.yml`; full resolved environment in `environment.lock.yml`.

| Tool | Version |
| --- | --- |
| bwa | 0.7.19 |
| samtools | 1.25 |
| GATK | 4.6.2.0 |
| fastp | 1.4.0 |
| FastQC | 0.13.0 |
| MultiQC | 1.35 |
| mosdepth | 0.3.3 |
| hap.py | TBD (container image) |

## Pipeline

1. Extract chr20 reads from a public HG002 BAM and convert to paired FASTQ
2. QC and trimming (FastQC, fastp, MultiQC)
3. Align with BWA-MEM, with read groups
4. Sort and index (samtools)
5. Mark duplicates (GATK MarkDuplicates)
6. Base quality score recalibration (GATK BaseRecalibrator, ApplyBQSR)
7. Coverage check (mosdepth)
8. Call variants (GATK HaplotypeCaller, chr20)
9. Hard-filter SNPs and indels (GATK VariantFiltration)
10. Benchmark with hap.py (vcfeval engine, v5.0q benchmark regions, GIAB v3.6 stratifications)

## How to run

TBD

## Results

TBD. Only results from my own runs will be reported here.

## What broke and how I fixed it

- Building the conda environment took about 2 hours, almost all of it in the "Executing transaction" step (writing files). Likely cause: slow writes to the shared `/home` filesystem (not confirmed).

## References

- Genome in a Bottle Consortium, NIST: https://www.nist.gov/programs-projects/genome-bottle
- v5.0q benchmark: until the paper is published, GIAB asks users to cite the T2T HG002 Q100 project (https://github.com/marbl/hg002, https://doi.org/10.1101/2025.09.21.677443)
- GA4GH small-variant benchmarking best practices (Krusche et al., 2019, Nature Biotechnology)
