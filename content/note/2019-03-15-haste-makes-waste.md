---
title: Haste makes waste
author: ''
date: '2019-03-15'
slug: haste-makes-waste
categories:
  - Bioinformatics
tags:
  - GATK4
---

I have been working on a cell line authentication project based on GATK tools since Feb. Thanks to the abundant tutorials&posts on the web, it just took around less than two weeks to make the pipeline run. Yet in  the following weeks, I kept discovering mistakes in my pipeline.

反思一下：之前搭这个pipeline的时候对什么是SNP/SNV一无所知。只是把别人分享的代码照葫芦画瓢贴到自己的script里，并没有好好的，认真地看GATK的document，知其然不知其所以然。前前后后不知发现多少错误。所幸现在发现的错误基本在后面variant filtering，改正起来的成本没那么大。但教训是深刻的。
以后做东西还是要弄明白为什么，别人的代码要仔细想想每一行都是在做什么，符不符合我自己的情况。不要一味求快。

今天发现的错误：GATK recommend filtering based on Fisher Strand values (FS > 30.0) and Qual By Depth values (QD < 2.0).
GATK3 的example code：

```css
java -jar GenomeAnalysisTK.jar -T VariantFiltration -R hg_19.fasta -V input.vcf -window 35 -cluster 3 -filterName FS -filter "FS > 30.0" -filterName QD -filter "QD < 2.0" -o output.vcf 
```

我的code：
```
time gatk VariantFiltration \
		-R $genomeFastaFiles \
        -V ${ID}.SNP.vcf \
        -window 35 \
        -cluster 3 \
        --filter-expression "FS >30.0 || QD <2.0" \
        --filter-name "my_filter" \
        -O ${ID}_filtered.vcf
```

怀疑这个有问题是因为发现一些.err文件有有说`invalid expression QD <2.0 `
今天认真看了GATK 的[帖子](https://software.broadinstitute.org/gatk/documentation/article.php?id=1255), 里面尤其提到'To be on the safe, do not use compound expressions with the logical "OR" || as a missing annotation will negate the entire expression.'

更正后的code：

```
gatk VariantFiltration \
		-R $genomeFastaFiles \
        -V ${ID}.SNP.vcf \
        -window 35 \
        -cluster 3 \
        --filter-expression "FS >30.0" \
        --filter-name "FS" \
		    --filter-expression "QD <2.0" \
		    --filter-name "QD" \
        -O ${ID}_filtered.vcf
```
