---
title: 'target-methylseq-qc: a lightweight pipeline for collecting Picard metrics from targeted sequence mapping files.'
tags:
  - nextflow
  - pipeline
  - bioinformatics
  - picard
  - bedtools
  - metrics
  - reference-fasta
  - target-methylseq
  - qc
  - quality-control
authors:
  - name: "Abhinav Sharma"
    orcid: 0000-0002-6402-6993
    affiliation: 1
  - name: "Talya Conradie"
    orcid: 0009-0007-6380-5737
    affiliation: "2, 3"
  - name: "David Martino"
    orcid: 0000-0001-6823-4696
    affiliation: "2"
  - name: "Stephen Stick"
    orcid: 0000-0002-5386-8482
    affiliation: "2, 4, 5"
  - name: "Patricia Agudelo-Romero"
    orcid: 0000-0002-3703-4111
    affiliation: "2, 6, 7"

affiliations:
  - name: Division of Molecular Biology and Human Genetics, Faculty of Medicine and Health Sciences, Stellenbosch University, Cape Town.
    index: 1
  - name: Wal-yan Respiratory Research Centre, Telethon Kids Institute, WA, Australia
    index: 2
  - name: Medical, Molecular and Forensic Sciences, Murdoch University, WA, Australia
    index: 3
  - name: Department of Respiratory and Sleep Medicine, Perth Children’s Hospital for Children, WA, Australia.
    index: 4
  - name: Centre for Cell Therapy and Regenerative Medicine, School of Medicine and Pharmacology, WA, Australia.
    index: 5
  - name: Australian Research Council Centre of Excellence in Plant Energy Biology, School of Molecular Sciences, The University of Western Australia, WA, Australia
    index: 6
  - name: European Virus Bioinformatics Center, TH, Germany.
    index: 7

date: 18 August 2024
bibliography: paper.bib

# Optional fields if submitting to a AAS journal too, see this blog post:
# https://blog.joss.theoj.org/2018/12/a-new-collaboration-with-aas-publishing
#aas-doi: 10.3847/xxxxx <- update this with the DOI from AAS once you know it.
#aas-journal: Astrophysical Journal <- The name of the AAS journal.
---

# Summary


Next-generation targeted genome sequencing offers the opportunity to analyse regions of interest within a genome.
While it is possible to incorporate targeted sequencing into whole-genome sequencing (WGS) pipelines, there remains a gap in accurately converting WGS metrics into precise target metrics. Here, we introduce the target-methylseq-qc pipeline (https://doi.org/10.5281/zenodo.8251379 ), designed to (i) collects metrics from alignment files generated in targeted-methylation sequence analysis and (ii) filtering `bedGraph` for features overlapping with the reference BED file, both of these subworkflows are written using Nextflow [@di_tommaso_nextflow_2017] workflow language.

target-methylseq-qc, when used in the `picard-profiler` mode accepts inputs in various alignment formats, including SAM, BAM and CRAM files [@hts_spec].
Additionally, to refine the metrics to the target regions the inclusion of a FASTA reference file and BED intervals file is required.
Subsequently, a MultiQC report [@ewels_multiqc_2016] will be generated, encompassing the updated sequencing coverage data for the targeted regions with some extras.

The `picard_profiler` mode of the pipeline integrates Picard metrics from GATK picard tools [@mckenna_genome_2010; @Picard2019toolkit], using two specific metrics: (i) collecthsmetrics [@picard_collecthsmetrics_2019], which relies upon the hybrid-selection technique to capture exon sequences for targeted sequencing experiments; and (ii) collectmultiplemetrics [@picard_collectmultiplemetrics_2021], which captures closely related metrics such as alignment summary, insert size, and quality score. On the other hand, `bed_filter` mode of the pipeline is designed to accommodate the use-case of filtering bedGraph files as per the reference bed panel, such as Twist Human Methylome panel [@twist_methylome] using bedtools [@quinlan_2010].

Regardless of the usage mode of the pipeline, the final MultiQC report automatically collates the relevant reports from FastQC [@andrews_fastqc_2010], Bedtool and Picard tools in an HTML document, which could be shared with collaborators or added as supplementary material in publications.

target-methylseq-qc is a portable pipeline compatible with multiple platforms, such as local laptop or workstation machines, high-performance computing environments and cloud infrastructure. Although target-methylseq-qc was originally created for calculating coverage in target sequencing as a follow-up step to the nf-core/methylseq pipeline, within the Airway Epithelium Respiratory Illnesses and Allergy (AERIAL) paediatric cohort study[@kicic-starcevich_airway_2023]; its versatility allows for extending its application to other sequencing panels from various next-generation methods.


# Design principles and capabilities

target-methylseq-qc pipeline builds upon the standardised pipeline template maintained by the nf-core community [@ewels_nf-core_2020] for Nextflow  pipelines as well as makes use of the nf-core/modules project to install modules for FastQC, MultiQC [@ewels_multiqc_2016] , Bedtool, Picard as well as Samtools [@danecek_twelve_2021] within the pipeline \autoref{fig:subway-map}.


The use of the nf-core template facilitates in keeping the design of the pipeline generic and portable across different execution platforms, therefore the target-methylseq-qc pipeline can be used on local machines, HPC orchestrators (e.g. SLURM, PBS), and cloud computing systems such as AWS Batch, Azure Batch, Google Batch, in addition to the more generic Kubernetes distribution.


![Subway map for various steps in the target-methylseq-qc pipeline.\label{fig:subway-map}](subway_pic.svg)

In addition to the base workflow as mentioned in \autoref{fig:subway-map}, the pipeline also includes optional picard/createsequencedictionary [@picard_createsequencedictionary_2022] and Samtools modules to aid users in automatically generating the required genome dictionary (DICT) file, in case they have only the reference FASTA and BED files but intend to use the pipeline. Furthermore, depending on the quality check requirements by the users, we have enabled the metrics collection for 10x, 20x, 30x and 50x coverage.

# Input and output

As standard input in the Nextflow pipelines, target-methylseq-qc expects a CSV samplesheet as an input with the following fields.

:An example of a samplesheet for target-methylseq-qc in `picard-profiler` mode containing three columns, capturing the (i) name of the sample (ii) path to BAM index file and (iii) path to the BAM file. []{label="samplesheet-1"}

| sample    | bai                    | bam                        |
|-----------|------------------------|----------------------------|
| sample-01 | /path/to/sample-01.bai | /path/to/sample-01.bam     |
| sample-02 | /path/to/sample-02.bai | /path/to/sample-02.bam     |

Whereas the `bed_filter` mode requires a different set of columns in the input samplesheet CSV file, as shown in table []{label="samplesheet-2"}

| sample    | bedGraph                    |
|-----------|-----------------------------|
| sample-01 | /path/to/sample-01.bedGraph |
| sample-02 | /path/to/sample-02.bedGraph |


The pipeline initialization step, as per the best practices of the nf-core template, checks the validity of the file paths specified to be either a POSIX compliant file system or a cloud object storage path for files storaged in AWS S3, Azure Blob Storage or Google Cloud Storage buckets. Upon completion, the pipeline generates a MultiQC file with the relevant results of the analysis \autoref{fig:multiqc}.

![MultiQC report generated for target-methylseq-qc, in `picard-profiler` highlighting the refine metrics from targeted sequencing at 10X, 20X, 30X and 50X coverage.\label{fig:multiqc}](multiqc.tiff)


# Tutorials and documentation

The steps needed to configure the pipeline inputs and configuration for your infrastructure are available in the documentation within the Github repository itself. Getting started with the pipeline setup is straightforward given that (i) `Java` (LTS > 11)  (ii) `Nextflow` (> 24.04) and (iii) a package manager (e.g. `conda`) or a container system (e.g. `docker` or `singularity`) are available in the execution environment. The in-built test profile from the pipeline can then be used to execute the profile on the relevant infrastructure with some test dataset.


```bash

$ nextflow run wal-yan/target-methylseq-qc -profile test,docker --picard_profiler –outdir test_results

$ nextflow run wal-yan/target-methylseq-qc -profile test,docker --bed_filter –outdir test_results
```

# Funding Statement
This work was supported by the National Health and Medical Research Council of Australia (NHMRC115648).

# References
