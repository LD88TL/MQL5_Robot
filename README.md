//+------------------------------------------------------------------+
//| EA template program                                             |
//| Copyright 2021, ChatGPT.                                       |
//| https://github.com/chatgpt                                      |
//+------------------------------------------------------------------+
#property copyright "2021, ChatGPT."
#property link      "https://github.com/chatgpt"
#property version   "1.00"
#property strict
//--- input parameters
input double  Lots=0.1;          // volume of the trade
input int     StopLoss=50;       // Stop Loss in pips
input int     TakeProfit=100;    // Take Profit in pips
input double  TrailingStop=30;   // Trailing Stop in pips
input int     MagicNumber=12345; // Magic number to identify trades
//+------------------------------------------------------------------+
//| expert initialization function                                  |
//+------------------------------------------------------------------+
int OnInit()
  {
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| expert deinitialization function                                |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
  }
//+------------------------------------------------------------------+
//| expert start function                                            |
//+------------------------------------------------------------------+
void OnTick()
  {
   // get current bid price
   double Bid = SymbolInfoDouble(_Symbol,SYMBOL_BID);
   // calculate the 200-period exponential moving average
   double EMA200 = iMA(NULL,0,200,0,MODE_EMA,PRICE_CLOSE,0);
   // calculate the VWAP for the last 30 periods
   double VWAP = iCustom(NULL,0,"Volume Weighted Average Price",30,0);
   // calculate the MACD histogram
