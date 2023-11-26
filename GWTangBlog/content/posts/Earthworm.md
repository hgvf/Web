+++
title = 'Earthworm'
date = 2023-11-26T01:30:46+08:00
tags = ["EEW System", "C/C++", "Linux"]
category = ["氣象局計畫"]
+++

## Ref
* [研習教學影片](https://www.youtube.com/playlist?list=PL1JK9KOKer9dQi_BCGuXaWaStkoTGZLEP) (感謝子毅提供！)

## 基本觀念
1. 參數檔案存在 /params (ex. earthworm.d)。
2. 執行檔存在 /bin。
3. **startstop.d**: 告訴 earthworm 目前有多少 ring, module...。
4. **earthworm.d**: 定義 ring 名稱、代號，以及 modules, message type...。
5. **earthworm_global.d**: 定義機構代號、

## 指令
0. **啟動 earthworm 系統**
```shell=
$ source ew_linux_xxx.bash

# 啟動 earthworm system
$ startstop

# 如果有新增的模組，在不關掉 earthworm 系統之下匯入模組
$ recon
```

1. **sniffwave**: 主要查看某個 RING 收到的資料
```shell=
$ sniffwave XXX_RING

# only station header
# sniffwave <RING_NAME> <Station> <CNL> <TW?> -- n
$ sniffwave WAVE_RING TWGB BHE TW -- n

# only station waveforms
# sniffwave <RING_NAME> <Station> <CNL> <TW?> -- y
$ sniffwave WAVE_RING TWGB BHE TW -- y
```

2. **findwave**: 查看某個 RING 有無收到資料，並寫到某個檔案之中
```shell=
$ findwave XXX_RING <index_name> <days> <file_name> <output types>

# index_name:   將來建立 wave server 時，每個 channel 檔案 index size，
#               日後在提取資料時就依照 index 來找資料，通常 40~60
# days:         每個 channel 檔案最多暫存幾天
# output_types: w -> for wave server output
#               d -> 其她資訊
#
# ex: findwave WAVE_RING 1 1 out.txt w
# 以下檔案只適用 wave server
```
![](https://i.imgur.com/9OAtTr3.png)

    - Tank:          資料型態
    - SCNL names:    測站名 (搜尋裡面的波形，通常測站都有 ENZ 三軸，所以 SCNL 通常要是三的倍數)
    - BH(E|N|Z):     Channel 
                    (E: 東西向, N: 南北向, Z: 垂直向, B: 取樣率不到 100, H: 寬頻資料)
    - TW:　          Network name
    - TW 後面的 '--': Location name
    - size:          封包長度，iris 資料大小都不同
    - ModuleID:      用哪個 module 接收資料
    - File size:     一天有多少 MB (取決於這個 ring 開的記憶體空間)
    - Index size:    每個 channel 的 index size (單位為 byte)
    - File name:     存這個資料的路徑 (目前為暫存檔，存在 memory)
    
3. **getmenu**: 可以看目前這個網路 server 位置的封包狀況
```shell=
$ getmenu <IP_address>:<Port number>
```

## Modules
1. **slink2ew**: 接收某些測站的 seed link 波型資料。
*     Stream:  指定要那些測站的資料進來 (TW_*: 所有 TW 測站)。
*     Selector: 篩選進來資料的條件 (channel, instrument...)。

2. **wave_serverV**: 建立 wave server 的參數檔。
*     裡面設定的 ring name 要與收到波型的 ring 相同 (ex. 與 slink 相同 ring)。
*     要另外新增資料夾放 tank files (在 .d 檔設定 TankStructFile，有兩個!)。
*     裡面要貼上用 findwave 製作出來的格式。
*     InputQueueLen: 設為前面 tank 格式數量的兩倍以上。
```shell=
# 檢查 wave_server 有沒有成功運作
$ getmenu 127.0.0.1:16022 
```


3. **waveman2disk**: 擷取 wave server 資料，然後改成任意格式，或是切片段波型 (不需掛載在 startstop。)
*     InputMethod: 可以改成 interactive。
*     SaveSNL: 限制要存的檔案類型 (用 station, channel, network, location 選擇)。
*     StartTime: 要確定時間區段正確性 (用 getmenu 確認波型時間區段)。
*     Duration: 切分的時間長度 (單位:秒)。
*     WaveServer: 設定 ip, port number。
*     輸出格式: 選擇波型輸出格式 (只能選一種!!!)。

```shell=
# 執行程式
$ waveman2disk waveman2disk.d
```

4. **tankplayer**: 重新跑一次地震波形資料並接收。(虛擬的 real-time)
*     RINGNAME: 可以改成別的以免混淆。
*     WaveFile: 改成要存放 tank files 的路徑。
```shell=
# 先將蒐集到的資料轉成 tank file (ex. SAC -> tb)
$ sac2tb <file_name> >> <tb file_name>

# 要照時間排序才能一次輸出三個 channels，否則一次只會播放一個 channel
$ remux_tbuf <tb file_name> r <sorted tb file_name>

# 將氣象局的 Afile 轉成 tank file
# 在 window 底下執行
$ CWB24toTank <AFile>
```

5. **pick_ew**: 自動挑出 P-wave, S-wave (使用 STA/LTA rule)，並放到指定的 ring，必須連帶更改一些測站參數檔等等。
*     StaFile: 要自動作 picking 的測站清單。
*     InRing, OutRing: 波型進出的 ring，要設定好。
*     依照模組 copy 對應所需 (ex. eqproc...)。
*     根據要 picking 的測站，更改 pick_ew.memphis 裡面的測站資料。

6. **binder_ew**: 把 pick_ew 的東西包起來做初步定位，藉此分隔不同測站的資料。

## 相關套件
1. **Swarm**: 用來看波形圖
```shell=
$ cd /mnt/nas7/weiwei/Earthworm/earthworm_7.10/src/display/swarm-2.8.11

# 啟動軟體
$ sh swarm.sh

# 在 GUI 左邊欄位按: New data source -> 選 earthworm wave server
    -> 輸入對應的 ip, port...
    
# 左下角的 wave to monitor -> 可以看到 wave server 裡面所有測站的即時波型
```

2. **Hypoinverse**: 收到 pick_ew 的資料來定位地震，並計算規模，然後寫進 EW shared memory。

3. **CWB_EEW**: 氣象局目前的預警系統。

```shell=
$ cd /mnt/nas7/weiwei/Earthworm/earthworm_7.10/src/eew/cwb
```