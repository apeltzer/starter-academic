+++
title = "Affordable WGS for personal usage (part 1)"

date = 2019-04-28T00:00:00
lastmod = 2019-04-28T20:00:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Alexander Peltzer"]

tags = ["Bioinformatics", "NF-Core", "Nextflow", "WGS", "Genetics"]
summary = "Analyzing my own genome data from DanteLabs WGZ service."
+++

## How and why I did it

I was already interested in sequencing my own genome during my time as a Ph.D. student, when the first 23andme & other genotyping services were available publicly. After all, back then this wasn't as exciting as I thought due to quite restricted resolution. Having access to the full genome-wide data was something I didn't think about for a while.

During last years Black Friday event, I decided to take the opportunity to pay for my own genome to be sequenced using [DanteLabs](http://dantelabs.com/) WGZ "Black Friday Offer". After the payment, I figured out that in order to get the final [BAM](https://en.wikipedia.org/wiki/SAM_(file_format)) or [FastQ](https://en.wikipedia.org/wiki/FASTQ_format) file formats that you need to additionally pay US$ 50 to get an external harddisk with your "real" RAW data.

Without that, you will receive VCF file(s) for SNPs, CNVs, SVs, INDELs and two PDFs with two individual reports for medications and health-related insights. The Bioinformatician in me was however more interested in doing everything from scratch which I'm going to explicitly elucidate here a bit upon. The data looks very similar to standard Illumina data I already have some experience with, so standard WGS/WES pipeline analysis was the way to go for looking into it in more detail. Feel free to ask me on [Twitter](https://twitter.com/alex_peltzer/status/1122589887054721026) for more details - I might update this or do several follow ups on this posting to keep people with interest in the loop.

## Delivery

Upon registering and paying upfront, I got my swab kit approximately 2 weeks after payment. Immediately after sending it in, it took another about 4 weeks to get confirmation that DNA extraction went successful and I got access to the data at the website to at least download the two pdf report and my VCF files.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2019-04-28_wgz/01_report.png)

Upon receiving this, it took another 6 weeks until I got my RAW data in April 2019. The harddisk has 500GB and contained raw data with compressed FastQ (split across multiple lanes), some QC files, BAM and Index for that final BAM and the called Variants found on the website as VCF files.

## First Analysis Steps

I decided to utilize a pipeline for modern WGS sequencing to analyze my data in a single run. My choice for this job was the [Sarek](https://github.com/SciLifeLab/Sarek) pipeline which can do all of the required steps for such an analysis in an easy manner, including QC, mapping, variant calling, annotation and reporting.

### Step 1: Create a TSV file

First step was to create a TSV file with my data in it. I anonymized some parts here for privacy reasons but it should be quite straightforward [what the columns mean](https://github.com/SciLifeLab/Sarek/blob/master/docs/INPUT.md).

```bash
Alex    XY      0       Alex_Normal     15180   RAW/foo15180_L01_543_1.fastq.gz RAW/foo15180_L01_543_2.fastq.gz
Alex    XY      0       Alex_Normal     525     RAW/foo_L01_525_1.fastq.gz  RAW/foo_L01_525_2.fastq.gz
Alex    XY      0       Alex_Normal     526     RAW/foo_L01_526_1.fastq.gz  RAW/foo_L01_526_2.fastq.gz
Alex    XY      0       Alex_Normal     527     RAW/foo_L01_527_1.fastq.gz  RAW/foo_L01_527_2.fastq.gz
Alex    XY      0       Alex_Normal     528     RAW/foo_L01_528_1.fastq.gz  RAW/foo_L01_528_2.fastq.gz
Alex    XY      0       Alex_Normal     529     RAW/foo_L01_529_1.fastq.gz  RAW/foo_L01_529_2.fastq.gz
Alex    XY      0       Alex_Normal     530     RAW/foo_L01_530_1.fastq.gz  RAW/foo_L01_530_2.fastq.gz
Alex    XY      0       Alex_Normal     531     RAW/foo_L01_531_1.fastq.gz  RAW/foo_L01_531_2.fastq.gz
Alex    XY      0       Alex_Normal     532     RAW/foo_L01_532_1.fastq.gz  RAW/foo_L01_532_2.fastq.gz
```

### Step 2: Run Mapping / QC

Second, I started the mapping and initial QC for the data:

```bash
nextflow run SciLifeLab/Sarek/main.nf -r dev -profile standard,docker --genome 'GRCh37' --sample sarek.tsv -resume
```

### Step 3: Run VariantCalling

As I was uncertain which VariantCallers would give me best results, I just selected all of the variant callers available in Sarek for germline data:

```bash
nextflow run SciLifeLab/Sarek/germlineVC.nf -profile standard,docker -r dev --genome 'GRCh37' --sample Preprocessing/Recalibrated/recalibrated.tsv --tools 'HaplotypeCaller,strelka,manta' -resume
```

### Step 4: Annotate the results & create a report

Having only the raw variants isn't really useful without annotation:

```bash
nextflow run SciLifeLab/Sarek/annotate.nf -profile standard,docker -r dev --genome 'GRCh37' --annotateTools 'HaplotypeCaller,strelka,manta' --tools 'snpEff'
nextflow run SciLifeLab/Sarek/runMultiQC.nf -r dev -profile standard,docker
```

## Some basic statistics and information

### General QC

The data looks quite good in terms of basic metrics on the raw data.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2019-04-28_wgz/fastqc_per_base_sequence_quality_plot.png)

As I mapped against the GRCh37 genome reference, all these reported values are against that. In total, 936 Million reads were properly mapped, with a 41%GC content. A total coverage of 31X was achieved, with 90.6% of the genome covered at least with 10X, and 56.7% of the genome covered with at least 30X as reported by the Sarek reports.  

### HLATyping

I also extracted all of the reads on Chromosome 6:

```bash
samtools view -bh -o chr6.alex.bam input.bam "6"
nextflow run nf-core/hlatyping --bam -profile docker --reads 'chr6.alex.bam'
```

and HLA-typed myself using the [nf-core/hlatyping](https://github.com/nf-core/hlatyping) pipeline. Might be just the same curiousity but this worked as well using the data provided and provided me with my HLA-A, HLA-B and HLA-C types. Doing this wasn't possible with the 23andme test back then for example.

### mtDNA / Y Typing

Using a simple command, I was able to extract and call the consensus sequence of my mtDNA and type myself to confirm my previous 23and me genotyping results. Fortunately, the [Haplogrep 2](https://haplogrep.uibk.ac.at/) service and the [Y-Haplo](https://github.com/23andMe/yhaplo) teams provide their services for everyone and uploading my data there to determine my mtDNA haplogroup and my Y-haplogroup was quit easy, too.

## Future updates

There will be follow up posts once I got more time to look at my data. I will probably integrate my data with population genetics datasets such as Human Origins and also have a more detailed look at some variant analysis methods.