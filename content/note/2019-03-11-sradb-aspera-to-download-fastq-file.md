---
title: SRAdb+Aspera to download fastq file
author: ''
date: '2019-03-11'
slug: sradb-aspera-to-download-fastq-file
categories:
  - Bioinformatics
tags: []
---

Background:

I used to use *prefetch* to download large batch of fastq/sra files. Yet for some reasons the bash script failed to run, and I had to type the command under the interactive mode, which often loged me out. I remembered that NCBI mentioned *Aspera* is faster than *prefetch*, and I also found that Dr. Ge used *SRAdb* + *Aspera* to download data, thus I decided to have a try.

Fisrt, I installed *SRAdb* package, downloaded and installed *Aspera connect*. Then I tried to run the following Rscript:

```css
library(SRAdb)

setwd('/gpfs/scratch/juan.xie/BreastCancer2/')
sqlfile = getSRAdbFile() 
sra_con = dbConnect(SQLite(), sqlfile)
rs1 <-read.table('/gpfs/scratch/juan.xie/IDs/breast_cell_lines_IDs_paired',header=T)

ascpCMD <- 'ascp -QT -l 300m -i /gpfs/home/juan.xie/.aspera/connect/etc/asperaweb_id_dsa.openssh'

getSRAfile( rs1$run, sra_con, destDir=getwd(),fileType = 'sra',
               srcType = 'fasp', ascpCMD = ascpCMD )

```
And I got this error message:

```css
sh:ascp: command not found
```

It is doubted that I didn't install/configure *Aspera* correctly. After searching, I re-configure as follows:

```css
## after downloading aspera, 
cd /gpfs/home/juan.xie/software
tar -zxvf ibm-aspera-connect-3.8.3.170430-linux-g2.12-64.tar.gz
sh ibm-aspera-connect-3.8.3.170430-linux-g2.12-64.sh

# check the .aspera directory
cd /gpfs/home/juan.xie  # go to root directory
ls -a # if you could see .aspera, the installation is OK

# add environment variable
echo 'export PATH=/gpfs/home/juan.xie/.aspera/connect/bin/:$PATH' >> ~/.bashrc
source ~/.bashrc   

#密钥备份到/home/的家目录（后面会用，否则报错）
cp ~/gpfs/home/juan.xie/.aspera/connect/etc/asperaweb_id_dsa.openssh ~/

# when I used 'cp ~/.aspera/connect/etc/asperaweb_id_dsa.openssh ~/', it did not work.

# check help file
ascp --help 
```
This time it seems that the *Aspera* was installed correctly.

Then I run the R script again, it worked.

And it was fast.
