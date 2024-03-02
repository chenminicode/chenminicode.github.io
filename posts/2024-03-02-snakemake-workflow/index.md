---
title: 使用Github的Snakemake流程
author: metseq
date: "2024-03-02"
categories: [Bioinformatics]
toc: true
number-sections: false
---

# 概述
GitHub有一个[snakemake流程的分类](https://snakemake.github.io/snakemake-workflow-catalog/)，里面有很多生信分析流程，rna-seq-star-deseq2，dna-seq-gatk-variant-calling。

我测试了[rna-seq-star-deseq2](https://github.com/snakemake-workflows/rna-seq-star-deseq2)这个流程。

每个流程都有使用说明，如果都看说明可以跑通，那就好了🥹

下面参考[rna-seq-star-deseq2的说明](https://snakemake.github.io/snakemake-workflow-catalog/?usage=snakemake-workflows%2Frna-seq-star-deseq2)，一步一步做.


# Step 1 安装snamake和snakedeploy

使用mamba安装snakeme和snakedeploy，如果没有mamba，自行搜索安装吧.

```shell
$ mamba create -c conda-forge -c bioconda --name snakemake snakemake snakedeploy

$ conda activate snakemake
```

# Step 2 流程部署

```shell
$ mkdir proj_240302_test_snakemake

$ cd proj_240302_test_snakemake

$ snakedeploy deploy-workflow https://github.com/snakemake-workflows/rna-seq-star-deseq2 . --tag v2.1.0
```

运行后的文件结构：

```shell
$ tree
.
├── config
│   ├── README.md
│   ├── config.yaml
│   ├── samples.tsv
│   └── units.tsv
└── workflow
    └── Snakefile
```

其实就下了config和workflow两个文件夹，里面分别是配置文件和流程。workflow里面只有一个Snakefile，Snakefile里把github上
rna-seq-star-deseq2的流程作为模块导入了。

# Step 3 配置流程

流程配置是最麻烦的，我下载了[仓库](https://github.com/snakemake-workflows/rna-seq-star-deseq2.git)自带的测试数据。

测试数据在`.test`文件夹下面，结构如下，应该很清楚了，`config_basic`就是简单版的配置文件，`config_complex`文件夹是复杂版的配置文件，`ngs-test-data`就是原始数据了。

```shell
$ tree .test
.
├── config_basic
│   ├── config.yaml
│   ├── samples.tsv
│   └── units.tsv
├── config_complex
│   ├── config.yaml
│   ├── samples.tsv
│   └── units.tsv
└── ngs-test-data
    └── reads
        ├── a.chr21.1.fq
        ├── a.chr21.2.fq
        ├── a.scerevisiae.1.fq
        ├── a.scerevisiae.2.fq
        ├── b.chr21.1.fq
        ├── b.chr21.2.fq
        ├── b.scerevisiae.1.fq
        ├── b.scerevisiae.2.fq
        ├── c.scerevisiae.1.fq
        └── c.scerevisiae.2.fq

5 directories, 16 files
```

删掉`Step 2 流程部署`下载的config文件，把`.test/config_basic`中的配置文件都拷贝`proj_240302_test_snakemake/config`文件夹下，然后把`.test/ngs-test-data`文件夹拷贝到`proj_240302_test_snakemake`文件夹。

修改`proj_240302_test_snakemake/config/config.yaml`文件的samples和unit配置路径：

```shell
samples: config/samples.tsv
units: config/units.tsv
```

配置有很多细节，比如原始数据的接头序列，差异表达分组信息，是否是链特异文库等等。

请参考配置文件的说明和流程自带的配置说明。当然`.test`文件夹中的配置文件格式值得参考。

# Step 4 运行流程

终于可以运行流程了：

```shell
$ snakemake --cores all --use-conda
```

果然报错了😭：

```shell
NotImplementedError: Remote providers have been replaced by Snakemake storage plugins. Please use the corresponding storage plugin instead (snakemake-storage-plugin-*).
```

Google一下，rna-seq-star-deseq2里有一个[issues](https://github.com/snakemake-workflows/rna-seq-star-deseq2/issues/71)就是这个问题。

原因是snakemake流程最近（2024年3月3日）日更新到版本8了，版本8引入了plugin功能，rna-seq-star-deseq2流程还不支持。issues里也给出了解决办法，删除已经装好的snakemake的conda环境，然后安装7.32.4版本的snakemake。

```shell
$ mamba deactivate

$ mamba remove -n snakemake --all

$ mamba create -c conda-forge -c bioconda -n snakemake7 snakemake=7.32.4 

$ mamba activate snakemake7
```

重新配置好环境后，再次运行流程：

```shell
$ snakemake --cores all --use-conda
```

流程会先下载依赖的环境，参考基因组（我用了大约20分钟），然后运行里面定义的步骤。

结果文件在`results`文件夹下：

```shell
$ ll -rth results/
total 120
drwxr-xr-x  14 chenmin  staff   448B Mar  3 00:17 trimmed
drwxr-xr-x   6 chenmin  staff   192B Mar  3 00:20 star
drwxr-xr-x   4 chenmin  staff   128B Mar  3 00:21 qc
drwxr-xr-x   4 chenmin  staff   128B Mar  3 00:22 counts
-rw-r--r--@  1 chenmin  staff    59K Mar  3 00:22 pca.condition.svg
drwxr-xr-x   5 chenmin  staff   160B Mar  3 00:22 deseq2
drwxr-xr-x   5 chenmin  staff   160B Mar  3 00:23 diffexp
```

# Step 5 生成报告

```shell
$ snakemake --report report.zip
```

解压report.zip，打开report.html，网页左侧有4个选项，Workflow是流程运行的步骤，说明；Statistics统计了每一步运行的时间；About是流程用到软件的使用权限说明；Results是最关心的结果了。

测试完成🎉