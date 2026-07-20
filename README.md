# gtfgff-scripts

A set of command-line Python scripts for common GTF/GFF processing tasks (format conversion, subsetting, BED/interval export, junctions, sequence extraction, QC, and expression utilities).

Most scripts were originally written by Georgi Marinov; a few were added or updated by Peng He. They live in `scripts/` and are designed to be run from the shell.

```bash
python scripts/GTFsubset.py genes.gtf chr1 1000 50000 region.gtf
python scripts/gtf-to-exons.py genes.gtf exons.bed
```

## Format conversion & cleanup

### `GFFToGTF.py`

Converts GFF-style annotation into GTF, keeping exon and CDS/UTR features.

Supports JGI, GFF3, and GiardiaDB attribute layouts.

```bash
python scripts/GFFToGTF.py genes.gff GFF3 genes.gtf
python scripts/GFFToGTF.py genes.gff JGI genes.gtf
```

### `fix_ENSEMBL_GTF.py`

Rewrites chromosome names in a GTF using an old→new name map (`old <tab> new`).

Optional `-genePredictionToGeneName` converts `GenePrediction` attributes into standard `gene_id` / `transcript_id` (and related) fields.

```bash
python scripts/fix_ENSEMBL_GTF.py input.gtf chr_map.tsv fixed.gtf
python scripts/fix_ENSEMBL_GTF.py input.gtf chr_map.tsv fixed.gtf -genePredictionToGeneName
```

### `ORFGTF.py`

Keeps only CDS features and relabels them as `exon` (useful for ORF-only models).

```bash
python scripts/ORFGTF.py input.gtf orfs_as_exons.gtf
```

### `GTFtoAntiSenseGTF.py`

Writes an antisense version of a GTF by flipping strand and appending a suffix to gene/transcript IDs and names.

```bash
python scripts/GTFtoAntiSenseGTF.py genes.gtf AS antisense.gtf
```

### `GTFtoTBL-NonCodingProk.py`

Converts a prokaryotic non-coding GTF into NCBI `.tbl` format for submission.

Optional `-tRNAScanSE` renames tRNAs for `tbl2asn` compliance. Intended for prokaryote genomes only.

```bash
python scripts/GTFtoTBL-NonCodingProk.py genes.gtf LOC 1 features.tbl
python scripts/GTFtoTBL-NonCodingProk.py genes.gtf LOC 1 features.tbl -tRNAScanSE
```

## Extraction / subsetting

### `GTFsubset.py`

Extracts GTF features that lie entirely within a genomic interval (`chr`, `left`, `right`).

```bash
python scripts/GTFsubset.py genes.gtf chr1 1000 50000 region.gtf
```

### `GTFTranscriptSubset.py`

Extracts GTF lines whose gene or transcript IDs appear in a wanted list.

The wanted file is tab-delimited; `IDfield` selects which column holds the ID to match.

```bash
python scripts/GTFTranscriptSubset.py genes.gtf wanted.txt 0 genes subset.gtf
python scripts/GTFTranscriptSubset.py genes.gtf wanted.txt 0 transcripts subset.gtf
```

### `getGTFGenes.py`

Subsets a GTF to genes listed in an input table, matching either `gene_name` or `gene_id`.

```bash
python scripts/getGTFGenes.py gene_list.txt 0 name genes.gtf subset.gtf
python scripts/getGTFGenes.py gene_list.txt 0 geneID genes.gtf subset.gtf
```

### `getGTFTranscripts.py`

Subsets a GTF to transcripts whose IDs appear in an input table.

```bash
python scripts/getGTFTranscripts.py tx_list.txt 0 genes.gtf subset.gtf
```

### `CDS-UTR-from-gtf.py`

From a GENCODE-style GTF with CDS/UTR features, writes separate CDS, 5′ UTR, and 3′ UTR outputs using an outfile prefix.

Only UTRs in the first/last exons are emitted.

```bash
python scripts/CDS-UTR-from-gtf.py gencode.gtf regions
# writes regions-CDS, regions-5UTR, regions-3UTR
```

### `CDS-UTR-from-gtf-V2.py`

Derives CDS/UTR intervals from protein-coding transcripts that have CDS (and exon) annotations; explicit UTR lines are not required.

Only lines containing `protein_coding` are considered.

```bash
python scripts/CDS-UTR-from-gtf-V2.py genes.gtf cds_utr.out
```

### `3ExonfromGTF.py`

Extracts terminal (3′) exons from multi-exon genes. Monoexonic genes are skipped.

```bash
python scripts/3ExonfromGTF.py genes.gtf single terminal_exons.gtf
```

### `3endGTF.py`

Builds fixed-length 3′-end features for each transcript (length in bases from the transcript end).

```bash
python scripts/3endGTF.py genes.gtf 200 three_prime.gtf
```

## Conversion to BED / intervals

### `gtftobed.py`

Converts GTF exons to BED-like transcript intervals. Optional `-only` keeps first, middle, or last exons.

```bash
python scripts/gtftobed.py genes.gtf exons.bed
python scripts/gtftobed.py genes.gtf first_exons.bed -only first
```

### `GTF-to-ExonBed.py`

Collapses identical exon intervals and reports associated gene/transcript IDs and names.

```bash
python scripts/GTF-to-ExonBed.py genes.gtf exons.bed
```

### `gtf-to-exons.py`

Writes one line per exon (`chr`, `left`, `right`, `strand`, attributes).

```bash
python scripts/gtf-to-exons.py genes.gtf exons.tsv
```

### `gtf-to-genes.py`

Collapses each gene to a single interval spanning all of its features.

```bash
python scripts/gtf-to-genes.py genes.gtf genes.bed
```

### `gtf-to-transcripts-bed.py`

Collapses each transcript to a single BED-like interval spanning all exons.

```bash
python scripts/gtf-to-transcripts-bed.py genes.gtf transcripts.bed
```

### `TSS_bed_FromGTF.py`

Writes TSS-centered BED windows of a given radius around each transcript start.

Optional `-bioType` filters by comma-separated `transcript_type` values.

```bash
python scripts/TSS_bed_FromGTF.py genes.gtf 500 tss.bed
python scripts/TSS_bed_FromGTF.py genes.gtf 500 tss.bed -bioType protein_coding,lincRNA
```

### `GTFtoTiles.py`

Splits annotation into per–gene-type / strand tile outputs using an output prefix.

```bash
python scripts/GTFtoTiles.py genes.gtf tiles
```

## Junctions / splice sites

### `gtf-to-juncs.py`

Lists splice junctions implied by exon chains, with gene/transcript annotation.

```bash
python scripts/gtf-to-juncs.py genes.gtf junctions.tsv
```

### `junctionsfromGTF.py`

Writes unique junction coordinates (`chr`, `left`, `right`, `strand`).

```bash
python scripts/junctionsfromGTF.py genes.gtf junctions.bed
```

### `splicesfromgtf.py`

Writes unique splice sites in `chr:donor-acceptorstrand` format.

```bash
python scripts/splicesfromgtf.py genes.gtf splices.txt
```

### `JunctionsFastaFromGTF.py`

Extracts circularized junction sequences from a reference FASTA (donor joined to upstream acceptors), with a fixed span of sequence on each side.

```bash
python scripts/JunctionsFastaFromGTF.py genome.fa genes.gtf 60 circ_juncs.fa
```

## Sequence extraction

### `gtf-to-fasta.py`

Extracts spliced transcript sequences using a cistematic `Genome` object. Optional poly(A) tail.

```bash
python scripts/gtf-to-fasta.py hg19 genes.gtf transcripts.fa
python scripts/gtf-to-fasta.py hg19 genes.gtf transcripts.fa -polyA 50
```

### `gtf-to-fastaFromFasta.py`

Extracts spliced transcript sequences from a FASTA genome file (no cistematic dependency).

Supports poly(A) tails, chromosome-field parsing, `chr` prefix adjustment, and undetermined-strand handling.

```bash
python scripts/gtf-to-fastaFromFasta.py genome.fa genes.gtf transcripts.fa
python scripts/gtf-to-fastaFromFasta.py genome.fa genes.gtf transcripts.fa -polyA 50 -KeepUundeterminedStrand
```

### `GTFtoProteinFasta.py`

Translates transcripts in all three reading frames using a cistematic genome. Stop codons become `.`.

Optional `-spliced` and `-class_code` filters.

```bash
python scripts/GTFtoProteinFasta.py hg19 genes.gtf proteins.fa
python scripts/GTFtoProteinFasta.py hg19 genes.gtf proteins.fa -spliced -class_code =
```

### `GTFtoProteinFasta2.py`

Same three-frame translation as `GTFtoProteinFasta.py`, but reads sequence from a FASTA file instead of cistematic.

```bash
python scripts/GTFtoProteinFasta2.py genome.fa genes.gtf proteins.fa
python scripts/GTFtoProteinFasta2.py genome.fa genes.gtf proteins.fa -spliced
```

## QC / statistics

### `GTFstats.py`

Summarizes gene/transcript structure statistics for a GTF (with optional JGI / GFF3 parsing modes).

```bash
python scripts/GTFstats.py genes.gtf stats
python scripts/GTFstats.py genes.gff stats -GFF3
python scripts/GTFstats.py genes.gff stats -JGI
```

### `GTFcomparison.py`

Compares exon-chain structure between two GTF files.

```bash
python scripts/GTFcomparison.py annot1.gtf annot2.gtf comparison.tsv
```

### `GTFExonicBasePairs.py`

Computes the total number of unique exonic base pairs (optional flank extension and gene-type filter).

```bash
python scripts/GTFExonicBasePairs.py genes.gtf exonic_bp.txt
python scripts/GTFExonicBasePairs.py genes.gtf exonic_bp.txt -extend 50 -genetype protein_coding
```

### `geneLengthsFromGTF.py`

Reports transcript (or longest-isoform) lengths per gene.

```bash
python scripts/geneLengthsFromGTF.py genes.gtf lengths.tsv
python scripts/geneLengthsFromGTF.py genes.gtf lengths.tsv -longestOnly
```

### `GTFGenesTranscriptsBioTypes.py`

Lists genes and transcripts with their biotypes (`gene_type` / `transcript_type`, or a fixed field via `-field`).

```bash
python scripts/GTFGenesTranscriptsBioTypes.py genes.gtf biotypes.tsv
python scripts/GTFGenesTranscriptsBioTypes.py genes.gtf biotypes.tsv -field 1
```

### `gtfGenesTranscripts.py`

Writes a table of unique gene/transcript ID and name combinations.

```bash
python scripts/gtfGenesTranscripts.py genes.gtf gene_tx.tsv
```

### `gtf-to-proteinIDs.py`

Maps gene/transcript IDs and names to `protein_id` attributes when present.

```bash
python scripts/gtf-to-proteinIDs.py genes.gtf protein_ids.tsv
```

### `GTFRepeatMaskOverlap.py`

Reports overlap between GTF exons and RepeatMasker entries (tab-delimited RM format assumed).

```bash
python scripts/GTFRepeatMaskOverlap.py genes.gtf repeats.rmsk overlap.tsv
```

### `gene_coverage_wig_gtf.py`

Computes gene-body coverage percentiles from a bedGraph or bigWig against a GTF annotation.

Requires `numpy`; optional `pyBigWig` / `pysam` for bigWig/BAM input.

```bash
python scripts/gene_coverage_wig_gtf.py coverage.bw --gtf genes.gtf -o coverage_profile.tsv
python scripts/gene_coverage_wig_gtf.py coverage.bedGraph --gtf genes.gtf -o coverage_profile.tsv --gene-type protein_coding
```

## Expression / quantification utilities

### `ENCODE-RPKM-from-GTF.py`

Extracts RPKM values embedded in GTF attributes and joins them to gene names from an annotation GTF.

The data GTF argument may be `-` for stdin.

```bash
python scripts/ENCODE-RPKM-from-GTF.py annotation.gtf rpkm.gtf rpkm.tsv
python scripts/ENCODE-RPKM-from-GTF.py annotation.gtf - rpkm.tsv < rpkm.gtf
```

### `FluxGTFtoFPKM.py`

Parses Flux Simulator–style transcript RPKM attributes from a GTF into a table.

```bash
python scripts/FluxGTFtoFPKM.py flux.gtf fpkm.tsv
```

### `SumIsoformValuesFromGTF.py`

Aggregates isoform-level quantification values up to the gene level using a GTF for transcript→gene mapping.

```bash
python scripts/SumIsoformValuesFromGTF.py isoform_counts.tsv genes.gtf gene_sums.tsv
```
