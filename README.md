[![Cite with Zenodo](https://zenodo.org/badge/DOI/10.5281/zenodo.8251379.svg)](https://doi.org/10.5281/zenodo.8251379)


[![Nextflow](https://img.shields.io/badge/nextflow%20DSL2-%E2%89%A523.04.0-23aa62.svg)](https://www.nextflow.io/)
[![run with conda](http://img.shields.io/badge/run%20with-conda-3EB049?labelColor=000000&logo=anaconda)](https://docs.conda.io/en/latest/)
[![run with docker](https://img.shields.io/badge/run%20with-docker-0db7ed?labelColor=000000&logo=docker)](https://www.docker.com/)
[![run with singularity](https://img.shields.io/badge/run%20with-singularity-1d355c.svg?labelColor=000000)](https://sylabs.io/docs/)
[![Launch on Nextflow Tower](https://img.shields.io/badge/Launch%20%F0%9F%9A%80-Nextflow%20Tower-%234256e7)](https://tower.nf/launch?pipeline=https://github.com/wal-yan/target-methylseq-qc)

## Introduction

**wal-yan/target-methylseq-qc** is a lightweight pipeline for collecting metrics from targeted sequence mapping files.

The `target-methylseq-qc` pipeline is designed to streamline the quality control process for target methylation sequencing data. Researchers and bioinformaticians working with methylation sequencing data often face challenges in ensuring data quality and consistency across different samples and experiments. This pipeline addresses these challenges by providing a standardized and automated workflow for quality control, leveraging the capabilities of the nf-core framework.

Key features of the target-methylseq-qc pipeline include

(i) **Standardized Input Format**: The pipeline expects a CSV samplesheet with specific fields tailored to different modes (`picard_profiler` and `bed_filter`), ensuring consistency and ease of use

(ii) **Flexible Execution Modes**: Users can choose between different subworkflows (picard_profiler and bed_filter) based on their specific needs, enabling tailored quality control processes

(iii) **Comprehensive Parameter Control**: Users can fine-tune the pipeline's behavior through a wide range of parameters, covering execution modes, input/output options, reference genome options, and infrastructural configuration.

By automating and standardizing the quality control process, the `target-methylseq-qc` pipeline helps researchers save time, reduce errors, and ensure high-quality data for downstream analysis and clinically applicable insights.


## Documentation

The documentation for the pipeline is hosted in https://wal-yan.github.io/target-methylseq-qc/usage.html


## Testing



### Test profiles

Two built-in test profiles are available in target-methylseq-qc pipeline for each mode of execution. These profiles can be used to run tests on the relevant infrastructure using the bundled test datasets ([published on Zenodo](https://doi.org/10.5281/zenodo.13597863)), to help users identify and resolve any infrastructural issue before the analysis stage.


**NOTE**: The snippets below assumes you have `docker` on the sever/machine you wish to test the pipeline. For other institutional configs please refer [nf-core/configs](https://nf-co.re/docs/usage/configuration#max-resources) project, which are all applicable to this pipeline.

```bash

# picard_profiler mode
$ nextflow run wal-yan/target-methylseq-qc \
  -profile docker,test_picard_profiler


# bed_filter mode
$ nextflow run wal-yan/target-methylseq-qc \
  -profile docker,test_bed_filter
```

## Contributions and Support

If you would like to contribute to this pipeline, please see the [contributing guidelines](.github/CONTRIBUTING.md).

## Citations

If you use the pipeline in your work, please cite the pipeline as shown below (in BIBTEX)

```tex
@software{Sharma_wal-yan_target-methylseq-qc_2024,
author = {Sharma, Abhinav and Conradie, Talya and Martino, David and Stick, Stephen and Agudelo-Romero, Patricia},
month = aug,
title = {{wal-yan/target-methylseq-qc}},
url = {https://github.com/wal-yan/target-methylseq-qc},
version = {2.0.0},
year = {2024}
}
```

In addition, an extensive list of references for the tools used by the pipeline can be found in the [`CITATIONS.md`](CITATIONS.md) file.

This pipeline uses code and infrastructure developed and maintained by the [nf-core](https://nf-co.re) community, reused here under the [MIT license](https://github.com/nf-core/tools/blob/master/LICENSE).

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).
