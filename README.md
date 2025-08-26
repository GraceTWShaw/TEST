# DACT: Differential ATAC–Chromatin–Transcriptome Pipeline

DACT is a Nextflow DSL2 pipeline that integrates ATAC-seq, RNA-seq, Hi-C, and promoter annotations to calculate Differential ATAC–Chromatin–Transcriptome (DACT) scores.
It supports modular workflows, containerized execution (Docker/Singularity), and flexible re-analysis with customizable sample pairings.

### Features

- End-to-end integration of ATAC-seq, RNA-seq, Hi-C, and promoter data
- Modular workflow in Nextflow DSL2
- Portable with Docker or Singularity containers
- Automatic generation of SampleInfo.tsv and SamplePair.tsv
- Re-analysis workflow recalDACT to test new combinations without rerunning the full pipeline


### Requirements

- Nextflow ≥ 23.10
- Java ≥ 11
- Docker or Singularity/Apptainer (recommended for HPC)


### Installation
Clone the repository:
<pre>
git clone https://github.com/your-org/DACT.git
cd DACT
</pre>

### Directory Layout
<pre>
DACT/
├─ dact.nf            # Main Nextflow pipeline
├─ dact.config        # Configuration file
├─ images/
│   └─ dact.sif       # Singularity container
├─ scripts/           # Pipeline scripts
│   ├─ *.py           # Python scripts
│   └─ *.R            # R scripts
├─ resources/
│   ├─ SampleInfo.tsv   # Auto-generated
│   ├─ SamplePair.tsv   # Auto-generated (editable for recalDACT)
│   └─ Promoter.tsv     # External promoter annotations
│   └─ hek293/
│       ├─ atac_seq/
│       │   ├─ rep1/*.narrowPeak.gz
│       │   ├─ rep2/*.narrowPeak.gz
│       │   ├─ rep3/*.narrowPeak.gz
│       │   └─ bam/*.bam
│       ├─ hic/*      # Hi-C data
│       └─ rna_seq/
│           ├─ rep1/rsem_quant/*.genes.results
│           ├─ rep2/rsem_quant/*.genes.results
│           └─ rep3/rsem_quant/*.genes.results
└─ output/
</pre>

### Inputs includes
Update the input paths and parameters in <b> dact.config </b>.
#### ATAC-seq:
  - Peak files:
    <pre> ATACpeakFile='/atac_seq/*.narrowPeak.gz' </pre>
  - Label for DESeq2 outputs:
    <pre> ATACSeq='ATACseq' </pre>
  - Quality score filter (remove peaks with Q < 5):
    <pre> atac_minQ=5 </pre>        
#### RNA-seq:
  - Gene expression files:
    <pre> RNAFilePattern='/rsem_quant/*genes.results' </pre>
  - Label for DESeq2 outputs:
    <pre> RNASeq='RNAseq'   </pre>
  - Gene annotation:
    <pre> RNA_ANN_File="${resources_dir}/hg38_annotation.txt" </pre>
  - Quantification type:
    <pre> RNA_quantification='expected_count' </pre>

### Workflow Overview
1. ATAC-seq peaks linked with promoters
2. ATAC-promoter links intersected with Hi-C loops
3. RNA-seq fold changes mapped to gene IDs in loops
4. Normalization restricted to Hi-C regions
5. DACT calculation

### Running the Pipeline

Run the full workflow:

<pre> bash nextflow run dact.nf -c dact.config </pre>

### Outputs include:

- ATAC_counts.txt, ATAC_ann.txt, ATAC_conds.txt
- RNA_counts.txt, RNA_ann.txt, RNA_conds.txt
- Comparison results: HEK293_vs_IMR32_ATACseq.txt, HEK293_vs_IMR32_RNAseq.txt, etc.

### Re-running with recalDACT

After the full run, edit resources/SamplePair.tsv to define new Target–Reference comparisons.

Example:
| Target | Reference                | Check |
|--------|--------------------------|-------|
| HEK293 | SHSY5Y, SKNSH, IMR32     | PASS  |
| IMR32  | HEK293                   | PASS  |

Then run:

<pre> nextflow run dact.nf -c dact.config --entry recalDACT </pre>





