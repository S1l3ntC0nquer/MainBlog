---
title: 2024 GDSC NCKU AI Team
date: 2024-06-22 03:33:41
mathjax: true
categories:
  - - DevCorner
    - AI
tags:
  - AI
  - GDSC
excerpt: Sharing the experience of being a member of 2024 GDSC NCKU AI Team. 
thumbnail: https://hackmd.io/_uploads/rJKaUHmIA.png
---

# Intro — What is GDSC
[GDSC](https://gdg.tw/about/gdsc/)(Google Developer Student Clubs)是一個由Google支持的學生社群，旨在通過學生與專業開發人員聯繫，促進學生對 Google 開發人員技術的學習和應用，並為學生提供與技術專家互動和學習的機會。 GDSC 在全球有超過 100 個國家/地區的 1000 多個社群。GDSC 是學生們學習和分享技術的理想社群，並且能夠與技術行業的其他開發人員和專家建立聯繫。(以上皆是Ctrl+c & Ctrl+v)

那我自己在成大待了一年的GDSC，我認為他是個甚麼東西呢？我會說他就是專案社群+課程+演講的集合。這裡集結了一群很有想法的學生聚在一起開心地做專案，然後在這一年的專案旅程中，每兩個禮拜就會有一次社團課程。有時候是幹部們輪流教學一些開發技巧，讓大家對每個領域的技術都稍有了解；有時候是會邀請業界大佬來和我們分享經驗。

總之就是一個非常有想法和對技術非常有熱忱的人所組合在一起的團隊，加入GDSC一定可以感受到那種英雄惜英雄的感覺！接下來，我們就從面試開始講起，來紀錄一下這一年的旅程吧！

# Interview
我在大一的時候(2023)加入了成大的GDSC，當初面試了三個組別，志願序從前面到後面分別是AI組、Web組、Data組。在面試的時候主要就是會問一些你之前有沒有過相關的經驗或是一些知識性的問題。舉AI組的面試來說，當初幹部就有問我甚麼是P value、我所熟悉的程式語言、然後給我一些題目問我要怎麼去找出兩者之間的相關性（具體問題有點忘記了，畢竟是一年前。但我記得我的回答跟Linear regression有關），再來就是可能要清楚Correlation doesn't imply causation等。除了這些比較Hard skill的問題，也會問你如果錄取了會如何安排時間以及GDSC對你而言的Priority等等。現在回頭看也十分可以理解當初為甚麼這麼問，畢竟一整年要和大家一起做專案，中途跑掉肯定是不太好的。

後來面試完，我順利地加入了第一志願的組別，AI組。雖然當初對於AI的知識沒有甚麼了解，但這剛好給了我一個機會開始展開學習。

# About my team
因為這次的AI組人有點多，所以我們又有分成了幾個小組。而我分到的這個小組中，我是唯一一個大一生。其他的成員從大三到碩班都有（AI組也總共兩個大一的><），總之我就是裡面資歷最淺的那位。但即便如此，也不會感受到被排斥的感覺，大家也不會因為我年紀比他們小而忽視掉我在團隊中的想法（大家真的很友善，很有Google那種多元友善包容的感覺XD）。

# Our project
我們在歷經將近一個學期的討論還有熟悉彼此以及熟悉這個團隊之後（具體來說是在寒假），討論出了我們的專案主題。我們想要透過AI模型的分析，讓我們能夠預測股票的價格走向。我們這次訓練模型是採用特斯拉的股票來作為我們的對象，並且我們除了一般的指標外，我們另外加入了推文、新聞和特斯拉財報作為我們的Indicators。我們的整體流程如下：
> 1. 針對推特的推文、各大媒體新聞、特斯拉財報這三個不同的文本訓練出不同的Sentiment analysis model
> 2. 訓練一個準備接收上述資料做預測的LSTM
> 3. 利用我們的SA模型去Label每日的資料，並傳入一個LSTM 

最後我們每天輸入當日的三種文本的SA分數到我們的LSTM去預測隔日的股票價格，並成功得到出了還蠻精準的結果，更詳細的可以看我們的海報，如下（最下面的圖片中，右下角的指標有誤植。藍色的線為真實股價，橘色的線為我們預測的）：

![我們的海報](https://hackmd.io/_uploads/rJKaUHmIA.png)

# My role in the team
我在我們這組是負責推文的部分。一開始，我是先用Kaggle找到的[這個資料集](https://www.kaggle.com/datasets/omermetinn/tweets-about-the-top-companies-from-2015-to-2020/data?select=Company_Tweet.csv)以及[cardiffnlp/twitter-roberta-base-sentiment-latest](https://huggingface.co/cardiffnlp/twitter-roberta-base-sentiment-latest)和[austinmw/distilbert-base-uncased-finetuned-tweets-sentiment](https://huggingface.co/austinmw/distilbert-base-uncased-finetuned-tweets-sentiment)這兩個Pre-trained Model去幫我們的資料做Label。

但是後來我們希望可以有更近期的資料，所以我寫了[這個爬蟲](https://github.com/CX330Blake/X-crawler)來爬取2021到2024五月的推文的資料（但因為我沒有買API，所以只能慢慢用Selenium滑）。在蒐集完資料後，我用了那兩個Model去做Label，並且在Label完後抓取相同比例的Positive、Neutral、Negative的資料去對BERT做Fine-tune，得到[最後的SA模型](https://huggingface.co/CX330Blake/tweet-sentiment-analysis-for-tesla)。

# NCKU GDSC Forum
終於到了成果發表的當天，當天就是有很多的講座以及我們自己擺設的每個小組的攤位。我們要對每位到攤位前的觀眾做講解，跟他們說我們到底做了些甚麼，然後當天的講座也是十分有趣，會很有收穫。我在聽完了前輩們的分享後，更有了想要努力前往矽谷的衝動，有種受到矽谷召喚的感覺XD。至於更詳細的，就看一下以下的紀錄影片吧！
<div style="position: relative; width: 100%; height: 0; padding-bottom: 56.25%;">
    <iframe style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" src="https://www.youtube.com/embed/MWK4geiYARs" frameborder="0" allowfullscreen></iframe>
</div>

# What I learned in this year
撇除掉一些Hard skill（這是一定會學習到的），我比較想來談談加入GDSC的一些Trade-off。加入GDSC後，勢必會壓縮到其他的時間，所以要更懂得如何安排自己的休閒娛樂以及讀書的時間。但同時，也確實可以在這裡獲得很多學業之外的東西。比如說拓展自己的人脈、吸取更多前輩的經驗，但我覺得最重要的，是可以找到一群志同道合夥伴們。這一來可以讓自己在自我精進的這條路上更不孤單，其次也可以增加自己對這種開發社群（或是身為「開發者」）的身分認同。

結論上來說，如果你同時有很多其他的活動，或是想要好好的衝一下GPA，或許可以慎重考慮一下要不要參加（因為畢竟做專案真的會花不少時間，還要上社課）；但如果你也和我一樣，相信比起分數有更重要的事，那就大膽地來參加吧，一定會收獲不少的！

最後附上我們今年GDSC的口號：

> Code your goal, Fuel your soul! — ***2024 GDSC NCKU***

如果這篇文章對你有幫助，歡迎[訂閱我的Blog](https://cx330.tw/subscribe/)，會在有新文章的時候通知你，十分感謝！