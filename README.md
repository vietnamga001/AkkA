// Cài đặt rủi ro
input double RiskPerTrade = 1.0; // % vốn mỗi lệnh
input double DailyMaxLoss = 5.0; // % thua lỗ hàng ngày
input double TotalMaxLoss = 20.0; // % thua lỗ tổng

double CalculateLotSize() {
    double tickValue = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_VALUE);
    double riskAmount = AccountInfoDouble(ACCOUNT_BALANCE) * RiskPerTrade / 100;
    return NormalizeDouble(riskAmount / (StopLoss * tickValue), 2);
}

//+------------------------------------------------------------------+
//|                                                      XAU_Bot.mq5 |
//|                                   Copyright 2025, YourCompany    |
//|                                        https://www.yourwebsite.com |
//+------------------------------------------------------------------+
#property copyright "Copyright 2025"
#property version   "1.00"

// Input parameters
input int      MAPeriod1 = 50;
input int      MAPeriod2 = 200;
input int      RSIPeriod = 14;
input double   RiskRewardRatio = 2.0;

// Global variables
int maHandle1, maHandle2, rsiHandle;

//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit() {
    maHandle1 = iMA(_Symbol, PERIOD_M1, MAPeriod1, 0, MODE_SMA, PRICE_CLOSE);
    maHandle2 = iMA(_Symbol, PERIOD_M1, MAPeriod2, 0, MODE_SMA, PRICE_CLOSE);
    rsiHandle = iRSI(_Symbol, PERIOD_M1, RSIPeriod, PRICE_CLOSE);
    return(INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
void OnTick() {
    double ma1[2], ma2[2], rsi[1];
    CopyBuffer(maHandle1, 0, 0, 2, ma1);
    CopyBuffer(maHandle2, 0, 0, 2, ma2);
    CopyBuffer(rsiHandle, 0, 0, 1, rsi);
    
    // Tín hiệu mua
    if(ma1[1] > ma2[1] && rsi[0] < 30) {
        ExecuteTrade(ORDER_TYPE_BUY);
    }
    
    // Tín hiệu bán
    if(ma1[1] < ma2[1] && rsi[0] > 70) {
        ExecuteTrade(ORDER_TYPE_SELL);
    }
}

//+------------------------------------------------------------------+
//| Thực hiện lệnh                                                   |
//+------------------------------------------------------------------+
void ExecuteTrade(ENUM_ORDER_TYPE orderType) {
    double sl = CalculateStopLoss(orderType);
    double tp = sl * RiskRewardRatio;
    double lotSize = CalculateLotSize(sl);
    
    MqlTradeRequest request = {0};
    MqlTradeResult result = {0};
    
    request.action = TRADE_ACTION_DEAL;
    request.symbol = _Symbol;
    request.volume = lotSize;
    request.type = orderType;
    request.price = SymbolInfoDouble(_Symbol, orderType==ORDER_TYPE_BUY ? SYMBOL_ASK : SYMBOL_BID);
    request.sl = sl;
    request.tp = tp;
    
    OrderSend(request, result);
}