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
   double MACD = iMACD(NULL,0,12,26,9,PRICE_CLOSE,MODE_MAIN,0);
   double Signal = iMACD(NULL,0,12,26,9,PRICE_CLOSE,MODE_SIGNAL,0);
   double Histogram = MACD - Signal;
   
   // check for buy signal
   if (Bid > EMA200 && Bid > VWAP && Histogram > 0) {
      // open a buy order
      int ticket = OrderSend(_Symbol,OP_BUY,Lots,Bid,3,Bid-StopLoss*Point,Bid+TakeProfit*Point,"Buy order",MagicNumber,0,Green);
      if (ticket>0) {
         Print("Buy order opened with ticket #",ticket);
      } else {
         Print("Error opening buy order with error code #",GetLastError());
      }
   }
   
   // check for sell signal
   if (Bid < EMA200 && Bid < VWAP && Histogram < 0) {
      // open a sell order
      int ticket = OrderSend(_Symbol,OP_SELL,Lots,Bid,3,Bid+StopLoss*Point,Bid-TakeProfit*Point,"Sell order",MagicNumber,0,Red);
      if (ticket>0) {
         Print("Sell order opened with ticket #",ticket);
      } else {
         Print("Error opening sell order with error code #",GetLastError());
      }
   }
}

