+++
title = 'Pyenv'
date = 2023-11-26T01:26:56+08:00
tags = ["Python", "Virtual Environment"]
category = ["程式"]
+++

### 參考網站
https://github.com/pyenv/pyenv#installation
https://github.com/pyenv/pyenv-installer

## 相關檔案
- Shims
    - 安裝之後會被加入到 PATH 當中。
    - 主要用來執行 user 呼叫的 python 相關指令，例如 pip, python...。
    - 指令的參數也都會被傳給 pyenv 執行。
- Versions
    - 放置所有下載的 python 版本。
- plugins
    - 存放 pyenv 相關插件，例如 pyenv-virtualenv...。
- pyenv-virtualenv
    - 用來管理 virtualenv 創建的虛擬環境等等。

## 如何選擇 python 版本 (照以下順序)
1. 使用 **PYENV_VERSION** 的環境變數，這個變數可以使用 "pyenv shell " 指令來設置。
2. 如果目前目錄底下的 **.python-version** 存在，則可以用 "pyenv shell" 來修改。
3. 會不斷往 parent 目錄尋找 **.python-version**，直到搜尋到檔案系統的 root。
4. 依照 Global 的 **$(pyenv root)/version** 檔案，可以用 "pyenv global" 來修改。
(如果 global version file 不存在，則 pyenv 會假設你希望用系統內建的 python)

## 安裝教學
1. 安裝 pyenv: 
    >`curl https://pyenv.run | bash`
2. *~/.profile* (加在最前面):
    > `export PYENV_ROOT="$HOME/.pyenv"`
    > `export PATH="$PYENV_ROOT/bin:$PATH"`
3. *~/.profile* (加在最後面):
    > `eval "$(pyenv init --path)"`
4. *~/.bashrc* (加在最後面):
    > `eval "$(pyenv init -)"`
    > `eval "$(pyenv virtualenv-init -)"`
5. 重新啟動 *~/.profile* & *~/.bashrc*:
    >`source ~/.profile`
    >`source ~/.bashrc`
6. 可能需要重新啟動整個 shell。

## 架設環境
1. 安裝 python 版本:
    > `pyenv install <version>`
2. 建立虛擬環境:
    > `pyenv virtualenv <version> <virtualenv_name>`
3. 啟動虛擬環境:
    > `pyenv activate <virtualenv_name>`

## 相關移除方法
1. 移除某個虛擬環境:
    - 去 *.pyenv* 資料夾底下的 *version* 找到 <virtualenv_name> 並刪除。
2. 刪掉整個 pyenv: 
    - 刪掉整個 */.pyenv*
    - 移除 *~/.bashrc* 中的
        > `export PATH="$HOME/.pyenv/bin:$PATH"`
        > `eval "$(pyenv init --path)"`
        > `eval "$(pyenv virtualenv-init -)"`
3. 刪完後重新啟動 shell。

## 移動整個 pyenv
1. 先把 .pyenv 資料夾移動好
2. 更改 ~/.profile 裡面的 #PYENV_ROOT 路徑
3. 去新的路徑中，$PYENV_ROOT/versions 裡面，重新 link 那些環境檔案
> 如果不知道 link 的路徑，可以用 readlink <link file> 檢查以前是怎樣 link 的
