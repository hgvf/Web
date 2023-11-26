+++
title = 'SpeechBrain'
date = 2023-11-26T01:33:25+08:00
tags = ["Python", "Deep-learning", "NLP"]
category = ["Toolkit"]
+++

### 參考網站
- https://speechbrain.github.io (SpeechBrain 官網)
- https://github.com/speechbrain/speechbrain (SpeechBrain github)
- https://speechbrain.readthedocs.io/en/latest/index.html (Speechbrain docs)

## YAML file
speechbrain 中會使用到 ".yaml" 類型檔案來定義不同的參數，例如訓練的 hparams、模型架構等等。

#### tags:
1. 新增一個 python object: **new** (ex. LogSoftmax)
```yaml=
log_softmax: !new: torch.nn.LogSoftmax
    dim: -1
```
2. 新增一個 function object: **name** (ex. seq_cost)
```yaml=
seq_cost: !name: speechbrain.nnet.losses.kldiv_loss
    label_smoothing: <label_smoothing>
```
3. 取用 yaml file 其他變數的值: **ref** (ex. SGD)
```yaml=
SGD: !name: torch.optim.SGD
    lr: !ref <lr_sgd>
```
4. 複製一份 variable or object，且不會有相同 reference: **copy**
```yaml=
apple: !copy <foo>
```
5. Include 其他的 yaml file: **include**
6. 讀取 & 執行其他的 python function: **apply** (ex. 設定 seed)
```yaml=
__set_seed: !apply: torch.manual_seed [!ref <seed>]
```

## Brain Class
1. 宣告方式: 
```python=
class ASR(sb.Brain):
```
2. 每個 sb.Brain object 都必須包含兩個 functions: 
- **compute_forward()**: 定義如何計算以及計算什麼值。
```python=
def compute_forward(self, batch, stage):
```
- **compute_objectives()**: 定義如何計算 loss。
```python=
def compute_objectives(self, predictions, batch, stage):
```
3. 一般的 sb.Brain object 中，初始化 init() 包含以下參數：
- **modules**: 以 dictionary 存在，可以存放 model 架構等等。
- **opt_class**: 存放 optimizer 相關參數。
- **hparams**: 存放 hyperparameters。
- **run_opts**: 用來控制 fit() 運作細節。
- **checkpointer**: 存放 checkpointer，若這個欄位不為 None，則會自動存 optimizer 狀態、讀取最近的 checkpointer & resume、預設 15 mins 存一次存一次、以及在 evaluation 時挑選最佳 checkpoint。
```python=
def __init__(
self, 
modules=hparams["modules"],
opt_class=hparams["Adam"],
hparams=hparams,
run_opts=run_opts,
checkpointer=hparams["checkpointer"],
):
```

## fit() structure
負責主要 training 的流程等等。

1. 初始化 init() 包含以下參數：
- **epoch_counter**: 在 EpochCounter class 中，可以控制 epoch。
- **train_set** & **valid_set**: 接收 Torch Dataset or DataLoader，如果沒有傳 dataloader，會在後面的 make_dataloader() 製作。
- **progressbar**: 顯示 tqdm 進度條。
- **train_loader_kwargs** & **valid_loader_kwargs**: 把製作 dataloader 相關參數傳給 make_dataloader()。
```python=
def fit(
self, 
epoch_counter=hparams["epoch_counter"],
train_set=train_data,
valid_set=valid_data,
progressbar=None,
train_loader_kwargs=hparams["train_dataloader_opts"],
valid_loader_kwargs=hparams["valid_dataloader_opts"],
):
```
2. 大致架構如下：
```python=
def fit(self, ...):
    self.make_dataloader()
    # fit() 開始
    self.on_fit_start()
    
    for epoch in epoch_counter:
        # Training stage 開始
        self.on_stage_start(Stage.TRAIN)
        
        for batch in train_set:
            # 裡面執行 compute_forward() & compute_objectives()
            self.fit_batch(batch)
        
        # Training stage 結束
        self.on_stage_end(Stage.TRAIN)
        # Valid stage 開始
        self.on_stage_start(Stage.VALID)
        
        for batch in valid_set:
            # 裡面執行 compute_forward() & compute_objectives()
            self.evaluate_batch(batch)
            
        # Valid stage 結束    
        self.on_stage_end(Stage.VALID)
```
- **make_dataloader()**: 可以將 train_set & valid_set 變成適合 iterate 的格式。
- **on_fit_start()**: 在訓練前有一些設定，例如初始化 optimizer、resume latest checkpoint...。
- **on_stage_start()**: 開始 iterate data，也可以在這建立一些 container 來搜集統計數據。
- **fit_batch()**: 會呼叫 compute_forword() & compute_objectives()，記錄下平均 loss、checkpoint 等等。
- **on_stage_end()**: 清除 stage 使用到的資料，也可以輸出一些 stage 的統計數據。
- validation 版本 **on_stage_end()**: 同上，也會更新 learning rate、紀錄 checkpoint、epoch 數據。

## DataIO
主要由三個 blocks 組成：
1. **DynamicItemDataset**: 會從 csv, json 檔案中取得資料，創建一個類似 pytorch dataset 的物件。
2. **Dynamic Items Pipelines (DIPs)**: 由使用者來定義，可以定義如何操作 dataset、或是 audio file augmentation。
3. **Categorical Encoder**: 可以將 encoding label 轉換成 discrete 數值，而 label 跟 unique index 的對應關係會由兩個 dictionary 維護：lab2ind & ind2lab，例如 speakerID: 1223 -> [5]。
```python=
from speechbrain.dataio.dataset import DynamicItemDataset

# .json file
dataset = DynamicItemDataset.from_json("data.json")

# .csv file
dataset = DynamicItemDataset.from_csv("data.csv")
```
這邊的 dataset 無法直接被使用，而是需要 user 指定 output 欄位，就可以使用 DIP 來操作。
```python=
@speechbrain.utils.data_pipeline.takes("file_path")
@speechbrain.utils.data_pipeline.provides("signal")
def audio_pipeline(file_path):
      sig = speechbrain.dataio.dataio.read_audio(file_path)
      return sig
```
指定接收 "file_path"，然後提供 "signal" dataset，也就是帶有 audio 的 tensor。

```python=
dataset.add_dynamic_item(audio_pipeline)
dataset.set_output_keys(["signal", "file_path"],)

# add_dynamic_item 另外寫法
dataset.add_dynamic_item(speechbrain.dataio.dataio.read_audio, takes="file_path", provides="signal")
```
指定完 dataset 輸出欄位後，必須轉換成為 **DynamicItemDataset** 形式，再使用 **set_output_keys** 來指定輸出什麼。
```python=
dataset[0]
```
{'file_path': './LibriSpeech/dev-clean-2/5338/284437/5338-284437-0014.flac',
 'signal': tensor([ 2.1362e-04,  6.1035e-05, -3.0518e-05,  ...,  3.0518e-04,
          3.9673e-04,  3.9673e-04])}
          
這時的 dataset 就會是根據 output keys 來輸出結果。

```python=
from speechbrain.dataio.encoder import CategoricalEncoder

# 宣告一個 Categorical encoder
spk_id_encoder = CateforicalEncoder()

# 傳統沒有 encode 的狀況
speechbrain.dataio.dataset.set_output_keys([dataset], ["spk_id"],)

dataset[0]    # result: {'spk_id': '5338'}
```
以往直接輸出數值，可能會遇到這些 id 不是唯一值的狀況，就會造成問題，因此需要借助 categorical encoder 來 encode 出 unique id。
```python=
# Iterate 整個 dataset 的 spkID 欄位，
# 來更新內部的 lab2ind & ind2lab
spk_id_encoder.update_from_didataset(dataset, "spkID")
```
盡量在前一步先做 **set_output_keys()**，避免計算上耗費資源。
而且只設定單一 key 也會使 encoder 有比較好的 fit 成果。

```python=
# 看有多少個 distinct value 
len(spk_id_encoder)

# label -> integer encoding
spk_id_encoder.lab2ind    

# integer encoding -> label
spk_id_encoder.ind2lab
```
當做完了 categorical encoding 之後，就可以套用在 data pipeline 之中。
```python=
@speechbrain.utils.data_pipeline.takes("spkID")
@speechbrain.utils.data_pipeline.provides("spkid_encoded")
def spk_id_encoding(spkid):
      return torch.LongTensor([spk_id_encoder.encode_label(spkid)]) 
      
speechbrain.dataio.dataset.add_dynamic_item(
        [dataset], spk_id_encoding) 
        
speechbrain.dataio.dataset.set_output_keys(
        [dataset], ["spkid_encoded"],)

dataset[100]
```
Output: {'spkid_encoded': tensor([3])}