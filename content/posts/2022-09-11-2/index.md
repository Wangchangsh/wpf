---
title: SNPable的使用
author: wangchangsheng
date: '2022-09-11'
slug: []
categories: 
  - Genomics
tags: 
  - SNPable
---

## 前言

植物基因组普遍存在全基因组加倍融合的现象。在高通量测序read回帖时，常出现多重比对的现象，最终会导致不准确的变异鉴定。SNPable可以鉴定出基因组上多重比对的变异位点。

## SNPable使用

### 参考基因组的准备

-   BWA索引参考基因组
-   模拟35bp的单端测序文件
-   回帖
-   生成mask参考基因组

#### BWA索引参考基因组

```shell
bwa index ref.fa
samtools faidx ref.fa
```
#### 模拟35bp的单端测序文件

```shell
splitfa ref.fa 35|split -l 20000000
```

#### 回帖

```shell
bwa aln -R 1000000 -O 3 -E 3 ref.fa xaa > xaa.sai
bwa samse ref.fa xaa.sai xaa > xaa.sam
```
#### 生成mask参考基因组

```shell
gen_raw_mask.pl *sam > rawMask_35.fa
gen_mask -l 35 -r 0.5 rawMask_35.fa > mask_35_50.fa  
```
### 参考基因组的应用

```shell
apply_mask_s mask_35_50.fa ref.fa > ref.mask.fa  
apply_mask_l mask_35_50.fa in.pos > out.pos  
```