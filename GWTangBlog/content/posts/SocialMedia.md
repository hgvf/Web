+++
title = 'SocialMedia'
date = 2023-11-26T01:37:37+08:00
category = ["NTUST 課程筆記"]
+++

## Ch2
### Power distribution
-    在社群網路中，degree 越大的點會越少，反之則越多。
-    $P(k)=Ck^{-\gamma}=\frac{1}{k^{\gamma}}C$
-    其中 k: degree, C: 不同 network 可能有不同的常數, $\gamma$: 通常為正數。
![power distribution](https://i.imgur.com/uh9FvYM.png)

- - -
### Local clustering coefficient (per vertex)
$$C_i=\frac{vertex_i 的鄰居彼此的 link 數量}{k_i(k_i-1)/2}$$

### Global clustering coefficient (per vertex)
$$C=\frac{3\times network 中三角形數量}{圖中 triplet 數量}$$
-    triplet: 一個點往外連，讓一條 link 連到 3 個 vertices 就算一個 triplet。
- - -
## Ch3
### Empirical network features
-    符合 power-law degree distribution。
-    Small average distance (graph diameter)，類似小世界定理，表示說人跟人之間的 link 最多距離不超過6。
-    Large clustering coefficient，表示說社群網路中會有很多的三角形存在。
-    Giant connected component，表示真實世界中常會有少數個很大的 connected component。


### Phase transition

-    $G_{n, p}$: 一個圖共有 $N=\frac{n(n-1)}{2}$ 條 edges，每條連接的機率是 $p$。
-    $<m>=p \frac{n(n-1)}{2}=N \cdot p$
-    $<k>=\frac{1}{n}\sum_{i}{k_i}=\frac{2<m>}{n}=p(n-1) \simeq pn$ (每個點平均有幾條 edge)
-    Graph density $\rho=\frac{<m>}{n(n-1)/2}=p$
-    **Phase transition** Graph G(n, p), critical value=$p_c$:
    1. $p<p_c$ (k<1)，表示圖中沒有一個 component 超過 $O(lnN)$，且最大的 component 為 tree。
    2. $p=p_c$ (k=1)，圖中最大 component 會有 $O(n^{2/3})$ 個點。
    3. $p>p_c$ (k>1)，有個 gigantic component 有 $O(n)$ 個點，表示幾乎 cover 整個 network。
    (case1, case3 畫出來的圖在結構上有很大的不同)

## Ch4
### Link prediction
-    Network 會隨著時間變化，所以根據 time $t$ 的 snapshot，來預測在時間區間 $(t, t')$ 之間 edge 的狀態。
-    預測方式，只要挑兩個沒有 edge 連接的點計算就好:
       
      > 1. 計算 similarity score C(v1, v2)。
      > 2. 依照相似度分數由高到低排序。
      > 3. 選擇 top n pairs (or threshold)當作新的 link。
      
-    **Scoring function**:
    1. Local neighborhood of $v_i$ and $v_j$
        *    Number of common neighbors:
            $$|N(v_i) \cap N(v_j)|$$
        *    Jaccard's coefficient:
            $$\frac{|N(v_i) \cap N(v_j)|}{|N(v_i) \cup N(v_j)|}$$
        *    Adamic / Adar: 若某個共同 neighbor 其 neighbor 比較少，反而會比較高分
            $$\sum_{v \epsilon N(v_i) \cap N(v_j)} \frac{1}{log|N(v)|}$$
            
     2. Paths and ensembles of paths between $v_i$ and $v_j$
     
        *    Shortest path: 兩點路徑越短越有機會連在一起
            $$-min_s\{path^s_{ij}>0\}$$
        *    Katz score: 考慮所有路徑，計算量大，絕對值裡代表算長度=$l$ 的路徑數量，\beta 則是介於$[0,1]$，所以當 $l$ 越長，所屬的 weight 越小。
            $$\sum^{\infty}_{l=1}\beta^l|paths^{(l)}(v_i, v_j)|$$
        *    Expected number of random walk steps of $v_i$ between $v_j$:
            $$hitting\  time=-H_{ij}$$
            
     3.  Preferential attachment
            *    先各自算兩點的分數，再綜合兩點分數得到總分。
        $$k_i \cdot k_j=|N(v_i)|\cdot|N(v_j)|$$
        or
        $$k_i + k_j=|N(v_i)|+|N(v_j)|$$
        
     4.  Clustering coefficient
            *    兩點若 clustering coefficient 很高，代表兩點之間可能存在很多 triangle，也代表有 edge 的機率很高。
        $$CC(v_i)\cdot CC(v_j)$$
        or
        $$CC(v_i)+ CC(v_j)$$
        
### Effcient prediction and recommendation of social network
        * 可以透過將 network sparsify 來減少計算量。
        * 但是太過稀疏反而使得可以用的 information 減少，準確率就下降。
        * 稀疏化: 把一些 edge 拿掉，使計算量減少。
         
        * Step1: 先透過上面介紹的一些 score function 算出一些 network features。
        * Step2: 依照 features 數值，model 會預測下一個時間點會出現哪些新的 edge。
        * Step3: 實際到下個時間點，查看 network 狀態產生出 label，來更新 model parameters。
        * Step4: 得到新的 network，然後 repeat step2, 3。
        (若在 step1 有做稀疏化，則以後每次得到新的 network 也都要做稀疏化，否則算出來的 features distribution 不在同個維度)
-    **Diverse Ensemble with Drastic Sparsification (DEDS)**
        -    透過 ensemble 多個 classifier，得到比較好的結果，若能增加 diversity，也會使 ensemble 效果更好。
        -    **Diversity**: 若 diversity 不足，那每個 classifier 預測結果也都會很相近，這樣做 ensemble 沒什麼意義。
        -    Diversity 可以透過 $\sigma/|\mu|$ (標準差 / 平均值)來計算。
        - Model 先透過不同的 sparsifying methods 將 networks 轉變為不同的 sparsified networks。
        - 透過 sparsified networks 訓練不同的 classfier 模型參數。
        - Ensemble 透過一些不同的方式得到最終的輸出(ex. 多數決, weighted sum...)。
        - **Variance of ensemble classifier**: $\sigma^2_{\eta^E}=\sum^k_{i=1}w_i^2\sigma^2_{\eta^i}/(\sum^k_{i=1}w_i)^2$.
        - **Weights of Classifiers**: $w_i=d/\sigma^2_{\eta^i}$
        
-    **Degree-based Sparfication**
    - 兩個點之間的 degree 之和若相對高，被保留的機率就越高。
    - ex. Jaccard's coefficient
-    **Random-walk-based Sparfication**
    - 透過有設計過的 random walk (ex. 依照 edge's weight) 實際走一遍，來得到 edge 分數，分數越高被保留的機率越大。
    - ex. hitting time
-    **Short-path-based Sparfication**
    - 設定一個 length threshold，小於這個 threshold 的 edge 被保留的機率較高。
    - ex. Katz score
-    **Random Sparfication**
    - 隨機保留某些 edges，目的是希望模型太過 overfitting。
        

---

## Ch7
### Graph cuts: 越小越好
-    有些型態的 community 不希望出現 overlapping，所以要移除不同 community 之間的 edges，且右希望不同 community 之間要移除的點越少越好，這樣對 network 的破壞比較少。

-    **Graph cut**: 希望兩個 communities 之間的 edges 越少越好，$e_{ij}$ 是指 i, j 兩個 communities 的 edge 存不存在 (0/1)。
$$Q=cut(V_1, V_2)=\sum_{i \epsilon V_1, j \epsilon V_2}e_{ij}$$

-    **Ratio cut**: graph cut 可能導致不平衡的 community size，為了平衡兩個 community 的點數量而產生。
$$Q=\frac{cut(V_1, V_2)}{||V_1||}+\frac{cut(V_1, V_2)}{||V_2||}$$

-    **Normalized cut**: 為了讓兩個 community 的 volume 平衡而產生。
$$Q=\frac{cut(V_1, V_2)}{Vol(V_1)}+\frac{cut(V_1, V_2)}{Vol(V_2)}$$
$$Vol(V_1)=Community\ V_1\ 的所有 degree 總和$$

-    **Quotient cut**: 幾乎同上。
$$Q=\frac{cut(V_1, V_2)}{min(Vol(V_1), Vol(V_2))}$$

### Modularity: 越大越好
-    看 edge 的密度，看是否足以形成一個 community。

-    **Modularity score**: 拿目前的 network 所形成的 adjacent matrix $A$ 與 相同 degree sequence 的 random graph 做比較，介於$[\frac{-1}{2}, 1]$，若只有一個 community 則等於 0。
(random graph 差別只在於他點之間的 edges 是隨機產生的，其他點數、degree 都相同)
$$Q=\frac{1}{2m}\sum_{ij}(A_{ij}-\frac{k_i k_j}{2m})\cdot \delta(c_i, c_j)$$
$$=\sum_u(\frac{m_u}{m}-(\frac{k_u}{2m})^2)$$
$$k_i: community\ i\ 的所有\ degree\ 總和$$
$$\delta(c_i, c_j): 演算法將 c_i, c_j 分在同一個\ community\ 時=1, otherwise =0$$

### Community detection
-    通常透過一些方法將 detection 問題轉換為 optimization 問題，且找到最佳解為 **NP-hard** 問題。

-    **Recursive partitioning**: 運用 divide-and-conquer 的手法，每次將 network 切成兩個 communities，持續下去。

![](https://i.imgur.com/maLNXVq.png)


-    **Multiway partitioning**: 一次就產生多個 communities。

![](https://i.imgur.com/I2k3Nbd.png)

-    **Edge Betweenness**: 以一條 edge 而言，去檢查任兩個 communities 他們的 shortest paths 當中，有幾條有通過該 edge，核心概念就是當這個值越大代表越是重要的 community 橋樑，所以當把值很高的 edge 刪掉，就容易分割出不同的 communities。
$$C_B(e)=\sum_{s\neq t}\frac{\sigma_{st}(e)}{\sigma_{st}}$$ 

![](https://i.imgur.com/9vMDuHZ.png)
(運算量過大，耗時)

-    **Fast community unfolding**: 使用 greedy modularity optimization，可以處理很大量資料的 network，且為 multi-resolution scalable method。

![](https://i.imgur.com/4AlraP3.png)
