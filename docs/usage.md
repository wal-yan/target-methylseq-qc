# Usage

`target-methylseq-qc`: a lightweight pipeline for collecting metrics from targeted sequence mapping files.

## Introduction

We developed the `target-methylseq-qc` pipeline (https://doi.org/10.5281/zenodo.13147688), designed to (i) collect metrics from alignment files generated in targeted-methylation sequence analysis using the `picard_profiler` mode and (ii) filtering `bedGraph` for features overlapping with the reference BED file using the `bed_filter` mode, both of these modes are subworkflows written using the Nextflow workflow language.


![Subway map for various steps in the target-methylseq-qc pipeline.](target-methylseq-qc_REVISED.svg)


## Pipeline setup

The steps needed to configure the pipeline inputs and configuration for the relevant infrastructure are available in the documentation within the GitHub repository as well as a dedicated documentation website [@targetmethylseqqc_website].

Getting started with the pipeline setup is straightforward given that (i) `Java` (LTS > 11)  (ii) `Nextflow` (> 24.04) and (iii) a package manager (e.g. `conda`) or a container system (e.g. `docker` or `singularity`) are available in the execution environment.

Two built-in test profiles are available in target-methylseq-qc pipeline for each mode of execution. These profiles can be used to run tests on the relevant infrastructure using the bundled test datasets [@test_dataset], helping users to identify and resolve any infrastructural issue before the analysis stage.


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





## Samplesheet input

You will need to create a samplesheet with information about the samples you would like to analyse before running the pipeline. Use this parameter to specify its location. It has to be a comma-separated file with 3 columns, and a header row as shown in the examples below.

```bash
--samplesheet '[path to samplesheet file]'
```

### Full samplesheet


# Input


## `picard_profiler` mode

Following the convention for standard input in the Nextflow pipelines, target-methylseq-qc expects a CSV samplesheet as an input with the following fields. An example of a samplesheet \autoref{samplesheet-1} for target-methylseq-qc in `picard-profiler` mode containing three columns, capturing the (i) name of the sample (ii) path to BAM file and (iii) path to the BAM index (BAI) file.


The pipeline requires the use of the compressed alignment files bam and bai, output from the nfcore/methylseq pipeline. Samplesheet structure for `picard_profiler` mode

| sample    | bam                    | bai                        |
|-----------|------------------------|----------------------------|
| sample-01 | /path/to/sample-01.bam | /path/to/sample-01.bai     |
| sample-02 | /path/to/sample-02.bam | /path/to/sample-02.bai     |



| Column    | Description                                                                                                                                                                            |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sample`  | Custom sample name. This entry will be identical for multiple sequencing libraries/runs from the same sample. Spaces in sample names are automatically converted to underscores (`_`) `## is converting spaces to _ true in picard case? ##`. |
| `bai` | Full path to bai file for sample. File must be those that are "mark duplicate alignments" in sorted files from previous methylseq runs.                                                              |
| `bam` | Full path to bam file for sample. File must be those that are "mark duplicate alignments" in sorted files from previous methylseq runs.                                                             |

## `bed_filter` mode

Whereas the `bed_filter` mode requires a different set of columns in the input samplesheet CSV file. Samplesheet structure for `bed_filter` mode

| sample    | bedGraph                    |
|-----------|-----------------------------|
| sample-01 | /path/to/sample-01.bedGraph |
| sample-02 | /path/to/sample-02.bedGraph |



## Running the pipeline


### Test profiles

Two built-in test profiles are available in target-methylseq-qc pipeline for each mode of execution. These profiles can be used to run tests on the relevant infrastructure using the bundled test datasets ([published on Zenodo](https://doi.org/10.5281/10.5281/zenodo.13597863)), to help users identify and resolve any infrastructural issue before the analysis stage.


**NOTE**: The snippets below assumes you have `docker` on the sever/machine you wish to test the pipeline. For other institutional configs please refer [nf-core/configs](https://nf-co.re/docs/usage/configuration#max-resources) project, which are all applicable to this pipeline.

```bash

# picard_profiler mode
$ nextflow run wal-yan/target-methylseq-qc \
  -profile docker,test_picard_profiler


# bed_filter mode
$ nextflow run wal-yan/target-methylseq-qc \
  -profile docker,test_bed_filter
```


### Using your own dataset

The command for running the pipeline is as follows:

```bash
nextflow run wal-yan/target-methylseq-qc \
         --samplesheet /path/to/samplesheet.csv \
         --bed /data/Twist_met/Twist_Methylome/bundle_reference_files/covered_targets_Twist_Methylome_hg38_annotated_collapsed.bed \
         -profile docker
```

This will launch the pipeline with the `docker` configuration profile. See below for more information about profiles.

Note that the pipeline will create the following files in your working directory:

```bash
work                # Directory containing the nextflow working files
<OUTDIR>            # Finished results in specified location (defined with --outdir)
.nextflow_log       # Log file from Nextflow
# Other nextflow hidden files, eg. history of pipeline runs and old logs.
```
### Unsure whether this section is a generalisation and should stay as is?

If you wish to repeatedly use the same parameters for multiple runs, rather than specifying each flag in the command, you can specify these in a params file.

Pipeline settings can be provided in a `yaml` or `json` file via `-params-file <file>`.

> ⚠️ Do not use `-c <file>` to specify parameters as this will result in errors. Custom config files specified with `-c` must only be used for [tuning process resource specifications](https://nf-co.re/docs/usage/configuration#tuning-workflow-resources), other infrastructural tweaks (such as output directories), or module arguments (args).

The above pipeline run specified with a params file in yaml format:

```bash
nextflow run wal-yan/target-methylseq-qc -profile docker -params-file params.yaml
```

with `params.yaml` containing:

```yaml
input: './samplesheet.csv'
outdir: './results/'
genome: 'GRCh37'
<...>
```

You can also generate such `YAML`/`JSON` files via [nf-core/launch](https://nf-co.re/launch).

### Updating the pipeline

When you run the above command, Nextflow automatically pulls the pipeline code from GitHub and stores it as a cached version. When running the pipeline after this, it will always use the cached version if available - even if the pipeline has been updated since. To make sure that you're running the latest version of the pipeline, make sure that you regularly update the cached version of the pipeline:

```bash
nextflow pull wal-yan/target-methylseq-qc
```

### Reproducibility

It is a good idea to specify a pipeline version when running the pipeline on your data. This ensures that a specific version of the pipeline code and software are used when you run your pipeline. If you keep using the same tag, you'll be running the same version of the pipeline, even if there have been changes to the code since.

First, go to the [wal-yan/target-methylseq-qc releases page](https://github.com/wal-yan/target-methylseq-qc/releases) and find the latest pipeline version - numeric only (eg. `1.3.1`). Then specify this when running the pipeline with `-r` (one hyphen) - eg. `-r 1.3.1`. Of course, you can switch to another version by changing the number after the `-r` flag.

This version number will be logged in reports when you run the pipeline, so that you'll know what you used when you look back in the future. For example, at the bottom of the MultiQC reports.

To further assist in reproducbility, you can use share and re-use [parameter files](#running-the-pipeline) to repeat pipeline runs with the same settings without having to write out a command with every single parameter.

> 💡 If you wish to share such profile (such as upload as supplementary material for academic publications), make sure to NOT include cluster specific paths to files, nor institutional specific profiles.

## Core Nextflow arguments

> **NB:** These options are part of Nextflow and use a _single_ hyphen (pipeline parameters use a double-hyphen).

### `-profile`

Use this parameter to choose a configuration profile. Profiles can give configuration presets for different compute environments.

Several generic profiles are bundled with the pipeline which instruct the pipeline to use software packaged using different methods (Docker, Singularity, Podman, Shifter, Charliecloud, Apptainer, Conda) - see below.

> We highly recommend the use of Docker or Singularity containers for full pipeline reproducibility, however when this is not possible, Conda is also supported.

The pipeline also dynamically loads configurations from [https://github.com/nf-core/configs](https://github.com/nf-core/configs) when it runs, making multiple config profiles for various institutional clusters available at run time. For more information and to see if your system is available in these configs please see the [nf-core/configs documentation](https://github.com/nf-core/configs#documentation).

Note that multiple profiles can be loaded, for example: `-profile test,docker` - the order of arguments is important!
They are loaded in sequence, so later profiles can overwrite earlier profiles.

If `-profile` is not specified, the pipeline will run locally and expect all software to be installed and available on the `PATH`. This is _not_ recommended, since it can lead to different results on different machines dependent on the computer enviroment.

- `test`
  - A profile with a complete configuration for automated testing
  - Includes links to test data so needs no other parameters
- `docker`
  - A generic configuration profile to be used with [Docker](https://docker.com/)
- `singularity`
  - A generic configuration profile to be used with [Singularity](https://sylabs.io/docs/)
- `podman`
  - A generic configuration profile to be used with [Podman](https://podman.io/)
- `shifter`
  - A generic configuration profile to be used with [Shifter](https://nersc.gitlab.io/development/shifter/how-to-use/)
- `charliecloud`
  - A generic configuration profile to be used with [Charliecloud](https://hpc.github.io/charliecloud/)
- `apptainer`
  - A generic configuration profile to be used with [Apptainer](https://apptainer.org/)
- `conda`
  - A generic configuration profile to be used with [Conda](https://conda.io/docs/). Please only use Conda as a last resort i.e. when it's not possible to run the pipeline with Docker, Singularity, Podman, Shifter, Charliecloud, or Apptainer.

### `-resume`

Specify this when restarting a pipeline. Nextflow will use cached results from any pipeline steps where the inputs are the same, continuing from where it got to previously. For input to be considered the same, not only the names must be identical but the files' contents as well. For more info about this parameter, see [this blog post](https://www.nextflow.io/blog/2019/demystifying-nextflow-resume.html).

You can also supply a run name to resume a specific run: `-resume [run-name]`. Use the `nextflow log` command to show previous run names.

### `-c`

Specify the path to a specific config file (this is a core Nextflow command). See the [nf-core website documentation](https://nf-co.re/usage/configuration) for more information.

## Custom configuration

### Resource requests

Whilst the default requirements set within the pipeline will hopefully work for most people and with most input data, you may find that you want to customise the compute resources that the pipeline requests. Each step in the pipeline has a default set of requirements for number of CPUs, memory and time. For most of the steps in the pipeline, if the job exits with any of the error codes specified [here](https://github.com/nf-core/rnaseq/blob/4c27ef5610c87db00c3c5a3eed10b1d161abf575/conf/base.config#L18) it will automatically be resubmitted with higher requests (2 x original, then 3 x original). If it still fails after the third attempt then the pipeline execution is stopped.

To change the resource requests, please see the [max resources](https://nf-co.re/docs/usage/configuration#max-resources) and [tuning workflow resources](https://nf-co.re/docs/usage/configuration#tuning-workflow-resources) section of the nf-core website.

### Custom Containers

In some cases you may wish to change which container or conda environment a step of the pipeline uses for a particular tool. By default nf-core pipelines use containers and software from the [biocontainers](https://biocontainers.pro/) or [bioconda](https://bioconda.github.io/) projects. However in some cases the pipeline specified version maybe out of date.

To use a different container from the default container or conda environment specified in a pipeline, please see the [updating tool versions](https://nf-co.re/docs/usage/configuration#updating-tool-versions) section of the nf-core website.

### Custom Tool Arguments

A pipeline might not always support every possible argument or option of a particular tool used in pipeline. Fortunately, nf-core pipelines provide some freedom to users to insert additional parameters that the pipeline does not include by default.

To learn how to provide additional arguments to a particular tool of the pipeline, please see the [customising tool arguments](https://nf-co.re/docs/usage/configuration#customising-tool-arguments) section of the nf-core website.

### nf-core/configs

In most cases, you will only need to create a custom config as a one-off but if you and others within your organisation are likely to be running nf-core pipelines regularly and need to use the same settings regularly it may be a good idea to request that your custom config file is uploaded to the `nf-core/configs` git repository. Before you do this please can you test that the config file works with your pipeline of choice using the `-c` parameter. You can then create a pull request to the `nf-core/configs` repository with the addition of your config file, associated documentation file (see examples in [`nf-core/configs/docs`](https://github.com/nf-core/configs/tree/master/docs)), and amending [`nfcore_custom.config`](https://github.com/nf-core/configs/blob/master/nfcore_custom.config) to include your custom profile.

See the main [Nextflow documentation](https://www.nextflow.io/docs/latest/config.html) for more information about creating your own configuration files.

If you have any questions or issues please send us a message on [Slack](https://nf-co.re/join/slack) on the [`#configs` channel](https://nfcore.slack.com/channels/configs).

## Azure Resource Requests

To be used with the `azurebatch` profile by specifying the `-profile azurebatch`.
We recommend providing a compute `params.vm_type` of `Standard_D16_v3` VMs by default but these options can be changed if required.

Note that the choice of VM size depends on your quota and the overall workload during the analysis.
For a thorough list, please refer the [Azure Sizes for virtual machines in Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes).

## Running in the background

Nextflow handles job submissions and supervises the running jobs. The Nextflow process must run until the pipeline is finished.

The Nextflow `-bg` flag launches Nextflow in the background, detached from your terminal so that the workflow does not stop if you log out of your session. The logs are saved to a file.

Alternatively, you can use `screen` / `tmux` or similar tool to create a detached session which you can log back into at a later time.
Some HPC setups also allow you to run nextflow within a cluster job submitted your job scheduler (from where it submits more jobs).

## Nextflow memory requirements

In some cases, the Nextflow Java virtual machines can start to request a large amount of memory.
We recommend adding the following line to your environment to limit this (typically in `~/.bashrc` or `~./bash_profile`):

```bash
NXF_OPTS='-Xms1g -Xmx4g'
```
