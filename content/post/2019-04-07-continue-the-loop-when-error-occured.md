---
title: continue the loop when error occured
author: ''
date: '2019-04-07'
slug: continue-the-loop-even-when-error-occured
categories:
  - R
tags: []
---

I was running an R script to extract the SNPs profiles for ~1050 cell lines from COSMIC data:

```
file <- "/gpfs/scratch/juan.xie/TEMP/CosmicCLP_MutantExport.tsv.gz"
cell_lines <- list_cosmic(file)
COSMIC <-foreach(i=1:length(cell_lines),.packages=c('seqCAT'))%dopar%{
	temp <-read_cosmic(file,cell_lines[i])
}

```
Yet the execution halted due to the error ` task 582 failed -"the sample NB(TU)110 is either not present in the data or has no listed SNVs.`

It is found that some cell lines do not have any SNVs in the `CosmicCLP_MutantExport.tsv.gz`. I wanted to ignore these empty cell lines and continue. Googling suggest to use *try* or *tryCatch*. I tried both, and found that *try* worked in my case. However, the results also include the error message for empty cell lines. Couldn't figure out a better way, I creat another list, whose element is empty if the cell line has no SNVs in COSMIC. Then I just keep the non-empty elements in the new list.

The modified code is as follows:
```
file <- "/gpfs/scratch/juan.xie/TEMP/CosmicCLP_MutantExport.tsv.gz"
                   
cell_lines <- list_cosmic(file)

#use try()to continue with errors
COSMIC <-foreach(i=1:length(cell_lines),.packages=c('seqCAT'))%dopar%{
	temp <-try(read_cosmic(file,cell_lines[i]))
}

## some cell lines may not have SNPs, thus need to remove them
tt <-list()
for (i in 1:length(COSMIC)){
	if(length(COSMIC[[i]])>1){
		tt[[i]] <-COSMIC[[i]]
	}
}

COSMIC2 = tt[length(tt)!=0] # the clean COSMIC profiles for non-empty cell lines

```
