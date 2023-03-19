# P.G-M-canal
códigos do canal do youtube
//+------------------------------------------------------------------+
//|                                          Robo youtube Neting.mq5 |
//|                             Copyright 2023, programandonomercado |
//|                                 https://programandonomercado.com |
//+------------------------------------------------------------------+
#property copyright "Programando no Mercado"
#property link      "www.programandonomercado.com"
#property version   "1.00"
//+------------------------------------------------------------------+
//|INCLUDES                                                          |
//+------------------------------------------------------------------+
#include <Trade/Trade.mqh>


#resource "\\Indicators\\MACD Divergência.ex5"

//+------------------------------------------------------------------+
//|INPUT                                                             |
//+------------------------------------------------------------------+
input int lote = 1;  // VOLUME FINANCEIRO


input int Fast_MA = 12;                         // PERÍODO DA MÉDIA RÁPIDA

input int Slow_MA = 26;                         // PERÍODO DA MÉDIA LENTA

input ENUM_MA_METHOD MA_Method_=MODE_EMA;       // MÉTODO DE CÁLCULO DO INDICADOR

input int Signal_SMA=9;                         // PERÍODO DO SINAL
//+------------------------------------------------------------------+
//|GLOBALS                                                           |
//+------------------------------------------------------------------+


// ----
CTrade trade;

// ----
int macd_handler = INVALID_HANDLE;
double alta_buffer[];
double baixa_buffer[];

MqlRates candle[];

MqlTick tick;

//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit()
  {
//---


      ArraySetAsSeries(alta_buffer,true);
      
      ArraySetAsSeries(baixa_buffer,true);
      
      ArraySetAsSeries(candle,true);

// atribuir valores para mainipulador da média

   macd_handler = iCustom(_Symbol,_Period,"\\Indicators\\MACD Divergência.ex5",Fast_MA,Slow_MA,MA_Method_,Signal_SMA);
      //
      if(macd_handler==INVALID_HANDLE)
        {
         Print("ERRO -> Falha na inicialização dos indicadores");
         return(false);
        }


      if(!ChartIndicatorAdd(0,1,macd_handler))
        {

         return(false);
        }
   
//---
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Expert deinitialization function                                 |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
//---
   
  }
//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
void OnTick()
  {
//---
      bool comprado = false;
      bool vendido = false;
      
    
    
    if(!SymbolInfoTick(_Symbol,tick))
     {
      Print("ERRO -> Dados de ticks não foram copiados corretamente");
     
     }
   
   if(isNewBar())
     {
      //+------------------------------------------------------------------+
      //| OBTENÇÃO DOS DADOS                                               |
      //+------------------------------------------------------------------+
       int copied1 = CopyBuffer(macd_handler,0,0,5,alta_buffer);
       int copied2 = CopyBuffer(macd_handler,1,0,5,baixa_buffer);
       
       int copied3 = CopyRates(_Symbol,_Period,0,5,candle);
    


 // se os dados tiverem sido copiados corretamente
 
 bool sinalCompra = false;
 bool sinalVenda = false;
  if(copied1 == 5 && copied2 == 5)
    {
     // sinal de compra

   if(alta_buffer[1] > alta_buffer[2])
     {
      sinalCompra = true;
     }
// lógica do SINAL VENDA -> fechar abaixo da em21
   if(baixa_buffer[1] < baixa_buffer[2])
     {
      sinalVenda = true;
     }
     
    }
      
      
      //+------------------------------------------------------------------+
      //|VERIFICAR SE ESTOU POSICIONADO                                    |
      //+------------------------------------------------------------------+
     
      
      if(PositionSelect(_Symbol))
        {
          if(PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY)
            {
             comprado = true;
            }
        
          if(PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_SELL)
            {
             vendido = true;
            }
        
      }
      
    //+------------------------------------------------------------------+
    //| LÓGICA DE ROTEAMENTO                                             |
    //+------------------------------------------------------------------+
    // ZERADO
    if(!comprado && !vendido)
      {
      // sinal de compra
       if(sinalCompra)
         {
            double sl = GetSL(true);
            double tp = GetTP(true);
      
            trade.Buy(lote,_Symbol,0,sl,tp,"Compra a Mercado");
            Print("COMPRA ENVIADA COM SUCESSO");
           
         }

      // sinal de venda
      if(sinalVenda)
        {
            double sl = GetSL(false);
            double tp = GetTP(false);
            
           trade.Sell(lote,_Symbol,0,sl,tp,"Venda a Mercado");
           Print("VENDA ENVIADA COM SUCESSO");
        }
       }   
    }  
    
  }
//+------------------------------------------------------------------+
bool isNewBar()
  {
   static datetime last_time=0;
   datetime lastbar_time=(datetime)SeriesInfoInteger(Symbol(),Period(),SERIES_LASTBAR_DATE);
   if(last_time==0)
     {
      last_time=lastbar_time;
      return(false);
     }
   if(last_time!=lastbar_time)
     {
      last_time=lastbar_time;
      return(true);
     }
   return(false);
  }
  
//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+

  
  
//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
double GetSL(bool isBuy)
  {
   double sl = 0;
//
      double tickSize = SymbolInfoDouble(_Symbol,SYMBOL_TRADE_TICK_SIZE);
      if(isBuy)
        {
         sl = candle[1].low;
        }
      else
        {
         sl = candle[1].high;
        }
   
 
//
   return(sl);
  }

//
double GetTP(bool isBuy)
  {
   double tp = 0;
   
   //

         double tickSize = SymbolInfoDouble(_Symbol,SYMBOL_TRADE_TICK_SIZE);
         if(isBuy)
           {
            tp = tick.ask + (candle[1].high - candle[1].low)*2;
           }
         else
           {
            tp = tick.bid - (candle[1].high - candle[1].low)*2;
           }
//
   return(tp);
  }
  
  //+------------------------------------------------------------------+
  //|                                                                  |
  //+------------------------------------------------------------------+
  
