//+------------------------------------------------------------------+
//|                                                   TimeisLife.mq5 |
//|                   Copyright 2009-2017, MetaQuotes Software Corp. |
//|                                              http://www.mql5.com |
//+------------------------------------------------------------------+
#property copyright "2009-2017, MetaQuotes Software Corp."
#property link      "http://www.mql5.com"
#property version   "1.00"

#include <Trade/Trade.mqh>
#include <Arrays/Array.mqh>

input ENUM_TIMEFRAMES TimeFrame = PERIOD_H1;
input int EMA_Period = 14;
input int SuperTrend_Period = 7;
input double SuperTrend_Multiplier = 3.0;

// Variáveis globais
CIndicators Indicators;
CArrayObj ArrayIndicators;

//+------------------------------------------------------------------+
//| Função de inicialização do robô                                  |
//+------------------------------------------------------------------+
int OnInit()
{
    ArrayIndicators.Create(3);

    ArrayIndicators.Add(Indicators.EMA(NULL, 0, EMA_Period, PRICE_CLOSE, MODE_EMA, TimeFrame));
    ArrayIndicators.Add(Indicators.VWAP(NULL, 0, TimeFrame));
    ArrayIndicators.Add(Indicators.SuperTrend(NULL, 0, SuperTrend_Period, SuperTrend_Multiplier, PRICE_CLOSE, MODE_MAIN, TimeFrame));

    return(INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Função de finalização do robô                                    |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
{
    ArrayIndicators.DeleteAll();
}

//+------------------------------------------------------------------+
//| Função de atualização do robô                                    |
//+------------------------------------------------------------------+
void OnTick()
{
    // Obtenha os valores dos indicadores
    double EMA_Value = Indicators.iMA(NULL, 0, EMA_Period, 0, MODE_EMA, PRICE_CLOSE, TimeFrame);
    double VWAP_Value = Indicators.iVWAP(NULL, 0, TimeFrame);
    double SuperTrend_Value = Indicators.iSuperTrend(NULL, 0, SuperTrend_Period, SuperTrend_Multiplier, PRICE_CLOSE, MODE_MAIN, TimeFrame);
    int SuperTrend_Color = Indicators.iSuperTrend(NULL, 0, SuperTrend_Period, SuperTrend_Multiplier, PRICE_CLOSE, MODE_UPPER, TimeFrame) ? clrLimeGreen : clrDeepPink;

    // Verifique o sinal de compra
    if (Bid > EMA_Value && Bid > VWAP_Value && (Bid > SuperTrend_Value || SuperTrend_Color == clrLimeGreen))
    {
        CTrade trade;
        if (trade.Buy(Symbol(), Volume))
        {
            Print("Compra realizada com sucesso. Preço: ", Bid);
        }
    }

    // Verifique o sinal de venda
    if (Bid < EMA_Value && Bid < VWAP_Value && (Bid < SuperTrend_Value || SuperTrend_Color == clrDeepPink))
    {
        CTrade trade;
        if (trade.Sell(Symbol(), Volume))
        {
            Print("Venda realizada com sucesso. Preço: ", Bid);
        }
    }
}
``
//+------------------------------------------------------------------+
//| Indicador VWAP                                                   |
//+------------------------------------------------------------------+

#property copyright "TimeIsLife, 2023"
#property link ""
//---- número da versão do indicador
#property version   "1.00"
//---- desenhando o indicador na janela principal
#property indicator_chart_window 
//---- um buffer é usado para calcular e renderizar o indicador
#property indicator_buffers 1
//---- um edifício gráfico é usado
#property indicator_plots   1
//+----------------------------------------------+
//|  Parâmetros de renderização do indicador          |
//+----------------------------------------------+
//---- desenhando o indicador 1 como uma linha
#property indicator_type1   DRAW_LINE
//---- A cor DarkOrchid é usada como a cor da linha indicadora normal
#property indicator_color1  clrDarkOrchid
//---- linha indicadora 1 - curva contínua
#property indicator_style1  STYLE_SOLID
//---- a espessura da linha do indicador 1 é igual a 3
#property indicator_width1  3
//---- exibindo o rótulo do indicador
#property indicator_label1  "VWAP"
//+----------------------------------------------+
//| Parâmetros de entrada do indicador        |
//+----------------------------------------------+
input uint n=30;
input ENUM_APPLIED_VOLUME VolumeType=VOLUME_TICK;
input int Shift=0; //movendo o indicador horizontal em barras
//+----------------------------------------------+
//---- Declarando arrays dinâmicos que estarão em 
// mais usado como buffers de indicador
double IndBuffer[];
//---- Declarando variáveis ​​inteiras para o início da amostragem de dados
int min_rates_total, size;
//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+  
void OnInit()
  {
//---- Inicialização da variável início da amostragem de dados
   min_rates_total = 30;
//---- Convertendo um array dinâmico em um buffer de indicador
   SetIndexBuffer(0,IndBuffer,INDICATOR_DATA);
//---- Deslocando o indicador 1 horizontalmente para Shift
   PlotIndexSetInteger(0,PLOT_SHIFT,Shift);
//---- Deslocando o início da renderização do indicador 1 para min_rates_total
   PlotIndexSetInteger(0,PLOT_DRAW_BEGIN,min_rates_total);
//---- Indexando os elementos no buffer como na série temporal
   ArraySetAsSeries(IndBuffer,true);
//--- criar um nome para ser exibido em uma subjanela separada e em uma dica pop-up
   IndicatorSetString(INDICATOR_SHORTNAME,"VWAP");
//--- Determinando a precisão da exibição dos valores do indicador
   IndicatorSetInteger(INDICATOR_DIGITS,_Digits);
//----
  }
//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(
                const int rates_total,    // quantidade de histórico em barras no tick atual
                const int prev_calculated,// quantidade de histórico em barras no tick anterior
                const datetime &time[],
                const double &open[],
                const double& high[],     // matriz de preço de máximas de preço para calcular o indicador
                const double& low[],      // matriz de preço de mínimos de preço para calcular o indicador
                const double &close[],
                const long &tick_volume[],
                const long &volume[],
                const int &spread[]
                )
  {
//---- verificando o número de barras para suficiência para cálculo
   if(rates_total<min_rates_total) return(0);

//---- nã consegui traduzir
   long vol,sum2;
   int limit,bar;
   double sum1;

//---- Cálculo do limite de número inicial para o ciclo de conversão de barras
   if(prev_calculated>rates_total || prev_calculated<=0)// verifique o primeiro início do cálculo do indicador
      limit=rates_total-min_rates_total-1; // número inicial para calcular todas as barras
   else limit=rates_total-prev_calculated; // número inicial para calcular novas barras

//---- indexando elementos em arrays como em séries temporais
   if(VolumeType==VOLUME_TICK) ArraySetAsSeries(tick_volume,true);
   else ArraySetAsSeries(volume,true);
   ArraySetAsSeries(close,true);

//---- ciclo principal de cálculo do indicador
   for(bar=limit; bar>=0 && !IsStopped(); bar--)
     {
      sum1=0;
      sum2=0;
      for(int ntmp=0; ntmp<int(n); ntmp++)
        {
         if(VolumeType==VOLUME_TICK) vol=long(tick_volume[bar+ntmp]);
         else vol=long(volume[bar+ntmp]);
         sum1+=close[bar+ntmp]*vol;
         sum2+=vol;
        }

      if(sum2) IndBuffer[bar]=sum1/sum2;
      else IndBuffer[bar]=IndBuffer[bar+1];
     }
//----     
   return(rates_total);
  }
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Indicador EMA                                                |
//+------------------------------------------------------------------+
#property copyright "2009-2020, MetaQuotes Software Corp."
#property link      "http://www.mql5.com"

//--- indicator settings
#property indicator_chart_window
#property indicator_buffers 1
#property indicator_plots   1
#property indicator_type1   DRAW_LINE
#property indicator_color1  Red
//--- input parameters
input int            InpMAPeriod=200;         // Period
input int            InpMAShift=0;           // Shift
input ENUM_MA_METHOD InpMAMethod=MODE_EMA;  // Method
//--- indicator buffer
double ExtLineBuffer[];
//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
void OnInit()
  {
//--- indicator buffers mapping
   SetIndexBuffer(0,ExtLineBuffer,INDICATOR_DATA);
//--- set accuracy
   IndicatorSetInteger(INDICATOR_DIGITS,_Digits+1);
//--- set first bar from what index will be drawn
   PlotIndexSetInteger(0,PLOT_DRAW_BEGIN,InpMAPeriod);
//--- line shifts when drawing
   PlotIndexSetInteger(0,PLOT_SHIFT,InpMAShift);
//--- name for DataWindow
   string short_name;
   switch(InpMAMethod)
     {
      case MODE_EMA :
         short_name="EMA";
         break;
      case MODE_LWMA :
         short_name="LWMA";
         break;
      case MODE_SMA :
         short_name="SMA";
         break;
      case MODE_SMMA :
         short_name="SMMA";
         break;
      default :
         short_name="unknown ma";
     }
   IndicatorSetString(INDICATOR_SHORTNAME,short_name+"("+string(InpMAPeriod)+")");
//--- set drawing line empty value
   PlotIndexSetDouble(0,PLOT_EMPTY_VALUE,0.0);
  }
//+------------------------------------------------------------------+
//|  Moving Average                                                  |
//+------------------------------------------------------------------+
int OnCalculate(const int rates_total,
                const int prev_calculated,
                const int begin,
                const double &price[])
  {
   if(rates_total<InpMAPeriod-1+begin)
      return(0);
//--- first calculation or number of bars was changed
   if(prev_calculated==0)
     {
      ArrayInitialize(ExtLineBuffer,0);
      PlotIndexSetInteger(0,PLOT_DRAW_BEGIN,InpMAPeriod-1+begin);
     }
//--- calculation
   switch(InpMAMethod)
     {
      case MODE_EMA:
         CalculateEMA(rates_total,prev_calculated,begin,price);
         break;
      case MODE_LWMA:
         CalculateLWMA(rates_total,prev_calculated,begin,price);
         break;
      case MODE_SMMA:
         CalculateSmoothedMA(rates_total,prev_calculated,begin,price);
         break;
      case MODE_SMA:
         CalculateSimpleMA(rates_total,prev_calculated,begin,price);
         break;
     }
//--- return value of prev_calculated for next call
   return(rates_total);
  }
//+------------------------------------------------------------------+
//|   simple moving average                                          |
//+------------------------------------------------------------------+
void CalculateSimpleMA(int rates_total,int prev_calculated,int begin,const double &price[])
  {
   int i,start;
//--- first calculation or number of bars was changed
   if(prev_calculated==0)
     {
      start=InpMAPeriod+begin;
      //--- set empty value for first start bars
      for(i=0; i<start-1; i++)
         ExtLineBuffer[i]=0.0;
      //--- calculate first visible value
      double first_value=0;
      for(i=begin; i<start; i++)
         first_value+=price[i];
      first_value/=InpMAPeriod;
      ExtLineBuffer[start-1]=first_value;
     }
   else
      start=prev_calculated-1;
//--- main loop
   for(i=start; i<rates_total && !IsStopped(); i++)
      ExtLineBuffer[i]=ExtLineBuffer[i-1]+(price[i]-price[i-InpMAPeriod])/InpMAPeriod;
  }
//+------------------------------------------------------------------+
//|  exponential moving average                                      |
//+------------------------------------------------------------------+
void CalculateEMA(int rates_total,int prev_calculated,int begin,const double &price[])
  {
   int    i,start;
   double SmoothFactor=2.0/(1.0+InpMAPeriod);
//--- first calculation or number of bars was changed
   if(prev_calculated==0)
     {
      start=InpMAPeriod+begin;
      ExtLineBuffer[begin]=price[begin];
      for(i=begin+1; i<start; i++)
         ExtLineBuffer[i]=price[i]*SmoothFactor+ExtLineBuffer[i-1]*(1.0-SmoothFactor);
     }
   else
      start=prev_calculated-1;
//--- main loop
   for(i=start; i<rates_total && !IsStopped(); i++)
      ExtLineBuffer[i]=price[i]*SmoothFactor+ExtLineBuffer[i-1]*(1.0-SmoothFactor);
  }
//+------------------------------------------------------------------+
//|  linear weighted moving average                                  |
//+------------------------------------------------------------------+
void CalculateLWMA(int rates_total,int prev_calculated,int begin,const double &price[])
  {
   int    weight=0;
   int    i,l,start;
   double sum=0.0,lsum=0.0;
//--- first calculation or number of bars was changed
   if(prev_calculated<=InpMAPeriod+begin+2)
     {
      start=InpMAPeriod+begin;
      //--- set empty value for first start bars
      for(i=0; i<start; i++)
         ExtLineBuffer[i]=0.0;
     }
   else
      start=prev_calculated-1;

   for(i=start-InpMAPeriod,l=1; i<start; i++,l++)
     {
      sum   +=price[i]*l;
      lsum  +=price[i];
      weight+=l;
     }
   ExtLineBuffer[start-1]=sum/weight;
//--- main loop
   for(i=start; i<rates_total && !IsStopped(); i++)
     {
      sum             =sum-lsum+price[i]*InpMAPeriod;
      lsum            =lsum-price[i-InpMAPeriod]+price[i];
      ExtLineBuffer[i]=sum/weight;
     }
  }
//+------------------------------------------------------------------+
//|  smoothed moving average                                         |
//+------------------------------------------------------------------+
void CalculateSmoothedMA(int rates_total,int prev_calculated,int begin,const double &price[])
  {
   int i,start;
//--- first calculation or number of bars was changed
   if(prev_calculated==0)
     {
      start=InpMAPeriod+begin;
      //--- set empty value for first start bars
      for(i=0; i<start-1; i++)
         ExtLineBuffer[i]=0.0;
      //--- calculate first visible value
      double first_value=0;
      for(i=begin; i<start; i++)
         first_value+=price[i];
      first_value/=InpMAPeriod;
      ExtLineBuffer[start-1]=first_value;
     }
   else
      start=prev_calculated-1;
//--- main loop
   for(i=start; i<rates_total && !IsStopped(); i++)
      ExtLineBuffer[i]=(ExtLineBuffer[i-1]*(InpMAPeriod-1)+price[i])/InpMAPeriod;
  }
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Indicador SuperTrend                                                |
//+------------------------------------------------------------------+
//------------------------------------------------------------------
#property copyright   "TimeisLife, 2023"
#property link        ""
#property description "SuperTrend"
//------------------------------------------------------------------
#property indicator_chart_window
#property indicator_buffers 4
#property indicator_plots   1
#property indicator_label1  "Super trend"
#property indicator_type1   DRAW_COLOR_LINE
#property indicator_color1  clrDarkGray,clrDeepPink,clrLimeGreen
#property indicator_width1  2
//--- input parameters
input int                inpPeriod    = 10;            // Cci Period
input ENUM_APPLIED_PRICE inpPrice     = PRICE_TYPICAL; // Cci Price
input int                inpAtrPeriod = 2;             // Atr period
//--- indicator buffers
double val[],valc[],prices[],trend[];
//+------------------------------------------------------------------+ 
//| Custom indicator initialization function                         | 
//+------------------------------------------------------------------+ 
int OnInit()
  {
//--- indicator buffers mapping
   SetIndexBuffer(0,val,INDICATOR_DATA);
   SetIndexBuffer(1,valc,INDICATOR_COLOR_INDEX);
   SetIndexBuffer(2,prices,INDICATOR_CALCULATIONS);
   SetIndexBuffer(3,trend,INDICATOR_CALCULATIONS);
//--- indicator short name assignment
   IndicatorSetString(INDICATOR_SHORTNAME,"Super trend ("+(string)inpPeriod+")");
//---
   return (INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Custom indicator de-initialization function                      |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
  }
//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(const int rates_total,const int prev_calculated,const datetime &time[],
                const double &open[],
                const double &high[],
                const double &low[],
                const double &close[],
                const long &tick_volume[],
                const long &volume[],
                const int &spread[])
  {
   if(Bars(_Symbol,_Period)<rates_total) return(prev_calculated);
   for(int i=(int)MathMax(prev_calculated-1,0); i<rates_total && !IsStopped(); i++)
     {
      prices[i]  = getPrice(inpPrice,open,close,high,low,i,rates_total);
      double avg = 0; for(int k=0; k<inpPeriod    && (i-k)>=0;   k++) avg +=         prices[i-k];                                            avg /= inpPeriod;
      double dev = 0; for(int k=0; k<inpPeriod    && (i-k)>=0;   k++) dev += MathAbs(prices[i-k]-avg);                                       dev /= inpPeriod;
      double atr = 0; for(int k=0; k<inpAtrPeriod && (i-k-1)>=0; k++) atr += MathMax(high[i-k],close[i-k-1])-MathMin(low[i-k],close[i-k-1]); atr /= inpAtrPeriod;
      double cci = (dev!=0) ?(prices[i]-avg)/(0.015*dev) : 0;
         trend[i] = (cci>0) ? 1 : (cci<0) ? -1 : (i>0) ? trend[i-1] : 0;
         val[i]  = (i>0) ? (trend[i]==1) ? MathMax(low[i]-atr,val[i-1]) : MathMin(high[i]+atr,val[i-1]) : close[i];
         valc[i] = (i>0) ? (val[i]>val[i-1]) ? 2 :(val[i]<val[i-1]) ? 1 : valc[i-1]: 0;
     }
   return(rates_total);
  }
//+------------------------------------------------------------------+
//| Custom functions                                                 |
//+------------------------------------------------------------------+
double getPrice(ENUM_APPLIED_PRICE tprice,const double &open[],const double &close[],const double &high[],const double &low[],int i,int _bars)
  {
   if(i>=0)
      switch(tprice)
        {
         case PRICE_CLOSE:     return(close[i]);
         case PRICE_OPEN:      return(open[i]);
         case PRICE_HIGH:      return(high[i]);
         case PRICE_LOW:       return(low[i]);
         case PRICE_MEDIAN:    return((high[i]+low[i])/2.0);
         case PRICE_TYPICAL:   return((high[i]+low[i]+close[i])/3.0);
         case PRICE_WEIGHTED:  return((high[i]+low[i]+close[i]+close[i])/4.0);
        }
   return(0);
  }
//+------------------------------------------------------------------+
