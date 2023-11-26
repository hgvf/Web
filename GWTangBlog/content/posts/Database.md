+++
title = 'Database'
date = 2023-11-26T01:37:15+08:00
category = ["NTUST 課程筆記"]
tags = ["Database"]
+++

## ER Model
-    Types of attributes:    
    1. **Simple**
    2. **Composite**, composed of several components: Name (FirstName, MiddleName,...)
    3. **Multi-valued**, may have multiple values: Student (High-school, College,...)
    
-    **Weak Entity**: 沒有 key attribute，必須有另外一個 relationship 當作 owner 來協助辨識 weak entity，辨識的組合如下:
    1. Partial key of weak entity.
    2. 另外一個 strong entity，用來辨識 weak entity。
-   圖示:
![annotations](https://i.imgur.com/ngHgnZY.png)

-    ER Model 範例:
![ER_Model](https://i.imgur.com/NkiEtw1.png)

## Relational Model
-    **Superkey**: 在一個 relational table $R$ 中，任兩個 rows 的 superkey 內容都不可能相同，superkey 是一個集合。

-    **Key**: Superkey of $R$，可以想成是 superkey 的最小集合，移除 key 中任何一個 attribute 都會使集合不符合 superkey。

-    **Candidate key**: relation schema 可能有很多的 key，candidate key 就是由很多 key 組成的集合，key 則是從中挑選。
-    **Foreign key**: 在 $R_1$ 中的 foreign key 可以是 null 或是與 $R_1$ 相關的 relation $R_2$ 中的 primary key ; 若 foreign key = null，則不能成為 primary key 的一個 attribute。
-    Relation schema 範例:
![relation schema](https://i.imgur.com/7U2ASUC.png)

## Relational Algebra
-    一些範例:
![Relational Algebra](https://i.imgur.com/UwmpPwN.png)

-    **JOIN**: 取代昂貴的 cartesian product，$R_1\bowtie_{join\_condition}R_2$，只有符合 join condition 的 tuples 才會出現，又稱為 Theta-join，而若是 join condition 是用等號，則又可以稱為 EQUIJOIN。
![JOIN](https://i.imgur.com/2zNa47E.png)

-    **NATURAL JOIN**: 因為 JOIN 會回傳兩個 table 各自的 attribute (上圖的 ssn 與 mgr_ssn 重複)，為了避免這樣的浪費才有了 natural join，使用條件為 **兩個 table要比對的 column name 必須要相同**。
-    **OUTER_JOIN**: 在 NATURAL JOIN & EQUIJOIN 中，不符合條件的 tuples 都會被刪除，而 OUTER JOIN 則是會全部都保留。
    1. **Left/Right/Full outer join**: 經過運算後會保留左邊/右邊/兩邊的所有 tuples。
    2. left outer join: ![left](https://i.imgur.com/dXFti7B.png), right outer join: ![right](https://i.imgur.com/7egKJNQ.png), full outer join: ![full](https://i.imgur.com/jaX5Xkq.png).
 
-    **Agregate**: $F_{MAX/MIN/SUM/COUNT... Salary}(EMPLOYEE)$，有各種不同 functions 來操作。

## Relational Calculus
-    一種 nonprocedural language，在找出符合 query 的過程中沒有順序，跟 relational algebra 有很大不同，通常是在 tuple 或是 atribute 上篩選條件。
-    Format:  $\{t|COND(t)\}$.
-    Tuple Relational Calculus: $\{t.FNAME, t.LNAME | EMPLOYEE(t) AND t.SALARY>50000\}$.
-    Existential/Universal: 存在、至少/全部都符合 -> ![existential](https://i.imgur.com/csfGekf.png) /![universal](https://i.imgur.com/yNK80it.png) ex. 
![example](https://i.imgur.com/TZsuj5t.png)

## Domain Relational Calculus
-    要窮舉出所有 attributes，並給定每個 attribute 一個變數，才能篩選條件。

-    Format: $\{x_1, x_2, ..., x_n|COND(x_1, x_2, ..., x_n, x_{n+1}, ..., x_{n+m})\}$, ex.

![example](https://i.imgur.com/0Z0nlJ2.png)

## SQL
-    **HAVING**: 放在語法最後面，目的是在前面語法找出來之後，再用最後一個條件去篩選，ex.

![having](https://i.imgur.com/XQBRNSe.png)

## Query processing
-    **Heuristics in Query Optimization**: 執行後產生結果越多的 query 越先執行。
    1. SELECT operation 會最先執行，也就是在 query tree 最底下。 
    2. 比較 restrictive 的 SELECT 會最先執行，也就是 select 完有較少結果的越先執行。
    3. 用 JOIN 取代 CARTESIAN PRODUCT。
    4. 每次 JOIN 完後用 PROJECT 縮小輸出的 attribute 數量。
    5. Query tree 範例如下:
    ![](https://i.imgur.com/DoO8QHE.png)

    
-    **Query graph**: 範例如下 (中括號為輸出 attribute，類似 SELECT): 
![query graph](https://i.imgur.com/g9ujUKG.png)


## Normalize
-    **Anomaly**:
    1. Update anomaly: 更改 PROJECT name 時，employee 對應的 project name 都要更改，可能要改很多次。
    2. Insert anomaly: Cannot insert a project unless an employee is assigned to it.
    3. Delete anomaly: 若刪除某 project，可能導致該 project 的成員一併被刪除。

-    **Functionally dependent X->Y**: 
> 若有兩筆 tuples 的 X 集合中的所有 attributes 都相同，  則這兩個 tuples 的所有在 Y     中的 attributes 都會相同。

    Inference rules for FDs:
            
            1. Reflexive: if Y is subset of X, then X -> Y
            2. Augmentation: if X -> Y, then XZ -> YZ
            3. Transitive: if X -> Y and Y -> Z, then X -> Z
            4. Decomposition: if X -> YZ, then X -> Y and X -> Z
            5. Union: if X -> Y and X -> Z, then X -> YZ
            6. Psuedotransitivity: if X -> Y and WY -> Z, then WX -> Z
            
    Rules: 
            1. Closure: F 集合的 FDs closure = 所有 FDs 集合中 can be inferred from F.
            2. Equivalent, F=G: 
                - F/G can be inferred from G/F.
                - F, G 的 closure 相等。
            3. Covers, F covers G: 如果 G 中所有 FD 都可以 inferred from F。
    
    Minimal sets of FDs:
            1. Every dependency in F has a single attributefor its RHS.
            2. We cannot rmove any dependency from F and have a set of dependencies that is equivalent to F.

-    **Prime attribute**: 必須是 candidate key 的成員。
-    **Nonprime attribute**: prime attribute 的相反，也就是不是 candidate key 的成員。
-    **Full functional dependency Y -> Z**: 移除任何在 Y 集合中的 attribute 就會使 FD 不成立。
-    **Transitive functional dependency**: FD X -> Z can be derived from two FDs X -> Y and Y -> Z.
-    **1NF**: 避免出現 multivalued, composed attribute 或是 nested relations。

-    **2NF**: 所有非 prime attributes 都 fully functionally dependent 到 primary key。
    
-    **3NF**: 在已經是 2NF 的情況下，不能有任何 non-prime attribute transitively dependent on the primary key。
    
-    **General Normal Form**: 上面的 normal form 是考慮 primary key 情形，這裡則是考慮考慮更實際的情形，例如 multiple candidate keys:

            General 3NF:
                若有 FD: X->A，則滿足以下其中一個條件
                1) X is a superkey
                2) A is a prime attribute 
        
-    **BCNF**: 類似 General 3NF，但只滿足 X is a superkey。
