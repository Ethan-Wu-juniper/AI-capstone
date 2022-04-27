# AI Capstone - Project 1
> 資工系 0816004 吳原博

## Public non-image dataset

### Dataset
+ 資料名稱 : Bank Marketing Data Set
+ 來源 : https://archive.ics.uci.edu/ml/datasets/Bank+Marketing
+ 資料視覺化 :  
<img src=https://i.imgur.com/J0lVevZ.png width="200"/>  <img src=https://i.imgur.com/a0ibx1g.png width="200"/>  
<img src=https://i.imgur.com/hrbFxyT.png width="200"/>  <img src=https://i.imgur.com/vtiuOx0.png width="215"/>  
+ 簡述 : 
    + 此資料集為銀行電話問卷調查的結果，旨在調查什麼樣的客戶會認購銀行定存  
    + 網站上總共提供四種資料集 -- (0) bank (1) bank-additional (2) bank-full (3) bank-additional-full，以下以編號簡稱之。其中 (2)(3) 各有四萬多筆資料，並且 (0)(1) 為前兩者分別取十分之一的資料。(0)(2) 只有16種 attribute ，而且幾乎都是 categorical。(1)(3) 則另外新增5種 numerical 的 attribute，但又比前者少了1種(以實際資料為主，這和網站上的說明有一些出入)  
    + label 只有兩種，yes 和 no，其中認購定存的客戶往往遠少於拒絕的人  
+ Data shape (包含 label): 
    (0) bank : (4521, 17)
    (1) bank-additional : (4119, 21)
    (2) bank-full : (45211, 17)
    (3) bank-additional-full : (41188, 21)  

### Algorithms
我總共使用三種演算法分析這份資料，分別是 naive bayes, decision tree 和 random forest，並且全部都是使用 sklearn 的套件來完成實作  

### Analysis
+ 我使用 3-fold cross validation 來檢驗模型成效，各模型在各資料的表現如下(由於 confusion matrix 比較多，而且只有兩個 label，就不放上來了) : 
    1. naive bayes
        ![](https://i.imgur.com/xJYcBJv.png)
    2. decision tree
        ![](https://i.imgur.com/kx56unA.png)
    3. random forest
        ![](https://i.imgur.com/Rp3EcTs.png)
        
### Discussion
+ 以上附圖為三個模型在不同資料上的表現，rank 代表四個資料集各項分數的排名，points 是我把每個資料集不同分數的表現根據排名做粗略的統計，排名越前面得分越高
+ 從不同模型的表現來看，大致上符合 random forest > decision tree > naive bayes，這樣的結果很符合我的預期，因為 naive bayes 是三種模型裡最簡單的
+ 對於同一個模型不同四個資料集 (0)(1)(2)(3)，大致符合 (3) > (2), (1) >(0), (3) > (1), (2) > (0)，說明白了就是資料量越多、attribute 越豐富，模型的表現就越好
+ 大部分情況下，資料最豐富的 (3) 往往在每項分數上都是略勝一籌，不過有時候 (1) 會有教突出的表現，而 (1) 的資料量卻只有 (2) 和 (3) 的十分之一。以下幾點是我的推測 ：
    1. **這是偶爾產生的意外**
        > 由於模型在四個資料上的表現效果相近，有時候結果和預期相悖也是正常的  
        
        雖然不只一次發生上述的現象，而且常常都是一次好幾項分數產生這種結果，但是往往差距不大，也許這不是一個太值得注意的問題
    2. **model 能力不足**
        > 增加資料無助於增加表現
        
        這樣的情況比較少在 random forest 出現，有可能數千筆這樣的資料對於 decision tree 和 naive bayes 來說已經足夠。但是同樣是增加十倍的資料量，(0) 和 (2) 卻鮮少有同樣的結果，這也可能不是最完美的解釋
    3. **attribute 選擇不當**
        > 新增的五個 attribute 不適合用來評估目標
        
        這樣的推測只是基於觀察 (0)(2) 和 (1)(3) 的差別所得出的粗略推測，一方面我也不清楚那五個 attribute 的專有名詞代表什麼意思，所以也很難進一步做推論
+ 除了上述情況，這次結果最吸引我注意的是模型在 "yes" 上的表現總是相當的差，即便增加資料也一樣，代表這不僅僅是一般的 overfitting 問題。這是不符合預期而且不希望在這類模型上看到的，因為比起透過傾向猜測 "no" 來增加準確率，銀行應該會更希望準確的找到願意收購定存的客戶，但模型在 "yes" 上不管是準確率和回召率都低。如果今天預測對象改為判斷目標是否得到高傳染率的流感，這種模型就幾乎沒有參考價值。我認為造成這種現象是因為兩種標籤的比例嚴重不均衡，而且 decision tree (random forest 的本質也是 decision tree) 又容易 overfitting，才會導致模型傾向猜測 "no"  

### Experiment
根據我所選的演算法，我做了以下幾個實驗，透過改變 hyper-parameter 或是修改資料等等來嘗試改善模型的表現
+ less layer for decision tree
    + 一般的 decision tree 會不斷長出新的 node，直到全部資料被分完為止，而做 leaf pruning 似乎可以減少 overfitting 的問題。這裡我選擇用 (2) bank-full 來做實驗，一方面是因為它的準確率較高，一方面是因為它的層數普遍最多(通常有35層左右)。我把深度從5開始到35，每隔五層都試過一遍，另外還試了只有兩層(原因稍後解釋)，下圖為其中一次結果 : 
    ![](https://i.imgur.com/3ei7Hho.png)
    + 我有點意外的發現，基本上層數越少表現越好，因此我很極端的試了只有兩層的模型，結果竟然不比 30 層以上的差，看來真的不是越多層越好
+ more trees in random forest
    + random forest 基本上是一堆 decision tree 的集合體，而我原本的 random forest 是用 100 棵樹，我試著增加樹的數目來做實驗，結果如下 : 
    ![](https://i.imgur.com/ufzt7Yq.png)
    + 其實是看不太出來差別的，我原本預期增加樹至少要變得好一點點，但是看來 100 棵樹也就差不多了
+ more balanced data
    + 在原本的結果裡，會有 "yes" 分數特別低的問題。我在想，既然無法增加資料，那我就反其道而行減少 "no" 的資料。我使用原本就效果最好的 random forest 訓練刪減過的 (3) bank-additional-full，結果如下 :  
    <img src=https://i.imgur.com/YyzoJqw.png width="150"/>
    + 我從 (3) 中 label 為 "no" 的資料裡隨機挑選跟 "yes" 一樣數量的資料，合成出一個新的資料，使其中兩個 label 數量相同。其餘的資料我把它另外存起來做成 test set (雖然裡面只有 "no")，結果效果還真的不錯。雖然原本高表現的 "no" 分數變低了不少，但取而代之的是 "yes" 明顯增加，而且整體準確率也沒降太多。我把剛剛做的 test set 拿進這個模型測試，發現 accuracy 也有 0.85 多，至少模型不會再抱有太過分的偏見了

## Public image dataset

### Dataset
+ 資料名稱 : Multi-class Weather Dataset for Image Classification
+ 來源 : https://data.mendeley.com/datasets/4drtyfjtfy/1
+ 簡述 : 總共有一千多筆資料，全部都是以天空為主的風景照，圖片大小不一。總共有四個 label，cloudy(多雲), rain(下雨), shine(晴天), sunrise(日出)，個人覺得圖片辨識度算蠻高的

### Algorithm
我使用 CNN 來實作圖片辨識，並且以 pytorch 的套件搭建模型，模型資訊如下 : 
```(python)
    self.conv1 = nn.Sequential(
            nn.Conv2d(3,16,5,1,2),
            nn.ReLU(),
            nn.MaxPool2d(2)
        )
    self.conv2 = nn.Sequential(
            nn.Conv2d(16,32,5,1,2),
            nn.ReLU(),
            nn.MaxPool2d(2)
    )
```
由兩個包含捲積層、activation function、池化層的 sequential 層所組成  

### Analysis
+ 圖片在被丟進模型之前，一律會被 resize 成 224*224 大小的 tensor
+ 我使用 holdout validation 把資料切成 7:3，我覺得 confusion matrix 的結果還不錯就也一起貼在下面了 :  
<img src=https://i.imgur.com/jDM7CVI.png width="300"/>

### Discussion
+ 整體而言表現還是不差，下雨的分數大部分時候還是低了點，我去看了原本的資料才發現下雨的圖片的確是比較複雜，除了刻意加強雨滴的線條，可能會有包括街道、樹林、天空等各種背景，甚至還有動漫圖片...  
+ 日出的準確率通常較高，我看了下資料發現落日的圖片的確都很相似，基本上畫面都是一片橘紅、清楚的拍到落日並且背景都是山或海
+ 在四個 label 中，機器必較容易搞混多雲和下雨，多雲的圖片普遍都是拍攝一片灰白的天空，也許這是跟那些背景單純的下雨搞混了?
+ 有時候會出現特定一項且下雨以外的分數特別差(爾偶發生，我沒有截到圖片)，我推測是 kfold validation 切資料時剛好那一項的資料切得比較少，我想可以透過改變切資料的方法(比如說把每個資料都切 k 份再合起來，而非一次切全部的資料)，或是增加資料量來解決問題。不過這種情況較少發生，而且多跑幾次就可以避免了


### Experiment
> 這裡我決定放 confusion matrix 而非各項分數，因為我覺得當 label 較多時 matrix 的呈現效果很不錯。不同的 confusion matrix 之間可能會有小矛盾，這是因為我直接把不同 fold 預測出的結果加起來平均並且四捨五入到整數位，以方便閱讀
+ 比較不同 epoch :  
    + 我分別嘗試當 batch size 為 5，epoch 分別為 1, 5, 20 的情況，結果如下 :  
    
        | epoch | confusion matrix | accuracy score |
        | -------- | -------- | -------- |
        |  1   |<img src=https://i.imgur.com/RFtl5wF.png width="250"/>|0.664|
        |  5   |<img src=https://i.imgur.com/MNvfYO8.png width="250"/>|0.799|
        |  20  |<img src=https://i.imgur.com/qC0K4sL.png width="250"/>|0.880|

    + 看起來在 epoch 20 以內，epoch 是越多越好，epoch 等於 20 時連下雨都猜得蠻準的。不過就我所了解 epoch 太多會導致 overfitting，只是 CNN 訓練速度很慢 gpu 又不夠，就沒繼續試下去了  
+ 比較不同 batch size :  
    +  這次是固定 epoch 為 5，batch size 分別為 8, 64, 256，結果如下 : 
        | batch size | confusion matrix | accuracy score |
        | -------- | -------- | -------- |
        |  8   |<img src=https://i.imgur.com/QKmCVMo.png width="250"/>|0.867|
        |  64  |<img src=https://i.imgur.com/MNvfYO8.png width="250"/>|0.799|
        |  256 |<img src=https://i.imgur.com/R9mAxds.png width="250"/>|0.733|
        
    + 結果 batch size 反而是小一點比較好，而且影響其實蠻明顯的。我想這是因為資料量比較小所導致，不然我原本預期 batch size 是大一點比較好  
+ Data Augmentation : 
    
    1. flip image : 
        > 我直接複製一份資料，把圖片全部翻轉，並且和原本的資料合併後拿進模型訓練
        + validation 的準確率高達 0.9，以下是各項分數 :   
            <img src=https://i.imgur.com/a7t7vBL.png width="300"/>  
        + 拿原本未處理的圖片進去預測，準確率更有 0.92，附上結果 :  
            <img src=https://i.imgur.com/KFUuj16.png width="300"/>  
    2. 加一大堆 : 
        > 我試著把圖片分別左右翻轉、擷取一部分、調整亮度, 對比, 飽和, 色調、還有小角度旋轉各做一份，組成一個比原本大好幾倍的圖片庫，試試看這樣不分青紅皂白的做圖像擴增會有什麼結果
        + validatin score 有 0.92 :  
            <img src=https://i.imgur.com/eM1n1Je.png width="300"/>
        + 請模型預測原本的資料集，accuracy score 竟然有 0.99!  
            <img src=https://i.imgur.com/UYtWLxu.png width="300"/>
    + 圖片翻轉後結果會變好這還在我的預料內，但是我真沒想到不顧後果的加上好幾種圖像擴增竟然效果會變得更好，我原本預期那些不太合適的擴增圖會導致模型誤判(比如說改色調後的圖片就和原本差蠻多的)。雖然有可能是我用原本的資料集下去預測，但是畢竟在做 kfold validation 時不會把所有圖片都丟進模型訓練，而且 flip image 也是做一樣的事情，所以我傾向認為這真的是一個好的結果
    + CNN 訓練起來真的超慢，一旦開始執行起手式就是一個小時起跳，colab 的 gpu 又一下子就用完了。所以上面有些結果也許多跑幾次就會不同，但是我還是以目前能得到的數據為準來寫報告 

## Self-made dataset

### Dataset
+ 資料名稱 : twitter emoji 表情預測
+ 來源 : 透過程式爬蟲下來，並且和 **王柏舜** 和 **王朋睿** 一起蒐集並共用同一份資料
+ 簡述 : 目前蒐集了兩萬多筆資料，刪掉重複或不適合的資料並經過整理後留下文字內容。其中以 "@" 為開頭的 tag 訊息還有 "https://" 開頭的網站都被視為無益於模型的雜訊被移除了。另外 emoji 則被移除作為 label，所以資料在被更進一步的預處理之前只有文字和 label 兩個維度
+ label 編號如下 :  
    |  0 |  1 |  2 |  3 |  4 |  5 |
    | -- | -- | -- | -- | -- | -- |
    | 😜 | 😍 | 😉 | 🔥 | 💜 | 💯 |


### Alogrithm
這裡我使用 keras 套件實作神經網路的模型，並且特地加入 LSTM 層希望能對 nlp 有所幫助。另外我使用了網路上找到的 pretrained 過的檔案 **glove.6B.50d.txt**，用來把文字轉成 vector 特徵值，讓資料可以被模型讀進去

### Analysis
+ 經過幾次模型和程式碼的修改以後，validation score 最多大概只能升到 0.48 左右
+ 使用 3fold cross validation 的準確率為 0.47 :  
    各項分數 :  
    <img src=https://i.imgur.com/j3ojmjc.png width=400>  
    confusion matrix :  
    <img src=https://i.imgur.com/n0gmQcE.png width=300>

### Discussion
+ 可能是我對自然語言處理還不熟悉，或是抓到的資料辨識度就是不高，但是效果一值不好。經過多次修改模型還有 hyper-parameters 也沒有顯著的提升。現在的結果是以上述的模型做出來的成績
+ 爬下來的資料經過篩選後變得不平均，而即使是這樣差的結果也能發現模型會傾向往資料較多的 label 去判斷，從 confusion matrix 可以明顯看出有大量的資料被誤判為數量最多的 1(愛心眼睛)。其中 4(紫色愛心) 的 precision score 試好幾次都是最低，而且是慘到接近 0 的程度，我想除了資料不平衡也有可能是因為愛心的文字內容比較相近，因此模型更容易把 4 判斷成 1

### Experiement
+ 我發現上述的 glove 檔案不只一種，分別還有 100 維到 300 維的檔案，因此決定試試不同檔案的效果，結果如下 :  
    | 維度 | 50 | 100 | 200 | 300 |
    | ------- | -------- | -------- | -------- | -------- |
    | validation score | 0.46 | 0.47 | 0.48 | 0.48 |  
    
    看起來維度高一點似乎會比較好，不過整體而言來是偏糟
+ 如果還有時間的話，我會希望可以再爬更多的資料，讓每個 label 的資料量更平衡
+ emoji 的選擇也許可以再思考一下，我想盡量避免意義相近的表情符號
+ 在 nlp 裡，似乎會在前處理時去掉句子裡的 'stop words'，如果有時間這也是我想嘗試的東西之一

## Conclusion  
+ 為了完成這次的作業，我花了許多時間去了解這些模型以及 hyper-parameters 對他們產生的影響，讓我對這些演算法有更進一步的了解
+ 撰寫這份作業也讓我有機會去分析模型訓練後的成果，並且根據這些結果試著對資料和模型作改良，對我來說是可貴的經驗
+ 自己蒐集資料集是一個蠻大的挑戰，最後選擇用爬蟲來做也是因為這樣能最有效率的在時間內蒐集大量的資料。我在不但在這樣的過程裡精進了自己的程式技巧，也學到在如何整理資料，為資料下標籤以及做資料的篩選
----------------------------------------------------------------
> 報告的部分到此結束，接下來是附在報告後的原始碼，謝謝助教!