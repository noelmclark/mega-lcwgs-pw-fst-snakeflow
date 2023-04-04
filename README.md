
<!-- README.md is generated from README.Rmd. Please edit that file -->

# mega-lcwgs-pw-fst-snakeflow

<!-- badges: start -->
<!-- badges: end -->

This is a simple Snakemake workflow for calculating pairwise Fsts (and
related quantities, eventually, I suppose) from BAM files using ANGSD.

Note that the `mega` in the name doesn’t refer to anything that I think
is “millions-esque” about this project. It is simply one of the
workflows we use in the **M**olecular **E**cology and **G**enetic
**A**nalysis (MEGA) Team at NOAA’s Southwest Fisheries Science,
Fisheries Ecology Division.

Setting up this workflow for yourself is pretty easy:

1.  You have to put appropriate values in the config file for your own
    data set
2.  You need a `bams.tsv` file that gives that paths to the BAM files
    and tells what group (population) each such sample/bam belongs to.
    **Spaces are not allowed in the bam file paths.**
3.  You need a `pwcomps.tsv` that lists the pairwise comparisons from
    the groups in `bams.tsv` that you wish to calculate pairwise Fsts
    for.
4.  To use `winsfs` (highly recommended because it is orders of
    magnitude faster than `realSFS`), you need to install it and make
    sure it is in your path (you don’t have to do this for ANGSD, since
    that is available on conda). Follow the installation directions at
    <https://github.com/malthesr/winsfs#installation>

This workflow comes with a small set of data and config files for
testing in the `.test` directory. This is a good place to go to see
examples of the `config` files, etc. I print them out below here, too:

**.test/config/config.yaml**

``` yaml
bams: ".test/config/bams.tsv"
pwcomps: ".test/config/pwcomps.tsv"
genome: "resources/genome.fasta"

modes: ["BY_CHROM"]
chroms: ".test/config/chroms.tsv"


params:
  angsd_bam_filters: " -minMapQ 20  -minQ 33 -baq 1 "
  calc_saf: " -dosaf 1 -gl 1 "
  realSFS: " -fold 1 "
  winsfs: " "
  
```

**.test/config/bams.tsv**

``` yaml
sample  path    group
s001    .test/data/bams/s001.bam    grpA
s002    .test/data/bams/s002.bam    grpA
s003    .test/data/bams/s003.bam    grpB
s004    .test/data/bams/s004.bam    grpB
s005    .test/data/bams/s005.bam    grpB
s006    .test/data/bams/s006.bam    grpC
s007    .test/data/bams/s007.bam    grpC
```

**.test/config/pwcomps.tsv**

``` yaml
pop1    pop2
grpA    grpB
grpA    grpC
grpB    grpC
```

## Modes

When I started this, I thought I would see what happened if we just did
the whole genome. That runs into big memory issues. So now I am working
though having different *modes* one of which is “BY_CHROM” which will do
things by chromosome. This will, I suspect, become the default mode.

## Reasons for some choices

See:
<http://www.popgen.dk/angsd/index.php/SFS_Estimation#Folded_spectra>
where it says, “If you don’t have the ancestral state, you can instead
estimate the folded SFS. This is done by supplying the -anc with the
reference genome and applying -fold 1 to realSFS.”

And then when we are doing Fst, it is relevant at:
<http://www.popgen.dk/angsd/index.php/Fst#Note_about_fst_for_folded_spectra>
where we see:

- Earlier versions of angsd/realSFS could output folded 1d sample allele
  frequencies which would be usefull for 1population neutrality test
  like Tajima. This is however not appropriate to use for calculating
  fst since the folding was done within population.
- We have therefore added a proper folding procedure for the
  optimization based on the UNFOLDED .saf.idx files generated by -doSaf.
  These are the ones that should be used for calculating fst.
- Therefore please remember to add -fold 1 if you want angsd (the
  realSFS subfunction) to perform fst and pbs estimation using the
  folded spectra.

So, we will be adding `-fold 1` to the `realSFS` commands done with the
`params` setting in the config file.

As far as doing it over chromosomes. It seems to me that both angsd and
realSFS can only take one argument to the `-r` option. So, it seems that
won’t let you put multiple scaffolds together, which is kinda messed up.
Maybe I am missing something. For that use case (i.e. when you only have
a lot of scaffolds) then I suppose one could try to use nSites or one
could just explicitly grab each region from the bam and make a new bam.
If I ever have a genome that is in lots of pieces like that, I will do
it that way. For now I am focusing on single chromosomes.
