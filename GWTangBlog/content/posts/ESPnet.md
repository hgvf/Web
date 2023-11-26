+++
title = 'ESPnet 安裝記錄'
date = 2023-11-26T01:33:07+08:00
tags = ["Python", "Deep-learning", "NLP"]
category = ["Toolkit"]
+++

### 參考網站
https://espnet.github.io/espnet/installation.html (ESPNet 官網)

## 安裝 Kaldi
1. Git clone Kaldi
```shell=
$ cd <要安裝的位置>
$ git clone https://github.com/kaldi-asr/kaldi
```
2. Install tools
```shell=
$ cd <kaldi-root>/tools
$ make -j <NUM_CPU>
```
>  (在 lab 時候 <NUM_CPU> 可以用 32。)
 
3. 使用 OpenBLAS 安裝 BLAS libraries
```shell=
$ cd <kaldi-root>/tools
$ ./extras/install_openblas.sh
```
4. 檢查有沒有 tools 缺漏
```shell=
$ ./extras/check_dependencies.sh
```
> (有缺漏就按照指示裝，沒有的話會顯示 all ok)
6. Compile & install Kaldi
```shell=
$ cd <kaldi-root>/src
$ ./configure --use-cuda=no
$ make -j clean depend
$ make -j <NUM_CPU>
```