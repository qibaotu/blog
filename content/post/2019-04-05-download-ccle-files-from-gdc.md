---
title: download CCLE files from GDC
author: ''
date: '2019-04-05'
slug: download-ccle-files-from-gdc
categories:
  - Bioinformatics
tags:
  - GDC data
---

I wanted to download bam files corresponding to the CCLE cell lines, and a paper mentioned that they downloaded these files from [GDC](https://portal.gdc.cancer.gov/). I spent 2~3 hrs exploring the website yet couldn't find such files. Finally, by emailing the authors, it is found that these data are under the `Legacy Archive`(the right bottom of the data portal page). Clicking the button, it will direct to https://portal.gdc.cancer.gov/legacy-archive/search/f. And on the left side of the page, we can select *CCLE* under the *Cancer Program* to choose CCLE data.

For downloading, I used the `gdc-client` tool. At first, I downloaded the manifest file(which contain information for all 82 bam files to-be-downloaed) and run the script on HPC:
```
gdc-client download --no-segment-md5sums --no-file-md5sum -m /gpfs/scratch/juan.xie/CCLE/gdc_manifest.2019-04-04.txt
```

However, it is found that the downloading is very time consuming(I have 82 bam files to download, yet 24 hrs passed and only 13 were downloaded ). Thus I was thinking of split the job into multiple ones. The gdc-client doc mentioned that we can download via UUIDs, and it is found that the manifest file actualy has that information in the first column. Thus I extracted that column, and run the following script:

```
#!/bin/sh
#SBATCH -J downloadCCLE          # Job name
#SBATCH -o myjob.%j.out   # define stdout filename; %j expands to jobid
#SBATCH -e myjob.%j.err   # define stderr filename; skip to combine stdout and stderr

#SBATCH -N 1              # Number of nodes, not cores (16 cores/node)
#SBATCH -t 120:00:00       # max time
#SBATCH --ntasks-per-node 20  
#SBATCH --array=1-100%20

#SBATCH --partition=test        # Partition/Queue

nCores=10

cd /gpfs/scratch/juan.xie/CCLE
ID=$( cat /gpfs/scratch/juan.xie/CCLE/UUID.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)


# gdc-client download --no-segment-md5sums --no-file-md5sum -m /gpfs/scratch/juan.xie/CCLE/gdc_manifest.2019-04-04.txt
gdc-client download --no-segment-md5sums --no-file-md5sum $ID

```
