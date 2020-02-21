# snakemake-resources

_The purpose of this document is to bookmark and annotate specific resources useful for making snakemake pipelines._

## basic snakemake syntax, tutorials, and concepts

* syntax and tutorial
  * [analysis pipelines with python](https://hpc-carpentry.github.io/hpc-python/) by Software Carpentry Foundation.
    * Short and sweet, enough to get started. Talks about snakefiles, wildcards, patterns, parallelism, scaling on HPC, and tricks. 
  * [reproducible research workflows with snakemake and r](https://lachlandeer.github.io/snakemake-econ-r-tutorial/logging-output-and-errors.html) by Lachlan Deer and Julian Langer
    * This guide is well structured and organized. Starts simple, shows advanced features (logging, config, sub-workflows, glob_wildcards, containers), and integration with R + Rmd (useful for making reproducible figures). Totally awesome. 

* concepts 
  * Directed Acyclic Graph (DAG) [simple](https://www.statisticshowto.datasciencecentral.com/directed-acyclic-graph/) and [complex](https://en.wikipedia.org/wiki/Directed_acyclic_graph). 
    * Basically, a DAG is an acyclic graph in which nodes are connected by edges with a direction. 
    * As the name suggests, these are never cyclic. 

## example of snakemake functions

* [expand() and glob_wildcards()](https://hpc-carpentry.github.io/hpc-python/15-snakemake-python/) by Software Carpentry Foundation
* [accessing wildcard values in a snakemake rule](http://tinyheero.github.io/2019/08/30/wildcards-in-snakemake.html) by Fong Chun Chan. 
* [functions as input files](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html) in original snakemake docs
* [mixing globbing and wildcards when specifying rule input](https://bioinformatics.stackexchange.com/questions/7184/mix-globbing-and-wildcards-when-specifying-rule-input) in bioinformatic stack exchange. 

## advanced snakemake 

* [original snakemake docs](https://snakemake.readthedocs.io/en/stable/tutorial/advanced.html) details the following:
  * parallelization
  * config files
  * input functions
  * params
  * logs
  * temp 
  * protected files

## personal experience + tips

1. snakemake does **not** log shell commands if invoked by python shell function like: 

   ```python
   rule exampleRule1:
   input: expand("data/exampleInputFolder/{sample}.fastq")
   output: expand("data/exampleOutputFolder/{sample}.bam")
   run: shell("<command> <option> <arg> {input} {output}")
   ```

   Only shell commands specified in the rule will be logged like below:

   ```python
   rule exampleRule2: 
   input: expand("data/exampleInputFolder/{sample}.fastq")
   output: expand("data/exampleOutputFolder/{sample}.bam")
   shell: "<command> <option> <arg> {input} {output}"
   ```

   This is unfortunate since you cannot see the literal shell commands in dry runs (`snakemake -np`), and the snakemake report will either not render correctly or not include the shell commands in the first scenario. 

2. If you have large snakemake jobs and your DAG is crowded and hard to understand, you can use `--rulegraph` argument to visualize the workflow at the rule level rather than the sample / wildcards resolution with the following command:

   `snakemake --rulegraph | dot -Tsvg > ruleGraph.svg`
