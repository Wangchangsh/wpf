---
title: VG使用
author: changshengwang
date: '2023-08-04'
slug: []
categories:
  - Bioinfor
  - Genetics
tags: []
---

# VG使用

VG是构建图形泛基因组和鉴定变异的利器。主要流程为图形基因组构建，短reads比对和变异鉴定。

## 图形基因组构建

初始文件为4个，基因组fasta和索引fai，变异VCF和索引tbi。保证文件可读可写。

```powershell
bcftools view vairant.vcf -Oz -o variant.vcf.gz
tabix -p vcf variant.vcf.gz

singularity exec -e ~/Singularity_lib/vg_1.48.sif vg autoindex --workflow giraffe -r genome.fa -v variant.vcf.gz -p genome_variant
```

## giraffe比对

因VG giraffe读取索引时会写入一些什么，多个任务同时读取时会导致速度降低成千上万倍，需在比对前将索引设置为不可写（`https://github.com/vgteam/vg/issues/3865`）。

```powershell
chmod 444 genome_variant*

singularity exec -e ~/Singularity_lib/vg_1.48.sif vg giraffe -t 8 --report-name ../02.Gam/${i}.rp -Z ../00.REF/NIP_GI.giraffe.gbz -m ../00.REF/NIP_GI.min -d ../00.REF/NIP_GI.dist -f ../01.fq/${i}_1.fq.gz -f ../01.fq/${i}_2.fq.gz -p > ../02.Gam/${i}.gam
```

## 变异鉴定

加入-a调用参考VCF的所有位点，即保留0/0

```powershell
i=$1

singularity exec -e ~/Singularity_lib/vg_1.48.sif vg pack -t 8 -x ../00.REF/NIP_GI.giraffe.gbz -g ../02.Gam/${i}.gam -Q 5 -o ../03.Pack/${i}.pack
singularity exec -e ~/Singularity_lib/vg_1.48.sif vg call ../00.REF/NIP_GI.giraffe.gbz -t 8 -k ../03.Pack/${i}.pack -a -s ${i} > ../04.Call/${i}.vcf
```