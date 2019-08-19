# TMBA MultiChart contest
<img width="150" height="150" src=https://img5.androidappsapk.co/300/4/6/5/com.Kway.MultiCharts.png>

#### 資料期間 : 
2012.01.01~2019.06.31
#### 商品標的 : 
台指期(TXF)
#### 手續費 : 
滑價600

#### 策略發想
根據有效市場理論(Efficient Markets Hypothesis)市場價格皆完整反映交易標的之訊息，且無資訊不對稱之情形發生，且本次競賽標的台指期(TXF)所在之市場台灣，為金融系統完善之市場，故本組假設市場基本上服從常態分布，並以此做策略發想之基礎

#### 逆勢
以移動平均線(Simple Moving Average)來做策略發想，SMA上下各抓兩條線，期望在常態下的狀況抓到約PR值95左右之高低點，以此高低點做逆勢之買低賣高，抓對進出場獲利

抓完高低點後，交易次數明顯不足，於是我們做了第二次逆勢交易，也就是在最內兩條線中，一樣做買低賣高，只是不用夾的，而是用簡單的近日高低點來跟當日收盤價做比較，高於高點就賣，低於低點就買，增加套利次數

這麼做的好處是，由於此次競賽規則每日僅能進場一次，故只抓高低點進出場次數是不夠的，於是才用此策補足

#### 順勢
那如果不是基本上的情況呢?那就做順勢，以最外那兩條移動平均線當作預測股價收盤之信賴區間，一旦突破區間濾網我們就做順勢買高追高、買低追低的策略，追著牛市的勢頭增加獲利

前面做逆勢有提到
交易次數不足會是此次競賽的敗點
必須增加進出場才能不斷套利結算

#### 增加交易次數
但光做順逆勢還是不夠

於是我們又發想了新的方式，便是在移動平均線附近做進出場套利:
斜率(rocCalc)>0且在MA之上→出場
斜率(rocCalc)<0且在MA之下→進場

補足規則缺點

#### 參數設定
<img width="400" height="450" src=https://i.imgur.com/ZgKGfq0.png>

#### 績效
![image](https://i.imgur.com/nAblI93.png)
![image](https://i.imgur.com/PYi9OLM.png)
![image](https://i.imgur.com/7IRUolK.png)
![image](https://i.imgur.com/SfFRI7h.png)
![image](https://i.imgur.com/A8EcpB3.png)
![image](https://i.imgur.com/zsN8dF8.png)
