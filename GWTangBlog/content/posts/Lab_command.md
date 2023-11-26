+++
title = 'Lab_command'
date = 2023-11-26T02:04:35+08:00
tags = ["Linux"]
category = ["指令大軍"]
+++

### 查看 GPU 狀態
```shell=
$ nvidia-smi

# 逐秒檢查 (ex. 每秒刷新一次)
$ watch -n 1 nvidia-smi
```

### 複製 (ex. dataset)
```shell=
$ cp -ar <dataset_path> <target_path>
```

### 用 pid 看是誰的 process
```shell=
$ ps -o user= -p <pid>
```

### 用 pid 看對應的 command
```shell=
$ ps -p <pid> -o args
```

### 看到底誰在用 SWAP
```shell=
$ (echo "COMM PID SWAP"; for file in /proc/*/status ; do awk '/^Pid|VmSwap|Name/{printf $2 " " $3}END{ print ""}' $file; done | grep kB | grep -wv "0 kB" | sort -k 3 -n -r) | column -t
```
### 用 command line 載 IEEE papers
```shell=
$ wget "http://ieeexplore.ieee.org/stampPDF/getPDF.jsp?tp=&isnumber=&arnumber=<numberID>" -O paper.pdf

```

### 掛載硬碟 SOP
```shell=
$ sudo fdisk -l
$ sudo mount <drive point> <mount point>
```


### 看磁碟使用程度 (-h: 以 GB 為單位)
```shell=
# 看所有空間的使用程度
$ df -h

# 看當前目錄大小
$ du -h

# 排序
$ du -h | sort -h
```

### 找檔案路徑 (只會從 $PATH 中搜尋)
```shell=
$ where <search_target>
$ which <search_target>
```
which 應該也是這樣用吧?
 
### 開啟 jupyter notebook
```shell=
$ jupyter notebook --no-browser --port=5000 --NotebookApp.token='' --NotebookApp.password=''

$ ssh -L 5000:localhost:5000 weiwei@140.118.127.90
```

### 在 juputer 安裝新的 kernel (pyenv)
```shell=
$ python -m ipykernel install --user --name <pyenv enviro_name> --display-name <name of kernel>

再去存放 kernel.json 那邊，改掉裡面的路徑，改成 env path

要用的環境必須要先安裝好 ipykernel
```

### Terminal 下載 google drive 檔案
```shell=
$ gdown https://drive.google.com/uc?id=<FileID>
```

### 在電腦 or Server 上架設 HTTP server
```shell=
$ python -m http.server <port>

# 在本機端 IP:port 就有了
```

### 啟動 NLPLAB website database
```shell=
$ cd project/nlplab/api

$ sudo bin/docker-compose up
```

### 更改 OJ 或 Overleaf 憑證
* Online Judge
```shell=
$ cd projects/OnlineJudgeDeploy/data/backend/ssl/

$ sudo cp /etc/letsencrypt/live/nlp.csie.ntust.edu.tw-0001/fullchain.pem server.crt 
$ sudo cp /etc/letsencrypt/live/nlp.csie.ntust.edu.tw-0001/privkey.pem server.key 

$ cd ../../..

$ sudo docker-compose down
$ sudo docker-compose up -d
```

* Overleaf
```shell=
$ cd ~/projects/overleaf/config/nginx/certs

$ sudo cp /etc/letsencrypt/live/nlp.csie.ntust.edu.tw-0001/fullchain.pem .
$ sudo cp /etc/letsencrypt/live/nlp.csie.ntust.edu.tw-0001/privkey.pem .

$ cd ../../../

$ sudo bin/docker-compose down
$ sudo bin/docker-compose up -d
```