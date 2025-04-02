---
title: 'target-methylseq-qc: a lightweight pipeline for collecting metrics from targeted sequence mapping files.'
tags:
  - nextflow
  - pipeline
  - bioinformatics
  - picard
  - bedtools
  - metrics
  - reference-fasta
  - target-methylseq
  - methylation-panel
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
    affiliation: "4, 5"
  - name: "Patricia Agudelo-Romero"
    orcid: 0000-0002-3703-4111
    affiliation: "2, 6, 7"

affiliations:
  - name: DSI-NRF Centre of Excellence for Biomedical Tuberculosis Research; SAMRC Centre for Tuberculosis Research; Division of Molecular Biology and Human Genetics, Faculty of Medicine and Health Sciences, Stellenbosch University, Cape Town, South Africa.
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

---

# Summary


Next-generation targeted genome sequencing allows the analysis of regions of interest within a genome. While it is possible to incorporate targeted sequencing into whole-genome sequencing (WGS) bioinformatics pipelines, there remains a gap in accurately converting WGS metrics into precise target sequencing metrics and filtering the raw BED files to the targeted regions. Here, we introduce the `target-methylseq-qc` pipeline (https://doi.org/10.5281/zenodo.13147688), designed to (i) collect metrics from alignment files generated in targeted methylation sequence analysis, using the `picard_profiler` mode, and (ii) filtering `bedGraph` files to features overlapping with the reference BED file, using the `bed_filter` mode. Both of these modes are subworkflows written using the Nextflow [@di_tommaso_nextflow_2017] workflow language.

The `target-methylseq-qc` pipeline, when used in the `picard_profiler` mode, accepts inputs in various alignment formats, including SAM, BAM and CRAM files [@hts_spec]. Additionally, to refine the metrics to the target regions the inclusion of a FASTA reference file and BED intervals file is required. Upon completion of the analysis, a MultiQC report [@ewels_multiqc_2016] will be generated, encompassing the updated sequencing coverage data for the targeted regions with some extras. The `picard_profiler` mode of the pipeline integrates Picard metrics from GATK Picard tools [@mckenna_genome_2010; @Picard2019toolkit], using two specific metrics: (i) collecthsmetrics [@picard_collecthsmetrics_2019], which relies upon the hybrid-selection technique to capture exon sequences for targeted sequencing experiments; and (ii) collectmultiplemetrics [@picard_collectmultiplemetrics_2021], which captures closely related metrics such as alignment summary, insert size, and quality score.

The `bed_filter` mode of the pipeline uses the bedtools [@quinlan_2010] filter command to filter the `bedGraph` files output by the nf-core/methylseq workflow [@methylseq] using a reference bed panel, in this case the Twist Human Methylome panel (https://www.twistbioscience.com/resources/data-files/twist-human-methylome-panel-target-bed-file) and best practices [@twist_methylome, @twist_methylome_technote]. Filtering raw BED files to the targeted regions is crucial because it ensures that the analysis focuses on specific genomic targets accurately and efficiently. This step minimizes the inclusion of off-target sequences and reduces the potential for including sequencing artifacts, which can be introduced during capture-based targeted sequencing processes. Downstream analyses from the filtered BED files will enable the calculation of CpG ratios and the testing for differentially methylated cytosines (DMCs) or regions (DMRs).

Regardless of the usage mode of the pipeline, the final MultiQC report automatically collates the relevant reports from FastQC [@andrews_fastqc_2010], bedtools and Picard tools in an HTML document, which could be shared with collaborators or added as supplementary material in publications.

`target-methylseq-qc` is a portable pipeline compatible with multiple platforms, such as local laptops or workstations, high-performance computing environments and cloud infrastructure. Although `target-methylseq-qc` was originally created for calculating sequencing coverage in target sequencing as a follow-up step to the `nf-core/methylseq` pipeline [@methylseq] within the Airway Epithelium Respiratory Illnesses and Allergy (AERIAL) paediatric cohort study [@kicic-starcevich_airway_2023], its versatility allows for extending its application to other sequencing panels from various next-generation sequencing methods.

# Statement of need

The `target-methylseq-qc` pipeline is designed to streamline the quality control process for target methylation sequencing data. Researchers and bioinformaticians working with methylation sequencing data often face challenges in ensuring data quality and consistency across different samples and experiments. This pipeline addresses these challenges by providing a standardized and automated workflow for quality control, leveraging the capabilities of the nf-core framework. Key features of the `target-methylseq-qc` pipeline include (i) standardized input format: The pipeline expects a CSV samplesheet with specific fields tailored to different modes (`picard_profiler` and `bed_filter`), ensuring consistency and ease of use, (ii) flexible execution modes: Users can choose between different subworkflows (`picard_profiler` and `bed_filter`) based on their specific needs, enabling tailored quality control processes, (iii) comprehensive parameter control: Users can fine-tune the pipeline's behavior through a wide range of parameters, covering execution modes, input/output options, reference genome options, and infrastructural configuration. By automating and standardizing the quality control process, the `target-methylseq-qc` pipeline helps researchers save time, reduce errors, and ensure high-quality data for downstream analysis and clinically applicable insights.

# Design principles and capabilities

The `target-methylseq-qc` pipeline builds upon the standardised pipeline template maintained by the nf-core community [@ewels_nf-core_2020] for Nextflow pipelines and makes use of the `nf-core/modules` project to install modules for FastQC, MultiQC [@ewels_multiqc_2016], bedtools, Picard tools as well as Samtools [@danecek_twelve_2021] within the pipeline (\autoref{subway-map}).

The use of the nf-core template facilitates keeping the design of the pipeline generic and portable across different execution platforms. Hence, the `target-methylseq-qc` pipeline can be used on local machines, HPC orchestrators (e.g. SLURM, PBS), and cloud computing systems such as AWS Batch, Azure Batch, Google Batch, in addition to the more generic Kubernetes distribution.


![Subway map for various steps in the target-methylseq-qc pipeline.\label{subway-map}](target-methylseq-qc_REVISED.svg)

In addition to the base workflow outlined in \autoref{subway-map}, the pipeline also includes optional picard/createsequencedictionary [@picard_createsequencedictionary_2022] and Samtools modules to aid users in automatically generating the required genome dictionary (DICT) file, in cases where only the reference FASTA and BED files are available. Furthermore, depending on the quality check requirements of the users, we have enabled the metrics collection for 10x, 20x, 30x and 50x coverage.


# Pipeline setup

The steps needed to configure the pipeline inputs and configuration for the relevant infrastructure are available in the documentation within the GitHub repository as well as a dedicated documentation website [@targetmethylseqqc_website].

Getting started with the pipeline setup is straightforward given that (i) `Java` (LTS > 11)  (ii) `Nextflow` (> 24.04) and (iii) a package manager (e.g. `conda`) or a container system (e.g. `docker` or `singularity`) are available in the execution environment.

Two built-in test profiles are available in the `target-methylseq-qc` pipeline for each mode of execution. These profiles can be used to run tests on the relevant infrastructure using the bundled test datasets [@test_dataset], helping users to identify and resolve any infrastructural issue before the analysis stage.


```bash


# Nextflow based installation of pipeline
$ nextflow pull https://github.com/wal-yan/target-methylseq-qc



# In-built test for picard_profiler mode
$ nextflow run wal-yan/target-methylseq-qc \
  -profile docker,test_picard_profiler


# In-built test for  bed_filter mode
$ nextflow run wal-yan/target-methylseq-qc \
  -profile docker,test_bed_filter
```

# Input

Following the convention for standard input in Nextflow pipelines, `target-methylseq-qc` expects a CSV samplesheet as an input. \autoref{samplesheet-1} shows an example samplesheet for `target-methylseq-qc` in `picard-profiler` mode. The samplesheet contains three columns, capturing the (i) name of the sample, (ii) the path to BAM file, and (iii) the path to the BAM index (BAI) file.

:Samplesheet structure for `picard_profiler` mode \label{samplesheet-1}.

| sample    | bam                    | bai                        |
|-----------|------------------------|----------------------------|
| sample-01 | /path/to/sample-01.bam | /path/to/sample-01.bai     |
| sample-02 | /path/to/sample-02.bam | /path/to/sample-02.bai     |

The `bed_filter` mode requires a different set of columns in the input samplesheet CSV file, as shown in \autoref{samplesheet-2}.

:Samplesheet structure for `bed_filter` mode \label{samplesheet-2}.

| sample    | bedGraph                    |
|-----------|-----------------------------|
| sample-01 | /path/to/sample-01.bedGraph |
| sample-02 | /path/to/sample-02.bedGraph |

# Execution


The pipeline initialization step, as per the best practices of the nf-core template, checks the validity of the file paths specified to be either a POSIX-compliant file system or a cloud object storage path for files stored in AWS S3, Azure Blob Storage or Google Cloud Storage buckets.

The behaviour of the pipeline can be controlled through the pipeline parameters which are divided into different groups such as (i) Execution Mode, (ii) Input/Output Options, and (iii) Reference Genome Options. In addition, generic parameters are inherited from the nf-core template, including (i) max job request options, (ii) generic options and (iii) institutional config options. A complete list of the parameters specific to the `target-methylseq-qc` pipeline is summarised in \autoref{parameters}.

:Summary of pipeline-specific parameters for target-methylseq-qc pipeline \label{parameters}.

| Parameter Name  | Description                                                                              |
|-----------------|------------------------------------------------------------------------------------------|
| picard_profiler | Enable this boolean option to use the picard_profiler subworkflow                        |
| bed_filter      | Enable this boolean option to use the bed_filter subworkflow                             |
| input           | Path to comma-separated file containing information about the samples in the experiment. |
| outdir          | The output directory where the results will be saved.                                    |
| ref_fasta       | Path to FASTA genome file.                                                               |
| ref_fai         | Path to the FASTA index file.                                                            |
| ref_bed         | Path to the BED file for the reference panel.                                            |



# Output

Upon completion, the two subworkflows generate different outputs which are presented together in the MultiQC file. For the `picard_profile` mode, a MultiQC file is produced, providing the relevant results related to the coverage metrics (\autoref{results-2a}). For the `bed_filter` mode, a BED file is generated with the methylation positions filtered based on the BED intervals file from the targeted methylation profile (\autoref{results-2b}).


![MultiQC report generated for target-methylseq-qc, in `picard-profiler` highlighting the refine metrics from targeted sequencing at 10X, 20X, 30X and 50X coverage.\label{results-2a}](multiqc.tiff)


![Filtered bedGraph file generated using the `bed_filter` mode of target-methylseq-qc.\label{results-2b}](bed_filter.tiff){width=70%}

# Funding Statement
This work was supported by the National Health and Medical Research Council of Australia (NHMRC115648).

# References
