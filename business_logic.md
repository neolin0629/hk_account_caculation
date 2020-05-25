# 港股产品业绩计算

一共分为持仓和交易两部分进行计算，注意点：

1. 需要计算transaction fee和financial fee。
2. Cubist和Prelude没有总资产概念，需要按定义的公示维护一个名义总资产，作为计算return(%)的分母

## 交易

**一些定义**

* 佣金(commission)：经纪商收的交易费用
* 印花税(stamp duty tax)：卖出时收10bps
* 跨市场通道费(market charge)：QFII 0.887 bps，stock connect 1.087 bps
* 成交价格(gross price)：实际成交价格
* 成交净价(net price)：扣费后的成交价格
* 结算价格(settle gross price)：美元计价的实际成交价格
* 费率(fx)：QFII用的是USDCNY，Stock connect用的是USDCNH 

**stock计算逻辑**

trd_pnl_usd_s =[ (close price - gross price) * quantity - fees] / fx

fees = (commission + market charge + side == sell ? 10bps : 0bps)  * (gross price * quantity)

> quantity 带正负号区分buy/sell
>
> fx 根据是QFII或者SC，分别使用usdcny/usdcnh

**index swap计算逻辑**

trd_pnl_usd_i =[ (close price - gross price) * quantity - (comms + market charges) * (gross price * quantity)] / fx

## 持仓

**一些定义**

* 持仓成本(Notional)：根据实际成交价格记录的成本，如果标的有反复的买卖，需要每笔计算
* 可用的利息利率(cash rate)：未使用的钱的每日利息
* 费率(fx)：QFII用的是USDCNY，SC用的是USDCNH 
* 持仓费用利息(Pos rate)：如果是swap

**Long position计算逻辑**

pos_pnl_usd_l = (Local price move * quantity - fees) / fx

fees = T day Notional * T-1 rate / 365

> Long position因为是swap，所以需要交费，rate = benchmark + spread

**Short side 计算逻辑**

pos_pnl_usd_s = (Local price move * quantity - fees) / fx

fees = Notional * [rate => 交易时确认的空头利息 - (benchmark + spread)] / 365

> CICC是Fixed rate，无扣减

汇率变化产生的收益也会体现在计算内。

## 名义总资产

Total asset = T-1 day position market value + T day trade net

T day trade net = T day (buy - sell) > 0 ? T day (buy - sell) : 0

## 可用利息

Argo账户的可用需要计息。

> 需要解决哪里取的问题？enfusion?

## 待确认问题

1. which is the correct way to caculate Notional? 
   1. Close to market Amount (RMB) / T day FX.
   2. SUM of each settle amount(USD) which means we use different FX depends on the trading day.

 