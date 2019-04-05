---
title: The slash r character in Linux
author: ''
date: '2019-03-25'
slug: the-slash-r-character-in-linux
categories: []
tags: []
---

## Background

I have 1451 vcf files called from samples from different cell lines, and I want to move those from MCF7 cell line to a folder named `MCF7`. I have a txt file named `MCF7.txt`, with each line corresponds to the file name of a MCF7 vcf file. I run the following script in Linux:

```
#!/bin/sh

#SBATCH --ntasks-per-node 20  # cores 
#SBATCH --array=1-810%80

nCores=10
module use /cm/shared/modulefiles_local

cd /gpfs/scratch/juan.xie/SNP_single

ID=$( cat /gpfs/scratch/juan.xie/IDs/single/MCF7.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

mv $ID.ann.filtered.vcf MCF7

```

However, it failed, and the error message looked like the following:
`mv: cannot stat SRR060899\r.ann.filtered.vcf: No such file or directory`

## Solution

I found that there is a `\r` in the error message, and I was suspecting that may be the reason. Googling confirmed my guess(https://superuser.com/questions/374028/how-are-n-and-r-handled-differently-on-linux-and-windows). So I run the following code:

```
sed -i 's/\r//' MCF7.txt 

```
Then run the previous script and it worked.