# TMBA X 日盛讀書會-第一組筆記

## 2019/9/24-線上開會
* 策略基礎元素(提供基礎開發架構，可以再看怎麼擴充)：
  1. 商品屬性(偏順勢or逆勢，可以套內建策略簡單觀察)
  2. 濾網(縮減資料，減少交易次數)
  3. 進出場點位(以前都是用市價進單，改掛指定價位某種程度也在減少過多的交易，同時期望提高獲利機會)

  以下舉例(ORB-進場)：
```
{condition1為濾網}
condition1 = time >= 0930 and time <= 1500 and 
	      EntriesToday(date) = 0 and
	      (high_band - low_band) > threshold and (high_band - low_band) < 300;

{下stop單於指定進場點位}
if condition1 then begin
	buy ("main_long_in") next bar at high_band stop;
	sellshort ("main_short_in") next bar at low_band stop;
end;

```
* 透過調整參數觀察資料，例如：開盤跳空多少點(參數)以上的獲利分布概況如何，但須先給定出場條件。
* 進場主邏輯決定後出場可嘗試方向：
  * 進場多久後沒賺到錢或是虧錢多少以上就出場。
  * 獲利達一定門檻後，若開始回吐獲利，則掛指定價位單準備出場，例如：多單賺100點後，回吐20%獲利則sell next bar at lowest(low, 10) stop，採用指定點位好處在於不會像market單一樣下一根直接出掉，如果沒有續跌則可以繼續抱單(因為不會觸發stop單)，如果跌了也還保有一定程度獲利。
* 如何定義波動→高低點、ATR...
  (可以嘗試把波動本身(或許取絕對值)拿去套布林通道、凱勒通道(中線上下加減ATR)判斷波動是否有放大，當成濾網)

* 可以收集(認識)不同屬性的指標例如：
  * 判斷趨勢強度ex. DMI
  * 波動程度ex. ATR
  * 方向性ex. KD, RSI
  ...

* 有興趣可以看一下價差濾網，正價差過大變做空、逆價差過大變做多，屬逆勢邏輯，期望期貨與現貨差距將收斂(期貨往現貨收斂)。

# A50
## 逆勢跳空(妤綺)


### 商品屬性
* 觀察開盤後9:00-11:00會看到當天最大的價格波動
* 用跳空就進場的策略測試，逆勢策略有較多獲利機會

### 交易邏輯
* 當今日開盤和昨收價格有跳空情形時進場，並採逆勢進場
* 當gap往上跳 >> 做空
* 當gap往下跳 >> 做多
* 設置停損停利，超過11點就平倉出場
* 使用1分K

### 待改善
* 2017交易次數太少
* 當盤勢轉為跳空順勢的停損方式
* 定義突破的K棒長度要>n值才算強勢行情


```
condition1 = high > upBB ;
condition2 = low < downBB ; 

if t<t[1] then begin
	if countif(condition1,20)>5 and marketposition<0 then buytocover next bar at market;
	if countif(condition2,20)>5 and marketposition>0 then sell next bar at market;
	end;
```
### GAP_03
![](https://i.imgur.com/EhJumh7.png)
![](https://i.imgur.com/FBy49oY.png)

```
inputs: gap(0.001),N(0.1);
//order
if t<t[1] and marketposition=0 then begin
	if (Low-high[1]) > CloseD(1)*gap then
	sellshort("GapUpSELL") next bar at market ;
end;

if t<t[1] and marketposition=0 then begin
	if (low[1]-High) > CloseD(1)*gap then
	buy ( "GapDnBUY" ) next bar at market ;
end;

//exit
if marketposition>0  then begin
	if c > entryprice+300 then sell next bar at market;
end;

if marketposition<0 then begin
	if c < entryprice-300 then buytocover next bar at market;
end;

//stoploss
if marketposition>0  then begin
	if high < entryprice*0.99 then sell("##") next bar at market;
end;

if marketposition<0 then begin
	if low>entryprice*1.01 then buytocover("###") next bar at market;
end;

//cover

if time >=1500 and marketposition <>0 then begin
	sell next bar at market;
	buytocover next bar market;
end;

```
### GAP_04

```
inputs: gap(0.001),N(350),len(10),shortlen(5),NumDn(2);
vars:avg(0),upBB(0),downBB(0),k(0);

avg= Average(c,len);
upBB= BollingerBand(c,len,NumDn);
downBB= BollingerBand(c,len,-NumDn);


//order
if time<=0901 and marketposition=0 then begin
	if (Low-high[1]) > c[1]*gap then
	sellshort("GapUpSELL") next bar at market ;
end;

if time<=0901 and marketposition=0 then begin
	if (low[1]-High) > c[1]*gap then
	buy ( "GapDnBUY" ) next bar at market ;
end;

//exit
if marketposition>0 then begin
	if low>entryprice+N then sell("exit1") next bar at highest(low,10) stop ;
end;

if marketposition<0 then begin
	if high<entryprice-N then buytocover("exit2") next bar at lowest(high ,10) stop;
end;

//stop

condition1 = c > high[1] ;
condition2 = c < low[1] ; 

if t<t[1] then k=1 else k=k+1 ;
if k= 20 then begin
	if countif(condition1,20)>10 and marketposition<0 then buytocover("st11111") next bar at market;
	if countif(condition2,20)>10 and marketposition>0 then sell("st2222") next bar at market;

end;

//cover
if time >=1100 and marketposition <>0 then begin
sell next bar at market;
buytocover next  bar market;
end;


```

### GAP_06

```
inputs: gap(0.005),N(0.01),len(10),shortlen(5),NumDn(1);
vars:avg(0),upBB(0),downBB(0),k(0);

//order
condition3 = c < o ;
condition4 = c > o ;

if t<t[1] then begin
	condition1 = (low-high[1]) > CloseD(1)*gap ;
	condition2 = (low[1]-High) > CloseD(1)*gap ;
	end;

if t<t[1] then k=1 else k=k+1 ;
if k= 1 then begin
	if countif(condition3,20)>1 and condition2 then buy next bar at market;
	if countif(condition4,20)>1 and condition1 then sellshort next bar at market;
end;

//exit

condition5 = high <  CloseD(1) ;
condition6 = low >  CloseD(1) ;

if marketposition>0 and condition6 then sell next bar at market ;
if marketposition<0 and condition5 then buytocover next bar at market ;
   
//stoploss

if marketposition>0  then begin
	if high < entryprice*0.99 then sell("##") next bar at market;
end;

if marketposition<0 then begin
	if low > entryprice*1.01 then buytocover("###") next bar at market;
end;

//cover
if time >=1100 and marketposition <>0 then begin
	sell next bar at market;
	buytocover next bar market;
end;


```
### GAP_MACD
```
inputs: gap(0.001),N(0.01),v1(10),v2(-10),FastLength( 12 ), SlowLength( 26 ), MACDLength( 9 ),MACDD(40);
var: var0( 0 ), var1( 0 ), var2( 0 ),k(0);

var0 = MACD( Close, FastLength, SlowLength ) ; //
var1 = XAverage( var0, MACDLength ) ; //
var2 = var0 - var1 ;

//order

condition1 = OpenD(1)<CloseD(1) ;
condition2 = OpenD(1)>CloseD(1) ; 
condition3 = var0 < -MACDD ;
condition4 = var0 > MACDD ;

if t<t[1] then begin
	condition1 = (low-high[1]) > CloseD(1)*gap ;
	condition2 = (low[1]-High) > CloseD(1)*gap ;
	end;

if condition2 then begin
	if t<t[1] then k=1 else k=k+1 ;
	if k< 10 and countif(condition4,10)>1 then buy next bar at market;
	end;
if condition1 then begin
	if t<t[1] then k=1 else k=k+1 ;
	if k< 10 and countif(condition3,10)>1 then sellshort next bar at market;
	end;

condition1 = value2>2 ;
condition2 = value2<-2 ;

if marketposition>0 and value1 cross under value2 then
sell next bar at market;

if marketposition<0 and value1 cross over value2 then
buytocovernext bar at market;


//cover
if time >=1130 and marketposition <>0 then begin
	sell next bar at market;
	buytocover next bar market;
end;  


```

## 順勢(丞剛) 

#### 交易策略
* 使用5分鐘K
* 以每日開盤前30鐘走勢為基礎
* 以走勢作為0930進場的方向依據，若走多則以stop進多單，反之則空單，無明顯方向則不進場

### 版本1

#### <程式碼>
```
//5 min

{entry}
if time=0930 and Close>OpenD(0) and CountIf(Close>OpenD(0),5)>=3 then
	buy next bar at Highest(high,5) stop;
	
if time=0930 and Close<OpenD(0) and CountIf(Close<OpenD(0),5)>=3 then
	sellshort next bar at Lowest(low,5) stop;


{stop loss & take profit}
If marketposition=1 then begin
	if c>entryprice*1.01 then 
		sell next bar at lowest(low,5) stop; //TP
	if c>entryprice*1.02 then 
		sell next bar at market; //TP
	if c<entryprice*0.98 then
		sell next bar at market; //SL
end;

If marketposition=-1 then begin
	if c<entryprice*0.99 then 
		buytocover next bar at highest(high,5) stop; //TP
	if c<entryprice*0.98 then 
		buytocover next bar at market; //TP
	if c>entryprice*1.02 then
		buytocover next bar at market; //SL
end;

{market close exit}
if marketposition<>0 and time>=1455 then begin
 	sell next bar at market;
 	buytocover next bar at market;
end;
```

#### <樣本內>
![](https://i.imgur.com/AUF3PSv.png)

![](https://i.imgur.com/h2dyavZ.png)

![](https://i.imgur.com/3k22fMf.png)

![](https://i.imgur.com/rjJKEip.png)

![](https://i.imgur.com/4NVQaMy.png)

#### <樣本外>
![](https://i.imgur.com/ianDzUE.png)

![](https://i.imgur.com/PXym7uK.png)

![](https://i.imgur.com/4FBs9nP.png)

![](https://i.imgur.com/ARodMwL.png)

![](https://i.imgur.com/n1yPuyw.png)

### 版本2

#### <程式碼>
```
//5 min

{entry}
if time=0930 and Close>OpenD(0) and CountIf(Close>OpenD(0),5)>=3 then
	buy next bar at Highest(high,5) stop;

if time=0930 and Close<OpenD(0) and CountIf(Close<OpenD(0),5)>=3 then
	sellshort next bar at Lowest(low,5) stop;

{stop loss & take profit}
if marketposition>0 then begin
	if c>entryprice*1.01 then
		sell("Take profit_buy") next bar at lowest(low,5) stop; //TP
	if c>entryprice*1.02 then
		sell("Take profit_buy#") next bar at market; //TP
	if c<entryprice*0.98 then
		sell("Stoploss_buy") next bar at low stop; //SL
end;

if marketposition<0 then begin
	if c<entryprice*0.99 then
		buytocover("Take profit_sell") next bar at highest(high,5) stop; //TP
	if c<entryprice*0.98 then
		buytocover("Take profit_sell#") next bar at market; //TP
	if c>entryprice*1.02 then
		buytocover("Stoploss_sell") next bar at low stop; //SL
end;

{market close exit}
if marketposition<>0 and time>=1455 then begin
	sell("market close exit_buy") next bar at market;
	buytocover("market close exit_sell") next bar at market;
end;
```

<樣本內>
![](https://i.imgur.com/fqtFffA.png)

![](https://i.imgur.com/h91paxo.png)

![](https://i.imgur.com/0RJsUT6.png)

![](https://i.imgur.com/wQSwHtB.png)

![](https://i.imgur.com/DZU21Fk.png)

<樣本外>
![](https://i.imgur.com/k49NE0L.png)

![](https://i.imgur.com/xSc4gJA.png)

![](https://i.imgur.com/k3nmzZR.png)

![](https://i.imgur.com/cJ70CQg.png)

![](https://i.imgur.com/8cahgna.png)


## 仲廷-20191010(60minK)
### Source code
INPUTS: LEN1(1),LEN2(1) ,X(3);
VARS: SLOPE(0);

//Value
IF DATE[0]<>DATE[1] THEN BEGIN	
	VALUE1 = HIGH[LEN1];
	VALUE2 = LOW[LEN1];
END;
//Condition

SLOPE = C - C[LEN2];
CONDITION1 = C >= VALUE1;
CONDITION2 = C <= VALUE2;
CONDITION3 = SLOPE > 0;
CONDITION4 = SLOPE < 0; 

IF TIME > 0900 AND TIME <1300 AND CONDITION1 AND CONDITION3 AND MARKETPOSITION <= 0 THEN BEGIN
	IF LOW > HIGH[1] THEN
	BUY ( "GapUpBUY" )NEXT BAR AT MARKET;
END;

IF TIME > 0900 AND TIME <1300 AND CONDITION2 AND CONDITION4 AND MARKETPOSITION >= 0 THEN BEGIN
	IF LOW[1] > HIGH THEN
	SELLSHORT("GapDnSELL") NEXT BAR AT MARKET;
END;

//Stop Loss
SETSTOPLOSS OF (100*X);

SETEXITONCLOSE;

### 樣本內 2015/01/01~2017/12/30
#### 總績效
![](https://i.imgur.com/EJgvwWO.png)
![](https://i.imgur.com/KodDYEg.png)
#### 總交易分析
![](https://i.imgur.com/q9H7EMX.png)
![](https://i.imgur.com/xxQCYCl.png)

### 樣本外 2018/01/01~2019/12/30
#### 總績效
![](https://i.imgur.com/njOgSs9.png)
![](https://i.imgur.com/TN5BCKG.png)
#### 總交易分析
![](https://i.imgur.com/6VcB28t.png)
![](https://i.imgur.com/pXNrBZB.png)

### 總樣本 2015/01/01~2019/12/30
#### 總績效
![](https://i.imgur.com/Zu1bFZT.png)
![](https://i.imgur.com/AHiTREe.png)
#### 總交易分析
![](https://i.imgur.com/F5PiwYN.png)
![](https://i.imgur.com/yg3V4Ew.png)

### 最佳化(不穩定)
![](https://i.imgur.com/Adveehz.png)

## 聖杯
### Source code
Inputs:I_will_making_much_money_HAHAHA(6);//Used on 60min of data K bar in Taiwan...
The buy an next on bar at h by stop;//Long Fighting!!
The sell at short an next on bar at l by stop;//Short Fighting!!
A setstoploss of (200*I_will_making_much_money_HAHAHA);//Stoploss of Money!!
A setexitonclose;
### 績效
![](https://i.imgur.com/eCuXAFs.png)
![](https://i.imgur.com/RroyxiB.png)
![](https://i.imgur.com/pTGWSrA.png)
![](https://i.imgur.com/Al1O583.png)
