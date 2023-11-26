+++
title = 'Seiscomp'
date = 2023-11-26T02:03:54+08:00
tags = ["EEW System", "C/C++", "Linux"]
category = ["氣象局計畫"]
+++

# 基本觀念

---

- 所有 SeiscomP 的模組: picking, magnitude, estimation, … 都是透過 XML 檔案來溝通
- 每次要做一個事件都必須將 metadata 匯入資料庫

---

# 系統

---

- **fdsnws**: web service
- **scamp**: 計算 amplitude
- **scautoloc**: 自動定位
- **scautopick**: 自動 picking
- **scdb**: seiscomp 資料庫連接的模組
- **scevent**: 判斷是否成為一個 event
- **scmag**: 計算 magnitude
- **scmaster**: 管理 Seiscomp 即時資料的訊息環

---

# 基本指令 (SeiscomP)

---

- 連進 mysql server

```bash
$ mysql -u root -p
```

- 啟動 Seiscomp
    - System
        - Update Configuration: 可確認資料庫是否正常運作
        - Enable module(s): 將模組 ON
        - Start → 啟動 Seiscomp

```bash
$ scconfig
```

- 觀察 mseed 檔案內的波型
    - 右鍵往右拖移 → Zoom-in
    - 右鍵往左拖移 → Zoom-out

```bash
# 看 offline data
$ scrttv <mseed filename>

# 看 real-time data
$ scrttv
```

- 建立事件 SeiscomP資料夾
    - 裡面會拆解該事件每個測站的波型，並分類，有固定格式

```bash
$ scart -I <mseed filename>
```

- 透過 UI 介面看事件
    - 要先 scart 把波型資料匯入

```bash
# Offline 模式
$ scolv -d <database name, default: localhost> -i <event xml filename, default: events.xml>

# 只輸入 scolv: 抓 real-time data
$ scolv
```

- 將人工檢視的事件匯入資料庫
    - 把處理完的事件存成另外的 xml，並用 scdb 匯入

```bash
$ scdb -d <database name, default: localhost> -i <event xml filename, default: events.xml>
```

- 將 autopicking 或事件檔案結果轉成 xml 檔案
    - 用 scrttv 看波形後，再匯入轉好的 xml 檔案來看 autopicking 結果等等
    - 格式參考: https://www.seiscomp.de/doc/apps/sh2proc.html
    

```bash
$ sh2proc <要轉的檔案> -d <database name> > <xml filename>
```

- 排序 mseed 檔案，以 endtime 為主，並移除重複的儀器
    - 相關參數: https://www.seiscomp.de/doc/apps/scmssort.html

```bash
$  scmssort -vuE <要轉的 mseed> > <轉好的 mseed 檔案名稱>
```

- 將資料庫中的事件資料刪除 (使用上要小心)
    - 相關參數: https://www.seiscomp.de/doc/apps/scdbstrip.html

```bash
$ scdbstrip -d <name of database> --days <保留最近X天的資料>
```

- 觀察 Seiscomp 即時資料訊息環內容
    - 類似 Earthworm shared memory (RING)

```bash
$ scmm
```

- 看資料庫，除了 scolv 的指令

```bash
$ scesv
```

- 在地圖上看即時的資料

```bash
$ scmv
```

# 其他動作

---

- 線上取得事件資訊 (下載事件 xml 檔案)
    - 瀏覽器網址: 127.0.0.1:8080
    - 使用 python obspy 套件: https://docs.obspy.org/packages/obspy.clients.fdsn.html
        
        (將 Client 裡面的參數改成 “https://127.0.0.1:8080”，就可透過套件取得波型等資料)
        

- 將 metadata 匯入 Seiscomp，才可做後續分析
    - 開啟 Seiscomp
    - **Inventory** → Import (.xml) → Check inventory → Sync keys → Test sync → Sync
    - **Bindings** → 右邊 global → 下面 add profile
        
        → 設定 *detecLocid* (location) & *detecStream* (channel)
        
        → (optional) scautopick→ detecFilter (每筆 trace 的 picking procedure)
        
        → 將剛剛做好的 profile(s) 按住拉到左邊資料夾的 *TW* 代表將設定檔 apply 到那些測站
        
- 設定系統波型資料來源
    - **Modules** → System → global → recordStream

- 實作一系列 offline 測試
    - by **run_offline_seiscomp.sh**
    - 步驟: autopicking → autoloc → amp → mag → event
    - 執行 shell script
    - 執行 **scart** 完後，會將 mseed 裡面的波形，有格式的放入設定的 recordStream 路徑
    
- 實作一系列 online 測試
    - 先用 run_seisbench 生出 phase.txt
    - sh2proc 將事件資料轉成 xml 檔案
    - **scdispatch**: 將 picks.xml 資訊寫入訊息環、資料庫
    - 直接輸入 **scolv**，看有沒有在 DB 中
    
- Scolv 手動 pick
    - **1**: 進入 picking 模式
    - **R**: 鍵讓系統找出合理的 P-arrival
    - **空白鍵**: 確定此 picking 結果
    - 全部做完後，右上角紅色按鈕 “Submit” 將結果傳回 scolv，再跳回剛剛的頁面做 "Relocate"，重新定位
    - **Z, N, E**: 打開波形，按 “Z”, “N”, “E” 切換顯示方式指定哪一軸
    - **T**: 打開波形，直接顯示三軸資訊
    - 裡面的 P, S 線如果是黑色，代表 unused，沒有被拿來做定位等事情，需要在下方欄位 activate，才會變綠色
    - **F3**: 將資料庫中 metadata 有記錄的測站顯示，可以在裡面挑選要看的測站 (ex. 用測站距離震央決定)

- 更改模組參數
    - scconfig → Modules → 改 !
    - 改完記得 Update configuration → Restart
    
- 將 seisbench autopicking 結果產生的檔案轉成 xml 檔案
    - 首先要先將 mseed 波形匯入 DB
    
    ```bash
    $ scart -I <mseed file>
    ```
    
    - 參考達毅 source code
    
    ```bash
    $ sh2proc phase.txt -d localhost > picks.xml
    ```
    
    - 因為免費版的 seiscomp 在定位等模組中不會考慮 S-wave，所以轉好 xml 檔案後，還須使用 scolv 去手動把 S 波 pick 出來，再產生新的 events.xml
    
    ```bash
    $ scolv -d localhost -i picks.xml
    ```
    
    - 最後再更新 DB
    
    ```bash
    $ scdb -d localhost -i new_events.xml
    ```
    
    - 將 mseed 轉成 MAN (tank files)
    
    ```bash
    # 需要 source earthworm 環境
    $ ms2tank <path/to/mseed> >> <outfile>
    
    # 設定每個封包有幾個點
    $ ms2tank <path/to/mseed> -n <number of sample per packet> >> <outfile>
    
    # 排序時間 (否則輸出是依照測站回播而不是時間順序)
    $ remux_tbuf <path/to/tankfile> <outfile>
    ```
    
    - IRIS ringserver
        - 先去 github 下載 zip
        - **make** 編譯
        
        ```bash
        $ CFLAGS=-std=gnu99 make
        ```
        
        - 進去 scconfig 把 **seedlink** 模組 unable
        - 進去 binding 在 **seedlink** 創建 profile，下方 source 選定即時資料來源，選完後按 **+**
        - seedlink profile 記得拉到每個測站
        - 再從中設定 address, port 等等，最後再去重啟模組
        - ring.conf 內容設定
            - **RingDirectory**: ****ring server需要一個資料夾存 buffer
            - **WriteIP**: 寫波形進 ring server 的 IP
            - d
        - 單獨執行 ringserver
        
        ```bash
        $ ringserver ring.conf
        ```
        
        - 觀察 ringserver
        
        ```bash
        # 觀察 ringserver 有無啟動
        $ slinktool -I <ringserver address:port> (ex. 127.0.0.1:18001)
        
        # 查看 ringserver 監聽的測站列表
        $ slinktool -L <ringserver address:port> (ex. 127.0.0.1:18001)
        
        # 看 ringserver 波形資訊有無進入
        $ slinktool -Q <ringserver addredd:port>
        ```
        
    
    - Note
        - **Travel time** 通常是左下往右上，線性的，如果有偏差太大的可能就不適合加入定位測站
        - 手動增加 **Multiple event**: 在 picker 介面中按右鍵 → Create Artifical Origin，系統就會從該點往就算理論 P/S 到時