# snakemake-resources

_The purpose of this document is to bookmark and annotate specific resources useful for making snakemake pipelines._

## basic snakemake syntax, tutorials, and concepts

* syntax and tutorial
  * https://hpc-carpentry.github.io/hpc-python/

* concepts 
  * Directed Acyclic Graph (DAG): 

## functions

* expand() and glob_wildcards()
  * https://hpc-carpentry.github.io/hpc-python/15-snakemake-python/
* lambda wildcards
  * link
* functions as input files 

## advanced snakemake 

* parallelization
* config files
* input functions
* params
* logs
* temp 
* protected files

https://snakemake.readthedocs.io/en/stable/tutorial/advanced.html

* version tracking
  * link

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

2.  If you have large snakemake jobs and your DAG is crowded and hard to understand, you can use --rulegraph to visualize the workflow at the rule level rather than the sample / wildcards resolution with the following command:

   `snakemake --rulegraph | dot -Tsvg > ruleGraph.svg`



