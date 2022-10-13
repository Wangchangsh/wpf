---
title: 下载测序数据
author: wangchangsheng
date: '2022-03-16'
slug: []
categories:
  - Bioinfor
tags: []
---

在新课题和新项目开展时，需要重复他人的实验以确定可靠的流程。一般在文献最后都会附上项目编号**PRJNA**。SRA数据库的层次结构是SRP(项目)-SRS(样本)-SRX(数据产生)-SRR(数据本身)。数据库的层级是PRJNA-SAMN。

以高粱比较基因组数据集PRJNA513297为例。

### NCBI SRA

目前NCBI正在将数据传输至亚马逊云AWS和谷歌云GS，美国境内服务器免费，其他的服务器收费，用户自己掏钱，由于正在迁移，NCBI也不保证数据的完整性。虽然也提供了亚马逊云的免费链接，但是国内实测速度奇慢无比。使用Aspera找不到SRA的下载链接，一般使用NCBI的工具prefetch或fastq-dump。

先搜索项目号，获取项目详细信息。

```shell
module load Singularity/3.1.1
# 下载数据,若数据量大于20G需加-X参数
awk -F ',' '{print $1}' sample.list|grep SRR|xargs -n 1 singularity exec -e ~/Singularity_lib/download.sif prefetch 
## fastq-dump
singularity exec -e ~/Singularity_lib/download.sif fastq-dump -A SRRXXX
singularity exec -e ~/Singularity_lib/download.sif fastqer-dump -e threads SRRXXX
```

### EMBL ENA

美国的NCBI的SRA与欧洲的EBI-EMBL以及日本的DDBJ数据库共享数据，我们可以使用Aspera在EBA数据中直接下载fastq文件。

先搜索项目号，点击PRJNA数据集，下载TSV的Download report(无法复制链接在linux下载)。

```shell
# 下载数据集，此处sample.list为ENA的TSV的Download report
grep -v study_accession sample.list|awk -F '[;\t]' '{print$7"\t"$8}'|while read i j;do singularity exec -e ~/Singularity_lib/download.sif ascp -i /usr/asperaweb_id_dsa.openssh -v -k 1 -T -l 200m -P 33001 era-fasp@fasp.sra.ebi.ac.uk:${i##*uk} .;singularity exec -e ~/Singularity_lib/download.sif ascp -i /usr/asperaweb_id_dsa.openssh -v -k 1 -T -l 200m -P 33001 era-fasp@fasp.sra.ebi.ac.uk:${j##*uk} .;done
```

```shell
ascp [options] ssh link dir
options:
-T 不进行加密，不添加此参数可能无法下载
-i 输入私钥 
--host ftp的host名，NCBI为ftp-private.ncbi.nlm.nih.gov;EBI为fasp.sra.ebi.ac.uk
--user 用户名，NCBI为anonftp，EBI为era-fasp
--mode 选择模式，上传为send，下载为recv
-l 设置最大传输速度
-v 详细模式，能及时查看下载进度
-k 断点续传，一般设置为1
-P 提供SSH port，端口一般是33001
```

Aspera提供--file-list，支持批量下载多个文件

filelist格式

```shell
/vol1/fastq/SRR126/063/SRR12628363/SRR12628363_1.fastq.gz
/vol1/fastq/SRR126/063/SRR12628363/SRR12628363_2.fastq.gz
```

```shell
singularity exec -e ~/Singularity_lib/download.sif ascp -i /usr/asperaweb_id_dsa.openssh -T -k 1 -v -l 200m -P 33001 --mode recv --host fasp.sra.ebi.ac.uk --user era-fasp --file-list filelist .
```

### BIGD GSA

GSA是2015年底，中科院北京基因组研究所生命与健康大数据中心开发的原始组学数据归档库。数据模型和数据格式遵照INSDC标准，在功能上等同于NCBI的SRA，EBI的ENA和DDBJ的DRA。目前没有特定的工具，使用wget下载FTP数据。

以GSA000167为例，先搜索项目号，点击RUN，点入任一样本，点击FTP下载链接。

```shell
wget -c ftp://download.big.ac.cn/gsa/CRA000167/CRR057355/CRR057355_f1.fastq.gz
##批量下载
wget -c -r -np -k -L -p  ftp://download.big.ac.cn/gsa/CRA000167/
```

### CNSA

国家基因库序列归档系统（CNSA）是一个归档全球组学数据的系统，致力于组学数据的存储、管理和共享，促进组学数据的再利用，推动生命科学的发展（有GSA还不够？）。目前支持aspera下载。

以CNP0000049为例，搜索点击go to FTP，复制FTP地址http://ftp.cngb.org/pub/CNSA/data1/CNP0000049。

```shell
singularity exec -e ~/Singularity_lib/download.sif ascp -i /usr/asperaweb_id_dsa.openssh -P 33001 -T -k 1 -l 100m aspera_download@183.239.175.39:/pub/CNSA/data1/CNP0000049/ .
```

### recheck

第一轮数据下载完后，由于网络原因不可避免会造成少数样品下载失败，需要找出这些下载失败的样品并重新下载。

```shell
ls *aspx|while read i;do rm ${i/.aspx/}*;done
grep -v study_accession smple.list |awk -F '[;\t]' '{print$7"\t"$8}'|while read i j
do
if [ ! -f "${i##*/}" ]
then
    echo "${i##*/} no found"
    singularity exec -e ~/Singularity_lib/download.sif ascp -i /usr/asperaweb_id_dsa.openssh -v -k 1 -T -l 200m -P 33001 era-fasp@fasp.sra.ebi.ac.uk:${i##*uk} .
fi
if [ ! -f "${j##*/}" ]
then 
    echo "${j##*/} no found"
    singularity exec -e ~/Singularity_lib/download.sif ascp -i /usr/asperaweb_id_dsa.openssh -v -k 1 -T -l 200m -P 33001 era-fasp@fasp.sra.ebi.ac.uk:${j##*uk} .
fi
done
```

### md5check

检查md5码，防止传输错误。

```shell
grep -f <(ls *.gz) filereport_read_run_PRJEB6180_tsv.txt  |awk -F '\t' '{print $5"\t"$6}'|sed 's/;/\t/g;s/\S\+\///g'|awk '{print $1"\t"$3"\n"$2"\t"$4}' > md5.sum
md5sum -c md5.sum > md5check.log
```
