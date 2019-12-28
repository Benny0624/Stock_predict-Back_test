# TMBA x JihSun Trading contest
## 2019.10.14(A50當沖比賽)

### 新華富時A50指數簡介:
* 基期 : 2003年7月21日；對外公開發佈 : 2004年1月
* 交易所 : 新加坡交易所（SGX）
* 跳動點 : 2.5點 = US$2.5
* 保證金 : US$935

A50指數是根據新華富時A股指數編製規則,從A股市場選擇合格的最大50家公司組成的指數。

其各項指標包括業績表現、流動性、波動性、行業分佈和市場代表性均處於市場領先水平。

A50指數占有A股市場流通市值的33.2％，新華富時中國A50指數與上證50指數高度相關，屬於競爭性指數。花旗集團以此指數建立了首個涵蓋中國A股、針對個人投資者的結構性產品，並即將向海外投資者公開發售。

* 含股比重(前十大):

招商銀行（9.0627％）;中國銀行（5.6025)
大秦鐵路（5.4295％）;中國石化（5.1959％）
寶鋼股份（5.1884％）;長江電力（5.0562％）
民生銀行（5.0302％）;深萬科A（4.8303％）
中國聯通（3.5077％）;貴州茅臺（3.0063％）
![](https://i.imgur.com/gDcKEix.png)

* 交易時間:

T 時段： 09 : 00 ~ 16 : 35
T+1時段：17 : 00 ~ 04 : 45
T+1時段的交易時間，正是國內開始發佈上市公司公告等信息的時段，這些信息將反映到T+1時段收盤價中，導致第二天滬深股市開盤受離岸市場的影響，是內地股市有成為“影子市場”的風險。

* 功能:

眾所周知，QFII((Qualified Foreign Institutional Investors,合格的境外機構投資者)投資中國A股，客觀上需要有避險工具用於對沖風險，因為其在發達國家市場投資時，早已熟悉並習慣於使用衍生品工具對手中的籌碼對沖風險。

### Trading data EDA:
* 日k: 
![](https://i.imgur.com/fgc6XGt.png)
![](https://i.imgur.com/vK49pc3.png)
![](https://i.imgur.com/628KWvK.png)
* 週k:
![](https://i.imgur.com/GDVTVgs.png)
![](https://i.imgur.com/1SlDXtD.png)
![](https://i.imgur.com/BmXHDVJ.png)
* 月k:
![](https://i.imgur.com/GBYoOsX.png)
![](https://i.imgur.com/WjeBTYm.png)
![](https://i.imgur.com/xNqUhzB.png)

三種k線皆顯示波動集中2015年(中國股災)，跳高跳低比率在日k就出現近50%的高頻率，以此作為跳空策略的發想及開端。


* 時間波動分析:

![](https://i.imgur.com/3Lg4n90.png)

T盤波動集中在早上9點到12點，以及下午2~3點，以此做為時間濾網的發想靈感來源;T+1盤波動較不明顯暫不納入交易區段。

## Gap_Strategy_20191010(60minK)
### 程式邏輯

計算昨日前LEN1根K之最高最低價以及當根K棒與前LEN2根K之差當作漲跌方向判斷，進出場如下:

* 進場
如果方向為漲(CONDITION3)，收盤價大於前LEN1根K之最高價(CONDITION1)之跳空則在市價買進
* 出場
如果方向為跌(CONDITION4)，收盤價小於前LEN1根K之最低價(CONDITION2)之跳空則在市價放空。
* 停損
100*X點停損出場

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
* 總績效
![](https://i.imgur.com/EJgvwWO.png)
![](https://i.imgur.com/KodDYEg.png)
* 總交易分析
![](https://i.imgur.com/q9H7EMX.png)
![](https://i.imgur.com/xxQCYCl.png)

### 樣本外 2018/01/01~2019/12/30
* 總績效
![](https://i.imgur.com/njOgSs9.png)
![](https://i.imgur.com/TN5BCKG.png)
* 總交易分析
![](https://i.imgur.com/6VcB28t.png)
![](https://i.imgur.com/pXNrBZB.png)

### 總樣本 2015/01/01~2019/12/30
* 總績效
![](https://i.imgur.com/Zu1bFZT.png)
![](https://i.imgur.com/AHiTREe.png)
* 總交易分析
![](https://i.imgur.com/F5PiwYN.png)
![](https://i.imgur.com/yg3V4Ew.png)

### 最佳化(不穩定)
![](https://i.imgur.com/Adveehz.png)

### 小結

本人第一支當沖策略，樣本內外獲利能力還不錯，並且在股災過後還是持續獲利，第一次寫的水準來說應該是可圈可點，甚至在最佳化後勝率都超過50%，很滿意，但參數最佳化不穩定算是下次可以改進的缺點之一。
