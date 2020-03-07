# snakemake-resources

_The purpose of this document is to bookmark and annotate specific resources useful for making snakemake pipelines._

![alt text](https://nbis-reproducible-research.readthedocs.io/en/latest/images/rulegraph_mrsa.svg "fig1")

## What is snakemake?

[SnakeMake](https://academic.oup.com/bioinformatics/article/28/19/2520/290322) is a pipeline manager written in python developed by Johannes Köster  and Svenn Rahmann. It has been developed with a bioinformatic  application in mind, and I happen to use SnakeMake a lot during my work  at Stowers. After 6+ months of using it, I now understand the value in  investing time and effort to make your pipelines portable and  reproducible – either for reviewers, co-workers around you, or even  future you. Thus I’ve decided to gather the internet’s most useful  resources in SnakeMake as a reference sheet for everyone, so enjoy!  

## basic snakemake syntax, tutorials, and concepts

* syntax and tutorial
  * [Original snakemake docs](snakemake-resources) by Johannes Köster  and Svenn Rahmann. 
    * The OG documentation, comprehensive, very useful FAQ section, though sometimes tersely written. 
  * [analysis pipelines with python](https://hpc-carpentry.github.io/hpc-python/) by Software Carpentry Foundation.
    * Short and sweet, enough to get started. Talks about snakefiles, wildcards, patterns, parallelism, scaling on HPC, and tricks. 
  * [SciLifeLab, National Bioinformatics Infrastructure Sweden (NBIS) Reproducible research course](https://nbis-reproducible-research.readthedocs.io/en/latest/about/) by Leif Wigge, Rasmus Ågren, John Sundh, Verena Kutschera, and Erik Fasterius. 
    * Very beginner friendly, and highly recommended if you're new to coding and / or bioinformatics. 
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
  * how to parallelize rules
  * set up your config files
  * define input functions
  * add parameters to your command 
  * make log files
  * protect files from being deleted

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

   This will produce an image like the first figure in this post. Neat!

3. [How to run only one rule in snakemake](https://stackoverflow.com/questions/55831923/how-to-run-only-one-rule-in-snakemake) by BioManil at StackExchange. 

## What do you like about snakemake?

1. Integration of external scripts written in R, python, and Julia. I usually make snakemake run an Rmd file to reproducibly create interactive analysis reports and figures in html format. All you have to do is define the input files in a snakemake rule, write the Rmd to accommodate the new variables, and you're good to go. Example below:

   ```python
   # python rule
   rule all:
   	"reports/analysis_report.html"
   
   rule analysisReport:
   	input: 
   		wt = "data/wildtype.fastq",
   		case = "data/case.fastq"
   	output:
   		"reports/analysis_report.html"
   	script: "code/scripts/analysis_report.Rmd"
   ```

   And the corresponding script in the `analysis_report.Rmd` file:

   ```R
   ---
   yaml header with metadata
   ---
   
   ​```{r}
   wt <- snakemake@input[["wt"]]
   case <- snakemake@input[["case"]]
   ...
   ​```
   
   # continue with the analysis below.
   ```

   Doing things this way is convenient because you can prototype and finalize the Rmd script as you normally do, then all you have to do is switch the input file to the snakemake S4 object with `wt <- snakemake@input[["wt"]]` and `case <- snakemake@[["case"]]`, and **you've successfully hooked up the Rmd to your snakemake pipeline.** No really, that's it! No extra commands are required in the body of the Rmd script. 

   Although the original snakemake docs says that the only output of an Rmd attachment is the html report, I've had success in writing out non-report files - e.g. like GATK interval files. 

2. Snakemake works with shared-resource clusters and job schedules - think SLURM - right out of the box. 

## Are there any cons to snakemake? 

The biggest gripe that I have with snakemake is that its error messages aren't as clear as they can be. 

There can only be one snakemake process running, so it's difficult to work on later part of pipeline if you're running a large job. This is adjustable with --no-lock option, but it's risky. 

Jupyter Notebook integration and ipython3 help make SnakeMake more interactive, but SM is mainly a command-line based tool, and will take a long time to generate the DAG if the pipeline gets large. 

## Miscellaneous

Upgrade your interactive RMarkdown reports here: [Pimp my Rmd](https://holtzy.github.io/Pimp-my-rmd/) by Yan Holtz

