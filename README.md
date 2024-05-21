# CpG_browser

Documentation for how CpG methylation(5mC, 5hmC, and duplex hemi-methylation) tracks were generated. Meant for generating bigwig files of methylation densities for hosting on the UCSC Genome Browser.

#### Input Data:
Generated using [ONTs modkit](https://github.com/nanoporetech/modkit) to aggregate MM/ML tags from individual reads across CpG sites in the reference genome. Installed by downloading from [their github](https://github.com/nanoporetech/modkit). Below is the settings used for all of the pileups.  

Simplex Data:
```
modkit pileup \
    -t <threads> \
    --cpg \
    --ref <reference_fasta> \
    --num-reads 100000 \
    --filter-percentile 0.666 \
    --force-allow-implicit \
    --combine-strands \
    --only-tabs \
    <input> <output>
```
Duplex Data:
```
modkit pileup-hemi \
    -t <threads> \
    --cpg \
    --ref <reference_fasta> \
    --num-reads 100000 \
    --filter-percentile 0.666 \
    --force-allow-implicit \
    --combine-strands \
    --only-tabs \
    <input> <output>
```


### 5mC and 5hmC Tracks
After running `modkit pileup` some post-processing is run to make the bed files into bedgraph files. This format is makes a barplot style graph, which is better at visualizing methylation density.   
This can be done in two ways:
1. Using the fraction modified    
    `awk 'BEGIN {FS=OFS="\t"} $10 >= 5 {print $1, $2, $3, $11}' <input> > <output>`    
2. Using the modified base coverage       
    `awk 'BEGIN {FS=OFS="\t"} $10 >= 5 {print $1, $2, $3, $12}' <input> > <output>`     

After this you are set for viewing in either `IGV` or `UCSC Genome Browser`. However in order to host in a Browser Hub on UCSC Browser the file will need to be converted to a `bigWig` using [`ucsc-bedgraphtobigwig`](https://bioconda.github.io/recipes/ucsc-bedgraphtobigwig/README.html). (Note if installed with conda when environment is activated you need to run `conda install -c bioconda openssl=1.0`)

### Duplex Hemi-Methylation Tracks
Duplex data processing is a bit different. Here I exracted CpG sites where methylation only occurs on one strand on the DNA. In the `pileup-hemi` output this is denoted by either a `m,-,C`(plus strand hemi-methylation) OR `-,m,C`(minus strand hemi-methylation).    
These two patterns can be grep'd for with something like: `grep -P '\tm,-,C\t' <input> > <output>`.   
This is done for both hemi-methylation types, then similar post-processing to simplex is followed:
1. Using the fraction modified    
    `awk 'BEGIN {FS=OFS="\t"} $10 >= 5 {print $1, $2, $3, $11}' <input> > <output>`    
2. Using the modified base coverage    
    `awk 'BEGIN {FS=OFS="\t"} $10 >= 5 {print $1, $2, $3, $12}' <input> > <output>`