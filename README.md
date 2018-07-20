# pretzel-input-generator

This is a [nextflow](https://www.nextflow.io) pipeline for generating input for [pretzel](https://github.com/plantinformatics/pretzel) from annotated and (mostly) contiguous genome assemblies. 



## Input

Input files are specified in [conf/input.config](conf/input.config). This can be supplemented/replaced by JSON/YAML formatted input spec.  

### Data sources

Currently all input data comes from the following sources:

* [Ensembl plants](https://plants.ensembl.org) - multiple datasets as specified in [`conf/input.config`](conf/input.config)
* [International Wheat Genome Sequencing Consortium](https://www.wheatgenome.org/)
  * [Triticum aestivum (Chinese Spring) IWGSC RefSeq v1.0 assembly](https://wheat-urgi.versailles.inra.fr/Seq-Repository/Assemblies)
* [The wild emmer wheat sequencing consortium (WEWseq)](http://wewseq.wixsite.com/consortium)
  * Zavitan assembly downloaded from [GrainGenes](https://wheat.pw.usda.gov/GG3/wildemmer)
  * Publication: [Wild emmer genome architecture and diversity elucidate wheat evolution and domestication. Raz Avni *et al.*, Science  07 Jul 2017, Vol. 357, Issue 6346, pp. 93-97 DOI: 10.1126/science.aan0032](http://science.sciencemag.org/content/357/6346/93)


### Remote

The pipeline pulls data from [Ensembl plants](https://plants.ensembl.org), included species and assembly versions are specified in [conf/input.config](conf/input.config). 
For each of the datsets the pipeline downloads:

* genome assembly index file 
* protein sequences 

### Local

The pipeline requires 

* a genome assembly index file
* gene annotations (either GTF or GFF3)
* matching protein sequences (presumably for representative isoform) 

Currently, some datasets are specified and to run this pipeline without the specified files, use `--localAssembly NA` at execution or modify your local copy of [conf/input.config](conf/input.config)

## Dependencies

* [nextflow](https://www.nextflow.io) 
* **Either** of the following:
  * [Singularity](http://singularity.lbl.gov)
  * [Docker](http://singularity.lbl.gov)
  * Required software installed (e.g. as module, in which case specify its name in [`conf/modules.config`](conf/modules.config)). In addition to standard linux tools, these include:
    * [FASTX-Toolkit](http://hannonlab.cshl.edu/fastx_toolkit/)
    * [MMSeqs2](https://github.com/soedinglab/mmseqs2)

When using Singularity or Docker, the required containers are specified in [`conf/containers.conf`](conf/containers.config)
 

## Execution

We provide several execution profiles, "locally" may mean a designated server or an interactive session on a cluster. By appending  e.g. `-revision v0.3` to your command you can specify a release tag to run a revision that may be more likely to be working than the latest. When re-running the pipeline after errors or changes use `-resume` to ensure only the necessary processes are re-run.

Run locally with docker

```
nextflow run plantinformatics/pretzel-input-generator \
-profile docker --localAssembly NA
```

Run locally with singularity

```
nextflow run plantinformatics/pretzel-input-generator \
-profile singularity --localAssembly NA
```

Dispatch on a SLURM cluster with singularity

```
nextflow run plantinformatics/pretzel-input-generator \
-profile slurm,singularity,singularitymodule --localAssembly NA
```

Dispatch on a SLURM cluster with modules (defined in [conf/modules.config](conf/modules.config))

```
nextflow run plantinformatics/pretzel-input-generator \
-profile slurm,modules --localAssembly NA
```

## Output

All generated JSON files generated by the pipeline are output to `results/JSON`. 

* For each of the input genome assemblies, these include:
  * `*_genome.json` - dataset (genome) definitions specifying outer coordinates of blocks (chromosomes)
  * `*_annotation.json` - specifications of coordinates of features (genes) within blocks
* In addition, for each (lexicographically ordered) pair of genome assemblies, the pipeline generates:
  * `*_aliases.json` which specify links between features between the two genomes.

The output files (hopefully) conform to the requirements of [pretzel data structure](https://github.com/plantinformatics/pretzel-data). 


The `results/flowinfo` directory contains summaries of pipeline execution.

```
results
├── flowinfo
└── JSON
```

To upload the generated data to your instance of pretzel, follow [these instructions](upload.md). 


## Pipeline overview

The pipeline requires approximately 1 cpu-day, but as many processes can run independently, the real run-time is much shorter.

If `-with-dag dag.dot` is specified, nextflow outputs a DOT language representation of the pipeline, as presented below. 

![doc/dag.png](doc/dag.png)
