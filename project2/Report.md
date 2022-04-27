# AI Capstone Game Project 1
> Team ID :**２**  
> Team Name : **Contributed by GitHub**  
> Member : **0816004 吳原博**, **0816001 王朋睿**, **0716089 王柏舜**  
> This project is done by 吳原博  

## Battle Sheep
### Introduction
在這次的 project 裡，我們決定用搜尋演算法來實做 agent，以隨機取樣的方式找到最適合的下一步  
### Algorithm
以 Monte Carlo Tree Search 決定每步驟的效益，細節如下 :  
+ **Monte Carlo Tree Search**
    ![](https://i.imgur.com/QvbkpK6.png)
    
+ **InitPos**  
    遍歷全部合法的起始位置，找到可移動的方向最多而且可以移動的距離總和最遠的位置當作是agent的起始位置，如果這樣的點有好幾個就隨機選一個出來。
+ **GetStep**  
    MCTS 的結構如下(psuedo code) :  
    ```python
    def MCTS(node):
        while 4.5 秒內:
            expand_node = tree_policy(node)
            reward = default_policy(expand_node)
            back_propagation()
        best_child = choos_best_child(node)
        return best_child
    ```
    1. tree policy  
        這裡包含 selection 和 expansion，selection 的部分根據 UCB1 公式(為了方便我有做一點修改，把 n 的分母和根號 2 約分掉，但是跟原本的公式沒有不同) 選擇分數最高的 child node，而 expansion 則是從可移動的點隨機取一個以隨機數量往隨機方向移動，並且任一個玩家移動一次後就會產生一個新 node。  
        + UCB1 公式 :  
            ![](https://i.imgur.com/LdowQZI.jpg)   
    2. default policy  
        mcts 裡 rollout 的部分。模擬四個玩家輪流隨機移動直到終局，並且以該節點用有者的終局分數做為 reward，同時 reward 還會加上 (4-排名)*10 分，以鼓勵 agent 除了盡量拿高分，也要把排名納入考量。  
    3. back propagation  
        更新 rollout 過程所有 node 的 visit time 還有 quality value。
    4. choose best child  
        根據最高平均分數 (quality value / visit times) 選擇 best child node，並且回傳下一個 step。  
### Analysis
和助教提供的 sample 對局基本上每場皆贏，也不會在最基本的地方犯錯(例如把大量的綿羊分配到無路可走的死路裡面)，平均表現可以在終局時拿到 50 幾到滿分 64 分，但是當 agent 必須和其他玩家互動時，很多時候還是不會做出最適當的決定。  

例如在下圖之中，如果黃色的 agent 往左下(5) 的方向走，就可以直接封住紅色的路而直接少一個對手，但是黃色卻選擇往右邊移動。  
![](https://i.imgur.com/RsP890x.png)  

這張圖裡面的黃色其實有機會把那六隻綿羊移走，但是最後卻錯失機會而被封死。  
![](https://i.imgur.com/A1BerZl.png)  
以上兩種情況算是多次遊戲下來後找出來比較明顯的案例，並且有些對手是我自己複製出來的幾個 agent，不一定是助教的 sample。  

我後來把 mcts 執行的過程輸出來分析，發現比較合理的解釋是這些選擇是偶然被機器認為是最佳的選擇。由於機器是用隨機 rollout 後得到的分數來做決定，並且個人的終局分數和排名才是做決定的關鍵，因此在第一個案例裡，我認為是紅色本來就沒有太大的威脅，講明白點不管其他人怎麼走他都是最後一名，因此有沒有堵住紅色並沒有造成太大的影響。  

至於第二種情況，我認為是在 rollout 時剛好比較少模擬到藍色向上堵住的情景。因為 rollout 時是模擬四個人輪流往隨機方向以隨機數量移動隨機羊群，而上圖中理論上具有高威脅的藍色羊群卻只有 3 隻，因此模擬過程裡不容易出現此羊群往上封路的情形。  
### Discussion
+ **起始位置**  

    這裡我們原本打算要用監督式學習的方法來挑選起點，但是在有限的時間精力裡，我們不具有這樣的能力將這些想法實現，因此退而求其次的想了其他做法。  
    
    一開始我嘗試挑選多個點以 mcts 模擬多次直接去算分數，但是在 5 秒內這樣做效益實在不大，不但挑的點不多，模擬次數也不夠。再加上除非是特別極端的情況，大部分起始位置也不容易看出來對遊戲的影響，因此後來決定，不如直接用一個簡單的方式找出我們認為最空曠的地方。  
+ **定義 state**  

    一個 state 裡包含當前玩家 ID、mapStat 和 sheepStat，透過這些資訊可以推得 remain move 等其他資訊，同時也可以用來模擬遊戲。  
+ **定義 node**  

    每一個 node 包含 parent, children, visit times, quality value, from step, state，用來作為 mcts 裡面的節點。  
    
    最一開始我是用四個玩家輪流動一次後作為一個 node，這樣的做法應該是不正確的，因為最後選出來的 best child 可能是對手做了較弱的選擇導致，而對手不會按照你所模擬的方法移動。但是這樣的作法在不太受對手干擾時卻意外的表現不差。  
    
    後來改成每個玩家移動一次就會產生一個新的 node，每個 node 除了當前 state 以外也包含上一個玩家所做的選擇(from step)，這樣機器才會真的按照玩家的下一步來做選擇。而 quality value 則是改成一個 node 上一個玩家的 reward，因為是上一個玩家的選擇才會產生現在的 node，而且不管擁有 node 的玩家是自己還是對手，我都會按照這個規則更新 quality value。一開始我有想過用對手的 reward 來做為其他 node 的 quality value  會不會導致機器總是模擬對手的特定幾步而產生偏見，但是我發現其實在四秒內通常也來不及把第二層給 expand 完，因此後來也沒有很在意這件事了。  
+ **UCB1 的參數**  

    這裡我主要討論的是用來決定 exploration 的參數 n，在寫作業的其間我注意到有時候機器會因為某個決定偶然間得到高分而不斷的去模擬它，尤其因為我的 reward 通常在 40 到 90 之間，變成原本的 UCB1 公式 exploration 的部分影響非常的小，因此我曾經把這個 n 從 1 調整到 30 都試過幾遍，但是我發現即使把 n 調到很高也不一定有很好的效果，反而可能是因為有些選擇模擬太少次結果變得不準確。  
    
    我也有試過讓 n 隨著遊戲的進行減少，因為我認為到了遊戲越後期，探索的重要性就越低，應該要著重於在最後幾部裡拿到最多的分數。不過後來我覺得這樣的想法可能有點多餘，因為到遊戲後期機器可以在較少的選擇裡模擬更多次，而那些分數好的選擇其實不管怎樣都會脫穎而出。  
    
    最後我覺得讓機器一直去跑那些高分的選擇也未必不好，因為如過他們真的只是偶然得到好成績反而會因為模擬多次而原形畢露，只不過時間都花在這樣的選擇上又會導致其他有潛力的選擇沒辦法被模擬到，寫到這裡我才體會到甚麼是 "exploration exploitation trade off" 的難題。  
+ **定義 reward**  
    
    reward 的定義應該是整個 mcts 最核心的部分，調整 reward 的確也會對機器有明顯不同的影響。  
    
    一開始的 reward 設計很簡單，我直接採用遊戲的終局分數，效果其實也不差，但就是差強人意了一點。後來我改成用排名來做分數，1 到 4 名分別可以得到 3, 2, 1, 0 分，結果機器看起來變得像是"有贏就好"，分數變得比以往都來的低，而且有不代表真的每次都會贏。我也有試過用自己的終局分數減掉其他人的分數平均，期待機器會去干擾其他人，但是這樣有時候機器有變的太過激進，贏不贏不重要它就是要干擾別人。因此最後我採用排名加終局分數的做法，為了讓排名有足夠的影響力 1 到 4 名分別可以加 30 到 0 分。  