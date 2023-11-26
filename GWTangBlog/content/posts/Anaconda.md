+++
title = 'Anaconda'
date = 2023-11-26T01:28:47+08:00
tags = ["Python", "Virtual Environment"]
category = ["程式"]
+++

## 安裝
參考網站: https://ithelp.ithome.com.tw/articles/10237621

## 指令

1. 看已建立的所有環境
```shell=
conda env list
```

2. 建立環境
```shell=
conda create --name <env_name> python=<python version>
```

3. 啟動環境
```shell=
conda activate <env_name>
```

4. 安裝套件
```shell=
conda install <package>
```

5. 退出環境
```shell=
conda deactivate
```

6. 刪除環境
```shell=
conda env remove --name <env_name>
```

7. 看已安裝套件
```shell=
conda list
```