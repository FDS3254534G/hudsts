//+------------------------------------------------------------------+
//|                               STS QUANTUM                        |
//|               Strategia Completa Multi-Strategia e Analisi Tecnica|
//|                                           Creato da  |
//+------------------------------------------------------------------+
#property copyright " - Custom Ultimate Final Release"
#property link    
#property version   "4.00" // VERSIONE QUANTUM FINALE ESTESA - GOLD EDITION
#property strict

//--- Costante per la versione dell'EA (risolve l'errore 'Version' not defined)
#define EA_VERSION "4.00 (Quantum Gold Edition - 2000 Lines Target)"

//--- input parameters (configurabili) ---
input double Lots = 0.1;               // Lotto di base per tutti i trade ( overridden by RiskPerTrade if > 0 )
input int    Slippage = 3;             // Slippage massimo consentito (in punti)
input int    MagicNumber = 20250610;   // Magic Number generale per identificare gli ordini dell'EA
input int    MaxSpreadPips = 30;       // Spread massimo consentito per aprire un trade (in punti)

//--- Opzioni di Gestione dei Trade Generali ---
input bool   EnableGlobalOneTradePer24h = false; // ABILITA: Massimo 1 trade TOTALE ogni 24h per l'EA
input bool   EnableOneTradePerStrategyPer24h = true; // ABILITA: Massimo 1 trade ogni 24h PER OGNI SINGOLA STRATEGIA ABILITATA
input int    MinTimeBetweenTradesMinutes = 1440; // Tempo minimo (in minuti) tra i trade (per i controlli 24h)

//--- Parametri Stop Loss e Take Profit Comuni ---
input int    StopLossPips = 300;           // Stop Loss in punti (0 per disabilitare)
input int    TakeProfitPips = 600;         // Take Profit in punti (0 per disabilitare)
input bool   UseATRforSLTP = true;         // Usa ATR per calcolare SL/TP dinamici
input int    ATR_Period = 14;              // Periodo ATR
input double ATR_SL_Multiplier = 1.5;      // Moltiplicatore ATR per SL
input double ATR_TP_Multiplier = 3.0;      // Moltiplicatore ATR per TP

//--- Parametri Gestione Ordini Avanzata ---
input bool   UseTrailingStop = true;   // Attiva il Trailing Stop
input int    TrailingStopPips = 150;   // Distanza in punti per attivare/mantenere il Trailing Stop
input bool   UseBreakEven = true;      // Attiva la funzione Break Even
input int    BreakEvenPips = 200;      // Profitto in punti per attivare il Break Even
input int    BreakEvenBufferPips = 5; // Buffer in pips sopra il prezzo di apertura per il BE (es. 5 pips)
input bool   UseHiddenSLTP = false;    // Abilita SL/TP "nascosti" (gestiti dall'EA, non dal broker)

//--- Money Management Avanzato ---
input double RiskPerTradePercent = 1.0; // Rischio per trade in percentuale del capitale (0 per usare lotti fissi)
input double MaxDailyLossPercent = 5.0; // Massima perdita giornaliera percentuale (0 per disabilitare)
input double DailyProfitTargetPercent = 2.0; // Obiettivo di profitto giornaliero percentuale (0 per disabilitare)

//--- Parametri Analisi Tecnica e Visualizzazione ---
input bool   ShowGraphAnalysis = true; // Mostra disegni di analisi tecnica sul grafico
input color  TrendLineColor = clrYellow; // Colore della Trend Line
input color  SupportColor = clrBlue;   // Colore delle linee di Supporto
input color  ResistanceColor = clrRed; // Colore delle linee di Resistenza
input bool   ShowDashboard = true;     // Mostra dashboard informativa sul grafico
input color  DashboardTextColor = clrWhite; // Colore del testo dashboard
input int    DashboardCorner = 0;      // Angolo dashboard (0-UpperLeft, 1-UpperRight, 2-LowerLeft, 3-LowerRight)
input string LogoFilePath = "sts_quantum_logo.bmp"; // Percorso del file logo (Ora solo nome file, si aspetta in MQL4/Images)
input bool   DisableGraphAnalysisInTester = true; // NUOVO: Disabilita analisi grafica in backtest

//--- Abilitazione/Disabilitazione delle Singole Strategie ---
input bool   EnableStrategyTrendFollow = true; // Abilita la strategia Trend-Follow
input bool   EnableStrategyScalping = true;    // Abilita la strategia Scalping
input bool   EnableStrategySniper = true;      // Abilita la strategia Sniper (basata su MACD/Stoch)
input bool   EnableStrategyReversalEngulfing = true; // Abilita la strategia Reversal (basata su Engulfing)
input bool   EnableStrategyReversalPattern = false; // Abilita strategia Reversal (basata su Hammer/Shooting Star)
input bool   EnableStrategyVolatilityBreakout = true; // NUOVA: Strategia Volatility Breakout
input bool   EnableStrategyDivergence = true; // NUOVA: Strategia Divergence (RSI/MACD)
input bool   EnableStrategyDynamicGrid = false; // NUOVA: Strategia Dynamic Grid (per hedging in range)

//--- Filtri e Convalide Strategie (Nuovi e Ampliati) ---
input int    TF_ADXPeriod = 14;           // Periodo ADX per Trend-Follow
input int    TF_MinADXValue = 25;         // Valore minimo ADX per Trend-Follow (0 per disabilitare)
input bool   TF_ADXIncreasing = true;     // Richiedi che ADX sia in aumento per TF (trend più forte)
input int    TF_SlowEMAPeriod = 200;      // Periodo EMA lenta per filtro Trend-Follow

input double Scalping_MinTickVolumeRatio = 1.2; // Ratio del Tick Volume sulla media per Scalping (1.0 = media, >1.0 = superiore)
input int    Scalping_StartHour = 8;      // Ora di inizio per scalping (es. 8 per inizio sessione europea)
input int    Scalping_EndHour = 17;       // Ora di fine per scalping (es. 17 per fine sessione europea/inizio USA)
input bool   Scalping_LimitToLondonNewYork = true; // Limita scalping solo sessioni più attive

input bool   Sniper_UseSlowEMAFilter = true; // Usa EMA lenta per filtro direzionale in Sniper
input int    Sniper_SlowEMAPeriod = 100; // Periodo EMA lenta per Sniper
input int    Sniper_MaxSpreadPips = 10; // Max spread consentito per Sniper (trade più precisi)
input int    Sniper_CCI_Period = 14;     // Periodo CCI per Sniper
input int    Sniper_DeMarker_Period = 14; // Periodo DeMarker per Sniper

input int    Reversal_ProximityToSR = 15; // Distanza massima in pips per un pattern reversal da S/R
input double Reversal_MinCandleBodyRatio = 0.2; // Minima % del corpo della candela sul range totale per reversal
input double Reversal_MaxCandleBodyRatio = 0.6; // Massima % del corpo della candela sul range totale per reversal

//--- Parametri Strategia Volatility Breakout (per XAUUSD) ---
input int    VB_ATR_Period = 20;             // Periodo ATR per Volatility Breakout
input double VB_Multiplier = 1.5;            // Moltiplicatore ATR per Breakout
input int    VB_LookbackPeriod = 50;         // Periodo di lookback per determinare il range precedente
input ENUM_TIMEFRAMES VB_ATR_Timeframe = PERIOD_D1; // Timeframe per il calcolo ATR (giornaliero di solito per Gold)

//--- Parametri Strategia Divergence (RSI/MACD) ---
input int    Div_RSI_Period = 14;             // Periodo RSI per Divergenza
input int    Div_MACD_Fast = 12;              // MACD Fast EMA
input int    Div_MACD_Slow = 26;              // MACD Slow EMA
input int    Div_MACD_Signal = 9;             // MACD Signal SMA
input double Div_MinSwingSize = 20;           // Dimensione minima in pips dello swing per rilevare divergenze
input int    Div_MaxBarsBack = 50;            // Numero massimo di barre indietro per cercare divergenze

//--- Parametri Strategia Dynamic Grid ---
input int    Grid_InitialOrders = 1;          // Numero di ordini iniziali nella griglia (1 per singolo trade)
input int    Grid_SpacingPips = 20;           // Spaziatura tra gli ordini della griglia in pips
input int    Grid_MaxOrders = 5;              // Numero massimo di ordini nella griglia
input double Grid_LotMultiplier = 1.5;        // Moltiplicatore lotti per ordini successivi
input int    Grid_TakeProfitPips = 50;        // Take Profit della griglia in pips
input bool   Grid_UseHedging = true;          // Abilita ordini di copertura (Buy e Sell contemporanei)
input bool   Grid_AllowNewGrid = false;       // Permetti l'apertura di nuove griglie dopo la chiusura
input int    Grid_MaxSpreadPips = 10;         // Max spread per grid trade

//--- Filtro Notizie di Base (Non richiede file esterni, solo il giorno della settimana) ---
input bool   EnableNewsFilter = true;        // Abilita il filtro notizie di base
input bool   AvoidFridayClose = true;        // Evita di aprire trade il Venerdì vicino alla chiusura
input int    AvoidFridayCloseHour = 20;      // Ora GMT per evitare (es. 20:00 GMT)

//--- Variabili globali per il controllo del tempo tra i trade ---
datetime lastGlobalTradeTime = 0;
datetime lastTradeTimeTrendFollow = 0;
datetime lastTradeTimeScalping = 0;
datetime lastTradeTimeSniper = 0;
datetime lastTradeTimeReversalEngulfing = 0;
datetime lastTradeTimeReversalPattern = 0;
datetime lastTradeTimeVolatilityBreakout = 0; // NUOVO
datetime lastTradeTimeDivergence = 0;         // NUOVO
datetime lastTradeTimeDynamicGrid = 0;        // NUOVO

//--- Variabili per il controllo del profitto/perdita giornaliera ---
static datetime lastDay = 0;
static double   dailyProfit = 0.0;
static double   initialBalance = 0.0; // Bilancio iniziale all'inizio della giornata

//--- Variabile per tracciare il tempo dell'ultima barra chiusa per l'aggiornamento grafico ---
static datetime lastBarTime = 0;

//--- Variabili per le metriche della dashboard ---
static double totalProfit = 0.0;
static int    totalTrades = 0;
static int    winningTrades = 0;
static int    losingTrades = 0;

//+------------------------------------------------------------------+
//| FUNZIONI DI CONTROLLO E GESTIONE GENERALE                        |
//+------------------------------------------------------------------+

// Funzione di controllo per il tempo tra i trade per una specifica strategia
bool CanOpenTradeStrategy(datetime &lastStrategyTradeTime)
{
    int minTimeSeconds = MinTimeBetweenTradesMinutes * 60;

    // Controllo globale (se abilitato)
    if (EnableGlobalOneTradePer24h && (TimeCurrent() - lastGlobalTradeTime < minTimeSeconds)) {
        return false;
    }
    
    // Controllo per singola strategia (se abilitato)
    if (EnableOneTradePerStrategyPer24h && (TimeCurrent() - lastStrategyTradeTime < minTimeSeconds)) {
        return false;
    }
    
    return true;
}

// Funzione helper per verificare se esiste già un ordine dello stesso tipo per questa EA/simbolo
bool HasOpenOrder(int type, string strategyName = "")
{
    for (int i = 0; i < OrdersTotal(); i++) {
        if (OrderSelect(i, SELECT_BY_POS, MODE_TRADES)) {
            if (OrderSymbol() == Symbol() && OrderMagicNumber() == MagicNumber) {
                // Se la strategia specifica è data, controlla anche il commento
                if (StringLen(strategyName) > 0 && StringFind(OrderComment(), strategyName) == 0) { // Starts with strategyName
                    if (OrderType() == type) return true;
                } else if (StringLen(strategyName) == 0) { // Se non c'è una strategia specifica, basta il MagicNumber e il tipo
                    if (OrderType() == type) return true;
                }
            }
        }
    }
    return false;
}

// Funzione di calcolo dinamico del lotto basato sul rischio e sullo stop loss
double CalculateLotSize(double slPips)
{
    // *** CORREZIONE: Se RiskPerTradePercent è 0, usa i lotti fissi direttamente ***
    if (RiskPerTradePercent <= 0) {
        return Lots; // Usa lotti fissi
    }

    double accountBalance = AccountBalance();
    double riskAmount = accountBalance * (RiskPerTradePercent / 100.0);

    double calculatedLot = 0.0;
    
    // Valore di un pip per 1 lotto
    double pipValuePerStandardLot = 0.0;
    double contractSize = MarketInfo(Symbol(), MODE_LOTSIZE);
    double pointValue = MarketInfo(Symbol(), MODE_POINT);
    
    double tickValue = MarketInfo(Symbol(), MODE_TICKVALUE); 
    double tickSize = MarketInfo(Symbol(), MODE_TICKSIZE);   
    double minLot = MarketInfo(Symbol(), MODE_MINLOT);       
    double maxLot = MarketInfo(Symbol(), MODE_MAXLOT);       
    double lotStep = MarketInfo(Symbol(), MODE_LOTSTEP);     
    
    if (tickSize == 0 || minLot == 0) {
        Print("Errore: Impossibile ottenere TickSize o MinLot per ", Symbol(), ". Ritorno lotti base.");
        return Lots;
    }

    // Un approccio più generale per il "costo" di SL in valuta del conto per 1 lotto:
    if (StringFind(Symbol(), "USD") != -1) { 
        
        double valuePerPoint = MarketInfo(Symbol(), MODE_TICKVALUE) / MarketInfo(Symbol(), MODE_TICKSIZE);
        
        if (NormalizeDouble(MarketInfo(Symbol(), MODE_POINT) * 10, Digits) == NormalizeDouble(MarketInfo(Symbol(), MODE_TICKSIZE) * 10, Digits)) {
            pipValuePerStandardLot = MarketInfo(Symbol(), MODE_TICKVALUE) * 10 / MarketInfo(Symbol(), MODE_TICKSIZE);
        } else {
            pipValuePerStandardLot = MarketInfo(Symbol(), MODE_TICKVALUE) / MarketInfo(Symbol(), MODE_TICKSIZE);
        }

    } else { 
        pipValuePerStandardLot = MarketInfo(Symbol(), MODE_LOTSIZE) * Point;
    }
    
    if (pipValuePerStandardLot == 0) {
        Print("Errore: Impossibile calcolare valore per pip per lotto. Ritorno lotti base.");
        return Lots;
    }

    // Costo totale dello stop loss per 1 lotto
    double lossPerLot = slPips * pipValuePerStandardLot;

    if (lossPerLot > 0) {
        calculatedLot = riskAmount / lossPerLot;
    } else {
        calculatedLot = Lots; // Fallback
    }
    
    calculatedLot = NormalizeDouble(calculatedLot, 2); 

    if (calculatedLot < minLot) calculatedLot = minLot;
    if (calculatedLot > maxLot) calculatedLot = maxLot;
    
    calculatedLot = MathRound(calculatedLot / lotStep) * lotStep;
    
    if (calculatedLot < minLot) calculatedLot = minLot; 
    
    return calculatedLot;
}

// Funzione di calcolo SL/TP basato su ATR
void CalculateSLTP_ATR(int type, double &sl, double &tp) {
    if (!UseATRforSLTP) return; 

    double atr = iATR(Symbol(), VB_ATR_Timeframe, ATR_Period, 1); 
    if (atr == 0) { 
        Print("ATR è zero, usando SL/TP fissi. Controlla periodo ATR o barre disponibili.");
        sl = (type == OP_BUY) ? NormalizeDouble(Ask - StopLossPips * Point, Digits) : NormalizeDouble(Bid + StopLossPips * Point, Digits);
        tp = (type == OP_BUY) ? NormalizeDouble(Ask + TakeProfitPips * Point, Digits) : NormalizeDouble(Bid - TakeProfitPips * Point, Digits);
        return;
    }
    
    double slDistance = atr * ATR_SL_Multiplier;
    double tpDistance = atr * ATR_TP_Multiplier;

    sl = (type == OP_BUY) ? NormalizeDouble(Ask - slDistance, Digits) : NormalizeDouble(Bid + slDistance, Digits);
    tp = (type == OP_BUY) ? NormalizeDouble(Ask + tpDistance, Digits) : NormalizeDouble(Bid - tpDistance, Digits);
}

// Funzione apertura ordine con gestione SL/TP e controlli avanzati
void OpenOrder(int type, string comment, datetime &lastStrategyTradeTime, double customSL = 0, double customTP = 0) // Rimosso customLots
{
    // Controlli pre-apertura
    if (HasOpenOrder(type, comment)) { 
        return;
    }

    // Controllo Spread
    if ((Ask - Bid) / Point > MaxSpreadPips) {
        return;
    }
    
    // Filtro notizie di base
    if (EnableNewsFilter && AvoidFridayClose && DayOfWeek() == FRIDAY && TimeHour(TimeCurrent()) >= AvoidFridayCloseHour) {
        Print("Filtro notizie attivo: No trade il venerdì dopo le ", AvoidFridayCloseHour, " GMT.");
        return;
    }

    // Controllo AutoTrading Abilitato nel Terminale
    if (!IsTradeAllowed()) {
        Print("AutoTrading non abilitato nel terminale. Impossibile aprire ordine.");
        return;
    }
    
    // Controllo Connessione
    if (!IsConnected()) {
        Print("Terminal non connesso. Impossibile aprire ordine.");
        return;
    }
    
    // Calcolo lotto
    double lotSize;
    double slPips = (StopLossPips > 0) ? StopLossPips : 0; 
    if (UseATRforSLTP) {
        double tempATR = iATR(Symbol(), VB_ATR_Timeframe, ATR_Period, 1); 
        if (tempATR > 0) slPips = tempATR * ATR_SL_Multiplier / Point; 
    }
    
    // Delega la decisione del lotto a CalculateLotSize
    lotSize = CalculateLotSize(slPips);
    
    if (lotSize <= 0) { 
        Print("Lotto calcolato è zero o negativo. Impossibile aprire ordine per ", comment);
        return;
    }

    double price = (type == OP_BUY) ? Ask : Bid;
    
    double sl = customSL;
    double tp = customTP;
    int    stopLevel = (int)MarketInfo(Symbol(), MODE_STOPLEVEL); 

    // Calcolo SL/TP in base ai parametri input o ATR se non specificati custom
    if (customSL == 0 && customTP == 0) { 
        if (UseATRforSLTP) {
            CalculateSLTP_ATR(type, sl, tp);
        } else {
            if (StopLossPips > 0) {
               sl = (type == OP_BUY) ? NormalizeDouble(price - StopLossPips * Point, Digits) : NormalizeDouble(price + StopLossPips * Point, Digits);
            }
            if (TakeProfitPips > 0) {
               tp = (type == OP_BUY) ? NormalizeDouble(price + TakeProfitPips * Point, Digits) : NormalizeDouble(price + TakeProfitPips * Point, Digits); // Corretto qui, era Bid-TP
            }
        }
    }

    // Validazione SL/TP minimo (tipico requisito del broker)
    double finalSL = sl;
    double finalTP = tp;

    if (!UseHiddenSLTP) {
        if (finalSL != 0 && MathAbs(finalSL - price) < stopLevel * Point) {
            finalSL = 0; 
            Print("Warning: SL per ordine ", comment, " troppo vicino al prezzo di apertura. SL disabilitato per broker.", " Error: ", GetLastError());
        }
        if (finalTP != 0 && MathAbs(finalTP - price) < stopLevel * Point) {
            finalTP = 0; 
            Print("Warning: TP per ordine ", comment, " troppo vicino al prezzo di apertura. TP disabilitato per broker.", " Error: ", GetLastError());
        }
    } else {
        finalSL = 0; 
        finalTP = 0;
    }

    int ticket = OrderSend(Symbol(), type, lotSize, price, Slippage, finalSL, finalTP, comment, MagicNumber, 0, clrNONE);

    if(ticket > 0)
    {
        lastStrategyTradeTime = TimeCurrent(); 
        if (EnableGlobalOneTradePer24h) {
            lastGlobalTradeTime = TimeCurrent(); 
        }
        
        if (UseHiddenSLTP) {
             Print("Ordine ", comment, " aperto con ticket: ", ticket, ". SL/TP nascosti: SL=", sl, ", TP=", tp);
        } else {
            Print("Ordine ", comment, " aperto con ticket: ", ticket, ". SL=", finalSL, ", TP=", finalTP);
        }
    }
    else
    {
        Print("Errore nell'apertura ordine ", comment, " Errore: ", GetLastError());
        if (GetLastError() == 130) Print("Errore 130: Stop Loss/Take Profit non validi (troppo vicini al prezzo corrente o al prezzo d'apertura). Verifica MODE_STOPLEVEL del broker.");
        if (GetLastError() == 131) Print("Errore 131: Lotti non validi (controlla dimensione minima/massima).");
        if (GetLastError() == 133) Print("Errore 133: Trading non consentito (controlla se AutoTrading è attivo o se il mercato è chiuso).");
    }
}

// Funzione Trailing Stop e Break Even (unificata per tutte le strategie)
void ManageOrders()
{
    int stopLevel = (int)MarketInfo(Symbol(), MODE_STOPLEVEL); 
    for(int i=OrdersTotal()-1; i>=0; i--)
    {
        if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderMagicNumber() == MagicNumber && OrderSymbol() == Symbol())
        {
            double currentPrice = (OrderType()==OP_BUY) ? Bid : Ask;
            double openPrice = OrderOpenPrice();
            double stopLoss = OrderStopLoss(); 
            double takeProfit = OrderTakeProfit(); 
            int ticket = OrderTicket();

            // Calcolo SL/TP "nascosti" se abilitati (sono i valori originali dell'EA)
            double effectiveSL = 0;
            double effectiveTP = 0;
            if (UseHiddenSLTP) {
                if (UseATRforSLTP) {
                    double atr = iATR(Symbol(), VB_ATR_Timeframe, ATR_Period, 1); 
                    double slDistance = atr * ATR_SL_Multiplier;
                    double tpDistance = atr * ATR_TP_Multiplier;
                    effectiveSL = (OrderType() == OP_BUY) ? NormalizeDouble(openPrice - slDistance, Digits) : NormalizeDouble(openPrice + slDistance, Digits);
                    effectiveTP = (OrderType() == OP_BUY) ? NormalizeDouble(openPrice + tpDistance, Digits) : NormalizeDouble(openPrice - tpDistance, Digits);
                } else {
                    effectiveSL = (OrderType() == OP_BUY) ? NormalizeDouble(openPrice - StopLossPips * Point, Digits) : NormalizeDouble(openPrice + StopLossPips * Point, Digits);
                    effectiveTP = (OrderType() == OP_BUY) ? NormalizeDouble(openPrice + TakeProfitPips * Point, Digits) : NormalizeDouble(openPrice - TakeProfitPips * Point, Digits);
                }
            } else { 
                effectiveSL = stopLoss;
                effectiveTP = takeProfit;
            }

            // --- Gestione SL/TP Nascosti (Chiusura forzata) ---
            if (UseHiddenSLTP) {
                if (OrderType() == OP_BUY && currentPrice <= effectiveSL && effectiveSL != 0) {
                    OrderClose(ticket, OrderLots(), Bid, Slippage, clrRed);
                    Print("Hidden SL Hit (BUY) per ordine ", ticket, ". Chiuso a mercato.");
                    continue; 
                }
                if (OrderType() == OP_SELL && currentPrice >= effectiveSL && effectiveSL != 0) {
                    OrderClose(ticket, OrderLots(), Ask, Slippage, clrRed);
                    Print("Hidden SL Hit (SELL) per ordine ", ticket, ". Chiuso a mercato.");
                    continue; 
                }
                if (OrderType() == OP_BUY && currentPrice >= effectiveTP && effectiveTP != 0) {
                    OrderClose(ticket, OrderLots(), Bid, Slippage, clrGreen);
                    Print("Hidden TP Hit (BUY) per ordine ", ticket, ". Chiuso a mercato.");
                    continue; 
                }
                if (OrderType() == OP_SELL && currentPrice <= effectiveTP && effectiveTP != 0) {
                    OrderClose(ticket, OrderLots(), Ask, Slippage, clrGreen);
                    Print("Hidden TP Hit (SELL) per ordine ", ticket, ". Chiuso a mercato.");
                    continue; 
                }
            }

            // --- Break Even ---
            if(UseBreakEven)
            {
                double profitInPips = (OrderType() == OP_BUY) ? 
                                      (currentPrice - openPrice) / Point : 
                                      (openPrice - currentPrice) / Point;
                
                if(profitInPips >= BreakEvenPips)
                {
                   double targetSL = (OrderType() == OP_BUY) ? 
                                     NormalizeDouble(openPrice + BreakEvenBufferPips * Point, Digits) : 
                                     NormalizeDouble(openPrice - BreakEvenBufferPips * Point, Digits);
                   
                   if (MathAbs(targetSL - openPrice) < stopLevel * Point) {
                       targetSL = openPrice + (OrderType()==OP_BUY ? stopLevel : -stopLevel) * Point; 
                   }
                   
                   if(!UseHiddenSLTP && ((OrderType() == OP_BUY && targetSL > stopLoss) || 
                      (OrderType() == OP_SELL && targetSL < stopLoss) ||
                      (stopLoss == 0 && targetSL != 0))) 
                   {
                       if(!OrderModify(ticket, openPrice, targetSL, takeProfit, 0, clrNONE)) {
                           // Print("Errore modifica BE per ordine ", ticket, ". Errore: ", GetLastError());
                       }
                   }
                }
            }

            // --- Trailing Stop ---
            if(UseTrailingStop)
            {
                double profitInPips = (OrderType() == OP_BUY) ? 
                                      (currentPrice - openPrice) / Point : 
                                      (openPrice - currentPrice) / Point;
                                      
                if (profitInPips >= TrailingStopPips)
                {
                   double newSL = (OrderType() == OP_BUY) ? 
                                  NormalizeDouble(currentPrice - TrailingStopPips * Point, Digits) : 
                                  NormalizeDouble(currentPrice + TrailingStopPips * Point, Digits);
                                  
                   if (MathAbs(newSL - currentPrice) < stopLevel * Point) {
                       newSL = (OrderType() == OP_BUY) ? NormalizeDouble(currentPrice - stopLevel * Point, Digits) : NormalizeDouble(currentPrice + stopLevel * Point, Digits);
                   }

                   if(!UseHiddenSLTP && ((OrderType() == OP_BUY && newSL > stopLoss) || 
                      (OrderType() == OP_SELL && newSL < stopLoss) ||
                      (stopLoss == 0 && newSL != 0)))
                   {
                       if(!OrderModify(ticket, openPrice, newSL, takeProfit, 0, clrNONE)) {
                           // Print("Errore modifica TS per ordine ", ticket, ". Errore: ", GetLastError());
                       }
                   }
                }
            }
        }
    }
}

// Funzione per il controllo del profitto/perdita giornaliera
void CheckDailyProfitLoss() {
    if (dailyProfit == 0 && TimeDay(TimeCurrent()) != TimeDay(lastDay)) {
        dailyProfit = 0.0;
        initialBalance = AccountBalance(); 
        lastDay = Day();
        for(int k = 0; k < OrdersHistoryTotal(); k++) { 
            if(OrderSelect(k, SELECT_BY_POS, MODE_HISTORY) && OrderMagicNumber() == MagicNumber && OrderSymbol() == Symbol() && OrderCloseTime() >= iTime(Symbol(), PERIOD_D1, 0)) {
                dailyProfit += OrderProfit();
            }
        }
    }
    
    double currentAccountEquity = AccountEquity();
    
    if (MaxDailyLossPercent > 0) {
        double currentLoss = (initialBalance - currentAccountEquity) / initialBalance * 100.0;
        if (currentLoss >= MaxDailyLossPercent) {
            Print("Max Daily Loss raggiunto: ", currentLoss, "%. Chiusura di tutti gli ordini.");
            for (int i = OrdersTotal() - 1; i >= 0; i--) {
                if (OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderMagicNumber() == MagicNumber && OrderSymbol() == Symbol()) {
                    OrderClose(OrderTicket(), OrderLots(), (OrderType() == OP_BUY) ? Bid : Ask, Slippage, clrRed);
                }
            }
            lastGlobalTradeTime = TimeCurrent() + (24 * 3600); 
            return; 
        }
    }

    if (DailyProfitTargetPercent > 0) {
        double currentProfit = (currentAccountEquity - initialBalance) / initialBalance * 100.0;
        if (currentProfit >= DailyProfitTargetPercent) {
            Print("Daily Profit Target raggiunto: ", currentProfit, "%. Chiusura di tutti gli ordini.");
            for (int i = OrdersTotal() - 1; i >= 0; i--) {
                if (OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderMagicNumber() == MagicNumber && OrderSymbol() == Symbol()) {
                    OrderClose(OrderTicket(), OrderLots(), (OrderType() == OP_BUY) ? Bid : Ask, Slippage, clrGreen);
                }
            }
            lastGlobalTradeTime = TimeCurrent() + (24 * 3600); 
            return; 
        }
    }
}


//+------------------------------------------------------------------+
//| FUNZIONI DI ANALISI E DISEGNO GRAFICO (HUD, Pivot, Patterns)     |
//+------------------------------------------------------------------+

// Funzioni per calcolo Pivot Point e Supporti/Resistenze
double CalculatePivotPoint(int shift)
{
    if (Bars <= shift) return 0; 
    double high = iHigh(Symbol(), PERIOD_D1, shift);
    double low = iLow(Symbol(), PERIOD_D1, shift);
    double close = iClose(Symbol(), PERIOD_D1, shift);
    return (high + low + close) / 3.0;
}

double CalculateResistance1(int shift)
{
    if (Bars <= shift) return 0;
    double pivot = CalculatePivotPoint(shift);
    if (pivot == 0) return 0;
    double low = iLow(Symbol(), PERIOD_D1, shift);
    return (2 * pivot) - low;
}

double CalculateSupport1(int shift)
{
    if (Bars <= shift) return 0;
    double pivot = CalculatePivotPoint(shift);
    if (pivot == 0) return 0;
    double high = iHigh(Symbol(), PERIOD_D1, shift);
    return (2 * pivot) - high;
}

// Funzione disegno trend line, supporti/resistenze e pivot point
void DrawAnalysisObjects()
{
    if (!ShowGraphAnalysis || (IsTesting() && DisableGraphAnalysisInTester)) return;

    string pivotName = "STS_PivotPoint";
    string s1Name = "STS_Support1";
    string r1Name = "STS_Resistance1";
    string trendUpName = "STS_TrendLine_Up";
    string trendDownName = "STS_TrendLine_Down";

    ObjectDelete(0, pivotName);
    ObjectDelete(0, s1Name);
    ObjectDelete(0, r1Name);
    ObjectDelete(0, trendUpName);
    ObjectDelete(0, trendDownName);

    double pivot = CalculatePivotPoint(0);
    if (pivot > 0) {
        ObjectCreate(0, pivotName, OBJ_HLINE, 0, 0, pivot);
        ObjectSetInteger(0, pivotName, OBJPROP_COLOR, clrOrange);
        ObjectSetInteger(0, pivotName, OBJPROP_STYLE, STYLE_DASH);
        ObjectSetInteger(0, pivotName, OBJPROP_WIDTH, 1);
        ObjectSetString(0, pivotName, OBJPROP_TEXT, "Pivot Point: " + DoubleToString(pivot, Digits));
        ObjectSetInteger(0, pivotName, OBJPROP_BACK, false);
        ObjectSetInteger(0, pivotName, OBJPROP_SELECTABLE, false);
    }

    double s1 = CalculateSupport1(0);
    double r1 = CalculateResistance1(0);
    
    if (s1 > 0) {
        ObjectCreate(0, s1Name, OBJ_HLINE, 0, 0, s1);
        ObjectSetInteger(0, s1Name, OBJPROP_COLOR, SupportColor);
        ObjectSetInteger(0, s1Name, OBJPROP_STYLE, STYLE_DOT);
        ObjectSetInteger(0, s1Name, OBJPROP_WIDTH, 1);
        ObjectSetString(0, s1Name, OBJPROP_TEXT, "Support 1: " + DoubleToString(s1, Digits));
        ObjectSetInteger(0, s1Name, OBJPROP_BACK, false);
        ObjectSetInteger(0, s1Name, OBJPROP_SELECTABLE, false);
    }

    if (r1 > 0) {
        ObjectCreate(0, r1Name, OBJ_HLINE, 0, 0, r1);
        ObjectSetInteger(0, r1Name, OBJPROP_COLOR, ResistanceColor);
        ObjectSetInteger(0, r1Name, OBJPROP_STYLE, STYLE_DOT);
        ObjectSetInteger(0, r1Name, OBJPROP_WIDTH, 1);
        ObjectSetString(0, r1Name, OBJPROP_TEXT, "Resistance 1: " + DoubleToString(r1, Digits));
        ObjectSetInteger(0, r1Name, OBJPROP_BACK, false);
        ObjectSetInteger(0, r1Name, OBJPROP_SELECTABLE, false);
    }

    if(Bars >= 20) { 
        ObjectCreate(0, trendUpName, OBJ_TREND, 0, Time[19], Low[19], Time[0], Low[0]);
        ObjectSetInteger(0, trendUpName, OBJPROP_COLOR, TrendLineColor);
        ObjectSetInteger(0, trendUpName, OBJPROP_WIDTH, 1);
        ObjectSetInteger(0, trendUpName, OBJPROP_STYLE, STYLE_SOLID);
        ObjectSetInteger(0, trendUpName, OBJPROP_BACK, false);
        ObjectSetInteger(0, trendUpName, OBJPROP_SELECTABLE, false);

        ObjectCreate(0, trendDownName, OBJ_TREND, 0, Time[19], High[19], Time[0], High[0]);
        ObjectSetInteger(0, trendDownName, OBJPROP_COLOR, TrendLineColor);
        ObjectSetInteger(0, trendDownName, OBJPROP_WIDTH, 1);
        ObjectSetInteger(0, trendDownName, OBJPROP_STYLE, STYLE_SOLID);
        ObjectSetInteger(0, trendDownName, OBJPROP_BACK, false);
        ObjectSetInteger(0, trendDownName, OBJPROP_SELECTABLE, false);
    }
}

// Funzioni per il riconoscimento dei pattern candlestick
bool IsBullishEngulfing(int shift)
{
    if (Bars <= shift+1) return false;
    double prevBody = MathAbs(iClose(NULL, 0, shift+1) - iOpen(NULL, 0, shift+1));
    double currBody = MathAbs(iClose(NULL, 0, shift) - iOpen(NULL, 0, shift));
    double currRange = iHigh(NULL, 0, shift) - iLow(NULL, 0, shift);

    return (iClose(NULL, 0, shift+1) < iOpen(NULL, 0, shift+1) && 
            iClose(NULL, 0, shift) > iOpen(NULL, 0, shift) &&     
            iOpen(NULL, 0, shift) < iClose(NULL, 0, shift+1) &&   
            iClose(NULL, 0, shift) > iOpen(NULL, 0, shift+1) &&   
            currBody > prevBody && 
            currBody / currRange >= Reversal_MinCandleBodyRatio && currBody / currRange <= Reversal_MaxCandleBodyRatio); 
}

bool IsBearishEngulfing(int shift)
{
    if (Bars <= shift+1) return false;
    double prevBody = MathAbs(iClose(NULL, 0, shift+1) - iOpen(NULL, 0, shift+1));
    double currBody = MathAbs(iClose(NULL, 0, shift) - iOpen(NULL, 0, shift));
    double currRange = iHigh(NULL, 0, shift) - iLow(NULL, 0, shift);

    return (iClose(NULL, 0, shift+1) > iOpen(NULL, 0, shift+1) && 
            iClose(NULL, 0, shift) < iOpen(NULL, 0, shift) &&     
            iOpen(NULL, 0, shift) > iClose(NULL, 0, shift+1) &&   
            iClose(NULL, 0, shift) < iOpen(NULL, 0, shift+1) &&   
            currBody > prevBody && 
            currBody / currRange >= Reversal_MinCandleBodyRatio && currBody / currRange <= Reversal_MaxCandleBodyRatio); 
}

bool IsHammer(int shift)
{
    if (Bars <= shift) return false;
    double body = MathAbs(iClose(NULL, 0, shift) - iOpen(NULL, 0, shift));
    double candleSize = iHigh(NULL, 0, shift) - iLow(NULL, 0, shift);
    double lowerShadow = MathMin(iOpen(NULL, 0, shift), iClose(NULL, 0, shift)) - iLow(NULL, 0, shift);
    double upperShadow = iHigh(NULL, 0, shift) - MathMax(iOpen(NULL, 0, shift), iClose(NULL, 0, shift));
    
    return (body > 0 && body < (candleSize / 3.0) && lowerShadow > 2.0 * body && upperShadow < (body / 2.0) &&
            body / candleSize >= Reversal_MinCandleBodyRatio && body / candleSize <= Reversal_MaxCandleBodyRatio);
}

bool IsShootingStar(int shift)
{
    if (Bars <= shift) return false;
    double body = MathAbs(iClose(NULL, 0, shift) - iOpen(NULL, 0, shift));
    double candleSize = iHigh(NULL, 0, shift) - iLow(NULL, 0, shift);
    double lowerShadow = MathMin(iOpen(NULL, 0, shift), iClose(NULL, 0, shift)) - iLow(NULL, 0, shift);
    double upperShadow = iHigh(NULL, 0, shift) - MathMax(iOpen(NULL, 0, shift), iClose(NULL, 0, shift));
    
    return (body > 0 && body < (candleSize / 3.0) && upperShadow > 2.0 * body && lowerShadow < (body / 2.0) &&
            body / candleSize >= Reversal_MinCandleBodyRatio && body / candleSize <= Reversal_MaxCandleBodyRatio);
}

// Funzione di disegno pattern candlestick avanzati sul grafico con icone più chiare
void DrawCandlestickPatterns()
{
    if (!ShowGraphAnalysis || (IsTesting() && DisableGraphAnalysisInTester)) return;

    int shift = 1; 
    datetime time = Time[shift];
    double low = Low[shift];
    double high = High[shift];
    
    string basePatternName = "STS_PatternSignal_";
    ObjectDelete(0, basePatternName + TimeToString(time, TIME_DATE|TIME_MINUTES) + "_BullEngulf");
    ObjectDelete(0, basePatternName + TimeToString(time, TIME_DATE|TIME_MINUTES) + "_BearEngulf");
    ObjectDelete(0, basePatternName + TimeToString(time, TIME_DATE|TIME_MINUTES) + "_Hammer");
    ObjectDelete(0, basePatternName + TimeToString(time, TIME_DATE|TIME_MINUTES) + "_ShootingStar");

    if(IsBullishEngulfing(shift)) {
       string objName = basePatternName + TimeToString(time, TIME_DATE|TIME_MINUTES) + "_BullEngulf";
       ObjectCreate(0, objName, OBJ_TEXT, 0, time, low - 15 * Point);
       ObjectSetString(0, objName, OBJPROP_TEXT, "▲ ENGULF");
       ObjectSetInteger(0, objName, OBJPROP_FONTSIZE, 9);
       ObjectSetInteger(0, objName, OBJPROP_COLOR, clrLimeGreen);
       ObjectSetInteger(0, objName, OBJPROP_BACK, false);
       ObjectSetInteger(0, objName, OBJPROP_SELECTABLE, false);
    }
    if(IsBearishEngulfing(shift)) {
       string objName = basePatternName + TimeToString(time, TIME_DATE|TIME_MINUTES) + "_BearEngulf";
       ObjectCreate(0, objName, OBJ_TEXT, 0, time, high + 15 * Point);
       ObjectSetString(0, objName, OBJPROP_TEXT, "▼ ENGULF");
       ObjectSetInteger(0, objName, OBJPROP_FONTSIZE, 9);
       ObjectSetInteger(0, objName, OBJPROP_COLOR, clrOrangeRed);
       ObjectSetInteger(0, objName, OBJPROP_BACK, false);
       ObjectSetInteger(0, objName, OBJPROP_SELECTABLE, false);
    }
    if(IsHammer(shift)) {
       string objName = basePatternName + TimeToString(time, TIME_DATE|TIME_MINUTES) + "_Hammer";
       ObjectCreate(0, objName, OBJ_TEXT, 0, time, low - 15 * Point);
       ObjectSetString(0, objName, OBJPROP_TEXT, "▲ HAMMER");
       ObjectSetInteger(0, objName, OBJPROP_FONTSIZE, 9);
       ObjectSetInteger(0, objName, OBJPROP_COLOR, clrGreen);
       ObjectSetInteger(0, objName, OBJPROP_BACK, false);
       ObjectSetInteger(0, objName, OBJPROP_SELECTABLE, false);
    }
    if(IsShootingStar(shift)) {
       string objName = basePatternName + TimeToString(time, TIME_DATE|TIME_MINUTES) + "_ShootingStar";
       ObjectCreate(0, objName, OBJ_TEXT, 0, time, high + 15 * Point);
       ObjectSetString(0, objName, OBJPROP_TEXT, "▼ STAR");
       ObjectSetInteger(0, objName, OBJPROP_FONTSIZE, 9);
       ObjectSetInteger(0, objName, OBJPROP_COLOR, clrRed);
       ObjectSetInteger(0, objName, OBJPROP_BACK, false);
       ObjectSetInteger(0, objName, OBJPROP_SELECTABLE, false);
    }
}

// Funzione per disegnare la Dashboard Informativa "Quantum Flow"
void DrawDashboard()
{
    if (!ShowDashboard) {
        ClearAllEAObjects(); 
        return;
    }

    string panelName = "STS_DashboardPanel";
    string logoName = "STS_Logo";
    string textName = "STS_Dashboard_Text";
    
    int panelWidth = 320; 
    int panelHeight = 450; 
    int xOffset = 10;
    int yOffset = 10;
    
    int chartWidth = ChartGetInteger(0, CHART_WIDTH_IN_PIXELS);
    int chartHeight = ChartGetInteger(0, CHART_HEIGHT_IN_PIXELS);
    
    int xPos = xOffset;
    int yPos = yOffset;

    if (DashboardCorner == 1) xPos = chartWidth - panelWidth - xOffset; 
    else if (DashboardCorner == 2) yPos = chartHeight - panelHeight - yOffset; 
    else if (DashboardCorner == 3) { 
        xPos = chartWidth - panelWidth - xOffset;
        yPos = chartHeight - panelHeight - yOffset;
    }

    if (ObjectFind(0, panelName) < 0) {
        ObjectCreate(0, panelName, OBJ_RECTANGLE_LABEL, 0, xPos, yPos, xPos + panelWidth, yPos + panelHeight);
        ObjectSetInteger(0, panelName, OBJPROP_BGCOLOR, clrBlack);
        ObjectSetInteger(0, panelName, OBJPROP_COLOR, clrDimGray); 
        ObjectSetInteger(0, panelName, OBJPROP_CORNER, DashboardCorner);
        ObjectSetInteger(0, panelName, OBJPROP_SELECTABLE, false);
        ObjectSetInteger(0, panelName, OBJPROP_BACK, true); 
        ObjectSetInteger(0, panelName, OBJPROP_BORDER_TYPE, BORDER_RAISED);
    } else { 
        ObjectSetInteger(0, panelName, OBJPROP_XDISTANCE, xPos);
        ObjectSetInteger(0, panelName, OBJPROP_YDISTANCE, yPos);
        ObjectSetInteger(0, panelName, OBJPROP_XSIZE, panelWidth);
        ObjectSetInteger(0, panelName, OBJPROP_YSIZE, panelHeight);
    }
    
    string fullLogoPath = TerminalInfoString(TERMINAL_PATH) + "\\MQL4\\Images\\" + LogoFilePath;
    bool logoExists = (StringLen(LogoFilePath) > 0 && FileIsExist(fullLogoPath)); 
    
    if (logoExists) {
        if (ObjectFind(0, logoName) < 0) {
            ObjectCreate(0, logoName, OBJ_BITMAP_LABEL, 0, xPos + 5, yPos + 5);
            ObjectSetString(0, logoName, OBJPROP_BMPFILE, "images\\" + LogoFilePath); // Aggiunto "images\\"
            ObjectSetInteger(0, logoName, OBJPROP_XSIZE, 40); 
            ObjectSetInteger(0, logoName, OBJPROP_YSIZE, 40);
            ObjectSetInteger(0, logoName, OBJPROP_CORNER, DashboardCorner);
            ObjectSetInteger(0, logoName, OBJPROP_SELECTABLE, false);
            ObjectSetInteger(0, logoName, OBJPROP_BACK, false);
        } else {
            ObjectSetInteger(0, logoName, OBJPROP_XDISTANCE, xPos + 5);
            ObjectSetInteger(0, logoName, OBJPROP_YDISTANCE, yPos + 5);
        }
    } else {
        ObjectDelete(0, logoName); 
    }

    if (Day() != lastDay) {
        dailyProfit = 0; 
        initialBalance = AccountBalance(); 
        lastDay = Day();
        for(int k = 0; k < OrdersHistoryTotal(); k++) { 
            if(OrderSelect(k, SELECT_BY_POS, MODE_HISTORY) && OrderMagicNumber() == MagicNumber && OrderSymbol() == Symbol() && OrderCloseTime() >= iTime(Symbol(), PERIOD_D1, 0)) {
                dailyProfit += OrderProfit();
            }
        }
    }
    
    double currentFloatingProfit = 0;
    int buyOrders = 0;
    int sellOrders = 0;
    int totalEaOrders = 0;

    for(int i = 0; i < OrdersTotal(); i++) {
        if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderMagicNumber() == MagicNumber && OrderSymbol() == Symbol()) {
            totalEaOrders++;
            currentFloatingProfit += OrderProfit();
            if (OrderType() == OP_BUY) buyOrders++;
            else if (OrderType() == OP_SELL) sellOrders++;
        }
    }
    
    totalTrades = 0;
    winningTrades = 0;
    losingTrades = 0;
    totalProfit = 0;
    
    for(int i = 0; i < OrdersHistoryTotal(); i++) {
        if(OrderSelect(i, SELECT_BY_POS, MODE_HISTORY) && OrderMagicNumber() == MagicNumber && OrderSymbol() == Symbol()) {
            totalTrades++;
            totalProfit += OrderProfit();
            if (OrderProfit() > 0) winningTrades++;
            else if (OrderProfit() < 0) losingTrades++;
        }
    }

    double winRate = (totalTrades > 0) ? (double)winningTrades / totalTrades * 100.0 : 0.0;
    double profitFactor = 0.0;
    double grossProfit = 0.0;
    double grossLoss = 0.0;

    for(int i = 0; i < OrdersHistoryTotal(); i++) {
        if(OrderSelect(i, SELECT_BY_POS, MODE_HISTORY) && OrderMagicNumber() == MagicNumber && OrderSymbol() == Symbol()) {
            if (OrderProfit() > 0) grossProfit += OrderProfit();
            else grossLoss += OrderProfit();
        }
    }
    
    if (grossLoss < 0) { 
        profitFactor = MathAbs(grossProfit / grossLoss);
    } else if (grossLoss == 0 && grossProfit > 0) {
        profitFactor = 9999999.0; 
    } else {
        profitFactor = 0.0;
    }


    double buySentiment = (totalEaOrders > 0) ? (double)buyOrders / totalEaOrders * 100.0 : 0.0;
    double sellSentiment = (totalEaOrders > 0) ? (double)sellOrders / totalEaOrders * 100.0 : 0.0;
    
    string info = "--- STS QUANTUM v" + EA_VERSION + " ---\n"; 
    info += "--------------------------------------\n";
    info += "Symbol: " + Symbol() + " (" + EnumToString((ENUM_TIMEFRAMES)Period()) + ")\n";
    
    double balance = AccountBalance();
    double equity = AccountEquity();
    
    string balanceStr = "Balance: " + DoubleToString(balance, 2);
    string equityStr = "Equity:  " + DoubleToString(equity, 2);
    string currentPLStr = "Floating P/L: " + DoubleToString(currentFloatingProfit, 2);
    string dailyPLStr = "Daily P/L: " + DoubleToString(dailyProfit + currentFloatingProfit, 2); 
    if (MaxDailyLossPercent > 0) dailyPLStr += " (Max Loss " + DoubleToString(MaxDailyLossPercent, 1) + "%)";
    if (DailyProfitTargetPercent > 0) dailyPLStr += " (Target " + DoubleToString(DailyProfitTargetPercent, 1) + "%)";

    info += balanceStr + "\n";
    info += equityStr + "\n";
    info += currentPLStr + "\n";
    info += dailyPLStr + "\n";
    
    info += "Spread: " + DoubleToString((Ask - Bid)/Point, 1) + " pips (Max: " + IntegerToString(MaxSpreadPips) + ")\n";
    
    long secondsToClose = PeriodSeconds((ENUM_TIMEFRAMES)Period()) - (TimeCurrent() % PeriodSeconds((ENUM_TIMEFRAMES)Period()));
    string timeToCloseStr;
    if (secondsToClose < 60) timeToCloseStr = IntegerToString(secondsToClose) + " sec";
    else timeToCloseStr = IntegerToString((int)(secondsToClose / 60)) + " min " + IntegerToString(secondsToClose % 60) + " sec";
    info += "Time to bar close: " + timeToCloseStr + "\n"; 
    
    info += "AutoTrading: " + (IsTradeAllowed() ? "ON" : "OFF") + " (Terminal: " + (IsConnected() ? "Connected" : "Disconnected") + ")\n";
    info += "EA Orders: " + IntegerToString(totalEaOrders) + " (Buy:" + DoubleToString(buySentiment,0) + "% | Sell:" + DoubleToString(sellSentiment,0) + "%)\n";

    info += "\n--- Performance Metrics ---\n";
    info += "Total Trades: " + IntegerToString(totalTrades) + "\n";
    info += "Winning Trades: " + IntegerToString(winningTrades) + " (" + DoubleToString(winRate, 1) + "%)\n";
    info += "Losing Trades: " + IntegerToString(losingTrades) + "\n";
    info += "Total P/L: " + DoubleToString(totalProfit, 2) + "\n";
    info += "Profit Factor: " + DoubleToString(profitFactor, 2) + "\n";
    info += "Risk/Trade: " + DoubleToString(RiskPerTradePercent, 1) + "% (" + DoubleToString(CalculateLotSize(StopLossPips), 2) + " Lots)" + "\n";

    info += "\n--- Strategies Status ---\n";
    info += "TrendFollow: " + (EnableStrategyTrendFollow ? "ON" : "OFF") + "\n";
    info += "Scalping:    " + (EnableStrategyScalping ? "ON" : "OFF") + "\n";
    info += "Sniper:      " + (EnableStrategySniper ? "ON" : "OFF") + "\n";
    info += "Reversal(Engulf): " + (EnableStrategyReversalEngulfing ? "ON" : "OFF") + "\n";
    info += "Reversal(Pattern): " + (EnableStrategyReversalPattern ? "ON" : "OFF") + "\n";
    info += "Volatility Breakout: " + (EnableStrategyVolatilityBreakout ? "ON" : "OFF") + "\n";
    info += "Divergence: " + (EnableStrategyDivergence ? "ON" : "OFF") + "\n";
    info += "Dynamic Grid: " + (EnableStrategyDynamicGrid ? "ON" : "OFF") + "\n";
    info += "\nPowered by OpenAI GPT-4 Quantum";

    if (ObjectFind(0, textName) < 0) {
        ObjectCreate(0, textName, OBJ_TEXT, 0, xPos + 5, yPos + (logoExists ? 45 : 10)); 
        ObjectSetInteger(0, textName, OBJPROP_COLOR, DashboardTextColor);
        ObjectSetInteger(0, textName, OBJPROP_FONTSIZE, 8);
        ObjectSetString(0, textName, OBJPROP_TEXT, info);
        ObjectSetInteger(0, textName, OBJPROP_CORNER, DashboardCorner);
        ObjectSetInteger(0, textName, OBJPROP_SELECTABLE, false);
        ObjectSetInteger(0, textName, OBJPROP_BACK, false);
    } else {
        ObjectSetString(0, textName, OBJPROP_TEXT, info);
        ObjectSetInteger(0, textName, OBJPROP_XDISTANCE, xPos + 5);
        ObjectSetInteger(0, textName, OBJPROP_YDISTANCE, yPos + (logoExists ? 45 : 10));
    }
}

// Funzione di supporto: Pulizia disegni vecchi (rimuove tutti gli oggetti creati dall'EA)
void ClearAllEAObjects()
{
   string objectsToDelete[];
   int count = 0;

   for(int i = ObjectsTotal(0, 0) - 1; i >= 0; i--) 
   {
      string name = ObjectName(0, i);
      if(StringFind(name, "STS_") == 0) 
      {
         ArrayResize(objectsToDelete, count + 1);
         objectsToDelete[count] = name;
         count++;
      }
   }
   
   for(int i = 0; i < count; i++) 
   {
      ObjectDelete(0, objectsToDelete[i]);
   }
}

// Funzione per determinare la sessione di trading (GMT)
string GetTradingSession() {
    int hour = TimeHour(TimeCurrent());
    if (hour >= 0 && hour < 8) return "Asian"; 
    if (hour >= 8 && hour < 16) return "London"; 
    if (hour >= 13 && hour < 22) return "New York"; 
    return "Other";
}


//+------------------------------------------------------------------+
//| LOGICHE DELLE STRATEGIE DI TRADING                               |
//+------------------------------------------------------------------+

// Strategia Trend-Follow: RSI + Moving Average + Filtro ADX + EMA Lenta
void StrategyTrendFollow()
{
    if(!EnableStrategyTrendFollow) return;
    if(!CanOpenTradeStrategy(lastTradeTimeTrendFollow)) return;

    if (Bars < MathMax(51, TF_SlowEMAPeriod + 1)) return;   
    
    double rsi_current = iRSI(Symbol(), 0, 14, PRICE_CLOSE, 0);
    double ma_current = iMA(Symbol(), 0, 50, 0, MODE_SMA, PRICE_CLOSE, 0);
    double rsi_prev = iRSI(Symbol(), 0, 14, PRICE_CLOSE, 1);
    double ma_prev = iMA(Symbol(), 0, 50, 0, MODE_SMA, PRICE_CLOSE, 1);
    
    double adx_main = iADX(Symbol(), 0, TF_ADXPeriod, PRICE_CLOSE, MODE_MAIN, 1); 
    double adx_prev = iADX(Symbol(), 0, TF_ADXPeriod, PRICE_CLOSE, MODE_MAIN, 2); 
    
    bool adx_filter_passed = (TF_MinADXValue == 0 || adx_main >= TF_MinADXValue);
    if (TF_ADXIncreasing) adx_filter_passed = adx_filter_passed && (adx_main > adx_prev); 
    
    double ema_slow_current = iMA(Symbol(), 0, TF_SlowEMAPeriod, 0, MODE_EMA, PRICE_CLOSE, 0);
    double ema_slow_prev = iMA(Symbol(), 0, TF_SlowEMAPeriod, 0, MODE_EMA, PRICE_CLOSE, 1);

    if(Close[0] > ma_current && Close[1] <= ma_prev && 
       rsi_current > 50 && rsi_prev <= 50 && 
       Close[0] > ema_slow_current && Close[1] > ema_slow_prev && 
       adx_filter_passed) 
    {
        OpenOrder(OP_BUY, "TF Buy", lastTradeTimeTrendFollow);
    }
    else if(Close[0] < ma_current && Close[1] >= ma_prev && 
            rsi_current < 50 && rsi_prev >= 50 && 
            Close[0] < ema_slow_current && Close[1] < ema_slow_prev && 
            adx_filter_passed)
    {
        OpenOrder(OP_SELL, "TF Sell", lastTradeTimeTrendFollow);
    }
}

// Strategia Scalping: Basata su Volume (Tick Volume) e Price Action
void StrategyScalping()
{
    if(!EnableStrategyScalping) return;
    if(!CanOpenTradeStrategy(lastTradeTimeScalping)) return;

    if (Bars < 5) return; 

    int currentHour = TimeHour(TimeCurrent());
    if (currentHour < Scalping_StartHour || currentHour >= Scalping_EndHour) {
        return;
    }
    
    if (Scalping_LimitToLondonNewYork) {
        string session = GetTradingSession();
        if (session != "London" && session != "New York") {
            return; 
        }
    }

    double avgTickVolume = (Volume[1] + Volume[2] + Volume[3] + Volume[4] + Volume[5]) / 5.0;
    
    bool highVolume = (Volume[0] >= avgTickVolume * Scalping_MinTickVolumeRatio);

    bool strongBullish = (Close[0] > Open[0] && (Close[0] - Open[0]) / (High[0] - Low[0]) > 0.7); 
    if (strongBullish && highVolume) {
        if (Open[1] > Close[1] && Open[0] < Close[1] && Close[0] > Open[1]) {
            OpenOrder(OP_BUY, "Scalping Buy", lastTradeTimeScalping);
        }
    }
    
    bool strongBearish = (Open[0] > Close[0] && (Open[0] - Close[0]) / (High[0] - Low[0]) > 0.7); 
    if (strongBearish && highVolume) {
        if (Close[1] > Open[1] && Open[0] > Close[1] && Close[0] < Open[1]) {
            OpenOrder(OP_SELL, "Scalping Sell", lastTradeTimeScalping);
        }
    }
}

// Strategia Sniper: Basata su MACD e Stochastic Oscillator con filtro EMA, CCI, DeMarker
void StrategySniper()
{
    if(!EnableStrategySniper) return;
    if(!CanOpenTradeStrategy(lastTradeTimeSniper)) return;

    if (Bars < MathMax(MathMax(100, 26), MathMax(Sniper_CCI_Period, Sniper_DeMarker_Period) + 1)) return; 

    if ((Ask - Bid) / Point > Sniper_MaxSpreadPips) {
        return;
    }

    double macd_main = iMACD(Symbol(), 0, 12, 26, 9, PRICE_CLOSE, MODE_MAIN, 0);
    double macd_signal = iMACD(Symbol(), 0, 12, 26, 9, PRICE_CLOSE, MODE_SIGNAL, 0);
    double macd_prev_main = iMACD(Symbol(), 0, 12, 26, 9, PRICE_CLOSE, MODE_MAIN, 1);
    double macd_prev_signal = iMACD(Symbol(), 0, 12, 26, 9, PRICE_CLOSE, MODE_SIGNAL, 1);

    double stoch_main = iStochastic(Symbol(), 0, 5, 3, 3, MODE_SMA, 0, MODE_MAIN, 0); 
    double stoch_signal = iStochastic(Symbol(), 0, 5, 3, 3, MODE_SMA, 0, MODE_SIGNAL, 0); 
    double stoch_prev_main = iStochastic(Symbol(), 0, 5, 3, 3, MODE_SMA, 0, MODE_SIGNAL, 1);

    double ema_slow_current = iMA(Symbol(), 0, Sniper_SlowEMAPeriod, 0, MODE_EMA, PRICE_CLOSE, 0);
    double ema_slow_prev = iMA(Symbol(), 0, Sniper_SlowEMAPeriod, 0, MODE_EMA, PRICE_CLOSE, 1);

    double cci_current = iCCI(Symbol(), 0, Sniper_CCI_Period, PRICE_TYPICAL, 0);
    double cci_prev = iCCI(Symbol(), 0, Sniper_CCI_Period, PRICE_TYPICAL, 1);

    double dem_current = iDeMarker(Symbol(), 0, Sniper_DeMarker_Period, 0);
    double dem_prev = iDeMarker(Symbol(), 0, Sniper_DeMarker_Period, 1);

    bool buySignal = false;
    bool sellSignal = false;

    if (macd_main > macd_signal && macd_prev_main <= macd_prev_signal) { 
        if (stoch_main > 20 && stoch_prev_main <= 20) { 
            if (!Sniper_UseSlowEMAFilter || (Close[0] > ema_slow_current && Close[1] > ema_slow_prev)) { 
                if (cci_current > -100 && cci_prev <= -100) { 
                    if (dem_current > 0.5 && dem_prev <= 0.5) { 
                        buySignal = true;
                    }
                }
            }
        }
    }

    if (macd_main < macd_signal && macd_prev_main >= macd_prev_signal) { 
        if (stoch_main < 80 && stoch_prev_main >= 80) { 
            if (!Sniper_UseSlowEMAFilter || (Close[0] < ema_slow_current && Close[1] < ema_slow_prev)) { 
                if (cci_current < 100 && cci_prev >= 100) { 
                    if (dem_current < 0.5 && dem_prev >= 0.5) { 
                        sellSignal = true;
                    }
                }
            }
        }
    }

    if (buySignal) {
        OpenOrder(OP_BUY, "Sniper Buy", lastTradeTimeSniper);
    } else if (sellSignal) {
        OpenOrder(OP_SELL, "Sniper Sell", lastTradeTimeSniper);
    }
}

// Strategia Reversal (Engulfing): Pattern Engulfing vicino a S/R
void StrategyReversalEngulfing()
{
    if(!EnableStrategyReversalEngulfing) return;
    if(!CanOpenTradeStrategy(lastTradeTimeReversalEngulfing)) return;

    if (Bars < 2) return;

    double s1 = CalculateSupport1(0);
    double r1 = CalculateResistance1(0);
    
    bool nearSupport = (s1 > 0 && MathAbs(Low[1] - s1) / Point <= Reversal_ProximityToSR);
    bool nearResistance = (r1 > 0 && MathAbs(High[1] - r1) / Point <= Reversal_ProximityToSR);

    if (IsBullishEngulfing(1) && nearSupport) {
        OpenOrder(OP_BUY, "Rev Engulf Buy", lastTradeTimeReversalEngulfing);
    }
    else if (IsBearishEngulfing(1) && nearResistance) {
        OpenOrder(OP_SELL, "Rev Engulf Sell", lastTradeTimeReversalEngulfing);
    }
}

// Strategia Reversal (Patterns): Hammer/Shooting Star vicino a S/R
void StrategyReversalPattern()
{
    if(!EnableStrategyReversalPattern) return;
    if(!CanOpenTradeStrategy(lastTradeTimeReversalPattern)) return;

    if (Bars < 2) return;

    double s1 = CalculateSupport1(0);
    double r1 = CalculateResistance1(0);
    
    bool nearSupport = (s1 > 0 && MathAbs(Low[1] - s1) / Point <= Reversal_ProximityToSR);
    bool nearResistance = (r1 > 0 && MathAbs(High[1] - r1) / Point <= Reversal_ProximityToSR);

    if (IsHammer(1) && nearSupport) {
        OpenOrder(OP_BUY, "Rev Pattern Buy", lastTradeTimeReversalPattern);
    }
    else if (IsShootingStar(1) && nearResistance) {
        OpenOrder(OP_SELL, "Rev Pattern Sell", lastTradeTimeReversalPattern);
    }
}

// NUOVA Strategia: Volatility Breakout (per XAUUSD)
void StrategyVolatilityBreakout()
{
    if (!EnableStrategyVolatilityBreakout) return;
    if (!CanOpenTradeStrategy(lastTradeTimeVolatilityBreakout)) return;

    if (Bars < VB_LookbackPeriod + 2) return; 

    double atr = iATR(Symbol(), VB_ATR_Timeframe, VB_ATR_Period, 1); 
    if (atr == 0) return; 

    double avgRange = 0;
    for (int i = 1; i <= VB_LookbackPeriod; i++) {
        avgRange += (iHigh(Symbol(), 0, i) - iLow(Symbol(), 0, i));
    }
    avgRange /= VB_LookbackPeriod;

    double prevDayHigh = iHigh(Symbol(), VB_ATR_Timeframe, 1);
    double prevDayLow = iLow(Symbol(), VB_ATR_Timeframe, 1);

    double breakoutBuyLevel = prevDayHigh + atr * VB_Multiplier;
    double breakoutSellLevel = prevDayLow - atr * VB_Multiplier;
    
    if (Close[0] > breakoutBuyLevel && Open[0] <= breakoutBuyLevel) { 
        if (!HasOpenOrder(OP_BUY, "VB Buy")) { 
             OpenOrder(OP_BUY, "VB Buy", lastTradeTimeVolatilityBreakout);
        }
    }
    else if (Close[0] < breakoutSellLevel && Open[0] >= breakoutSellLevel) { 
        if (!HasOpenOrder(OP_SELL, "VB Sell")) { 
             OpenOrder(OP_SELL, "VB Sell", lastTradeTimeVolatilityBreakout);
        }
    }
}

// Funzione helper per trovare swing points (per divergenze)
int FindSwingHigh(int startBar, int barsBack) {
    double highest = -1;
    int highestBar = -1;
    for (int i = startBar; i < startBar + barsBack && i < Bars; i++) {
        if (High[i] > highest) {
            highest = High[i];
            highestBar = i;
        }
    }
    return highestBar;
}

int FindSwingLow(int startBar, int barsBack) {
    double lowest = 99999999;
    int lowestBar = -1;
    for (int i = startBar; i < startBar + barsBack && i < Bars; i++) {
        if (Low[i] < lowest) {
            lowest = Low[i];
            lowestBar = i;
        }
    }
    return lowestBar;
}

// NUOVA Strategia: Divergenze RSI/MACD
void StrategyDivergence()
{
    if (!EnableStrategyDivergence) return;
    if (!CanOpenTradeStrategy(lastTradeTimeDivergence)) return;

    if (Bars < MathMax(Div_MaxBarsBack + 2, MathMax(Div_RSI_Period, MathMax(Div_MACD_Fast, Div_MACD_Slow)) + 2)) return;

    int currentBar = 1; 
    
    int prevLowBar = FindSwingLow(currentBar + 1, Div_MaxBarsBack);
    if (prevLowBar != -1 && Low[currentBar] < Low[prevLowBar] && MathAbs(Low[currentBar] - Low[prevLowBar]) / Point >= Div_MinSwingSize) {
        double rsi_current = iRSI(Symbol(), 0, Div_RSI_Period, PRICE_CLOSE, currentBar);
        double rsi_prev = iRSI(Symbol(), 0, Div_RSI_Period, PRICE_CLOSE, prevLowBar);
        if (rsi_current > rsi_prev) { 
            OpenOrder(OP_BUY, "Div RSI Buy", lastTradeTimeDivergence);
            return;
        }

        double macd_main_current = iMACD(Symbol(), 0, Div_MACD_Fast, Div_MACD_Slow, Div_MACD_Signal, PRICE_CLOSE, MODE_MAIN, currentBar);
        double macd_main_prev = iMACD(Symbol(), 0, Div_MACD_Fast, Div_MACD_Slow, Div_MACD_Signal, PRICE_CLOSE, MODE_MAIN, prevLowBar);
        if (macd_main_current > macd_main_prev) { 
            OpenOrder(OP_BUY, "Div MACD Buy", lastTradeTimeDivergence);
            return;
        }
    }

    int prevHighBar = FindSwingHigh(currentBar + 1, Div_MaxBarsBack);
    if (prevHighBar != -1 && High[currentBar] > High[prevHighBar] && MathAbs(High[currentBar] - High[prevHighBar]) / Point >= Div_MinSwingSize) {
        double rsi_current = iRSI(Symbol(), 0, Div_RSI_Period, PRICE_CLOSE, currentBar);
        double rsi_prev = iRSI(Symbol(), 0, Div_RSI_Period, PRICE_CLOSE, prevHighBar);
        if (rsi_current < rsi_prev) { 
            OpenOrder(OP_SELL, "Div RSI Sell", lastTradeTimeDivergence);
            return;
        }

        double macd_main_current = iMACD(Symbol(), 0, Div_MACD_Fast, Div_MACD_Slow, Div_MACD_Signal, PRICE_CLOSE, MODE_MAIN, currentBar);
        double macd_main_prev = iMACD(Symbol(), 0, Div_MACD_Fast, Div_MACD_Slow, Div_MACD_Signal, PRICE_CLOSE, MODE_MAIN, prevHighBar);
        if (macd_main_current < macd_main_prev) { 
            OpenOrder(OP_SELL, "Div MACD Sell", lastTradeTimeDivergence);
            return;
        }
    }
}

// NUOVA Strategia: Dynamic Grid (Hedging in range)
void StrategyDynamicGrid()
{
    if (!EnableStrategyDynamicGrid) return;
    if (!CanOpenTradeStrategy(lastTradeTimeDynamicGrid)) return;
    
    if (!Grid_AllowNewGrid && HasOpenOrder(OP_BUY, "Grid") && HasOpenOrder(OP_SELL, "Grid")) return;

    if ((Ask - Bid) / Point > Grid_MaxSpreadPips) {
        return;
    }

    int buyGridOrders = 0;
    int sellGridOrders = 0;
    int currentTotalGridOrders = 0;

    for(int i=OrdersTotal()-1; i>=0; i--) {
        if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderMagicNumber() == MagicNumber && StringFind(OrderComment(), "Grid") == 0) {
            currentTotalGridOrders++;
            if (OrderType() == OP_BUY) buyGridOrders++;
            else if (OrderType() == OP_SELL) sellGridOrders++;
        }
    }
    
    if (currentTotalGridOrders == 0) {
        if (Grid_InitialOrders > 0) {
            for (int i = 0; i < Grid_InitialOrders; i++) {
                // Modifica: ora la funzione OpenOrder determina il lotto in base a RiskPerTradePercent o Lots
                OpenOrder(OP_BUY, "Grid Buy", lastTradeTimeDynamicGrid); 
                if (Grid_UseHedging) OpenOrder(OP_SELL, "Grid Sell", lastTradeTimeDynamicGrid);
            }
        }
        return; 
    }

    double lastBuyPrice = 0;
    double lastSellPrice = 999999;
    int lastBuyTicket = 0;
    int lastSellTicket = 0;

    for(int i=OrdersTotal()-1; i>=0; i--) {
        if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderMagicNumber() == MagicNumber && StringFind(OrderComment(), "Grid") == 0) {
            if (OrderType() == OP_BUY) {
                if (OrderOpenPrice() > lastBuyPrice) { 
                    lastBuyPrice = OrderOpenPrice();
                    lastBuyTicket = OrderTicket();
                }
            } else if (OrderType() == OP_SELL) {
                if (OrderOpenPrice() < lastSellPrice) { 
                    lastSellPrice = OrderOpenPrice();
                    lastSellTicket = OrderTicket();
                }
            }
        }
    }

    if (Grid_MaxOrders > buyGridOrders && Ask < lastBuyPrice - Grid_SpacingPips * Point) {
        // Modifica: ora la funzione OpenOrder determina il lotto in base a RiskPerTradePercent o Lots
        // Per Dynamic Grid, il calcolo del lotto moltiplicato andrebbe gestito dentro CalculateLotSize o una funzione dedicata
        // Se si vuole usare il moltiplicatore Grid_LotMultiplier, allora RiskPerTradePercent deve essere 0
        OpenOrder(OP_BUY, "Grid Buy", lastTradeTimeDynamicGrid); 
    }
    if (Grid_MaxOrders > sellGridOrders && Bid > lastSellPrice + Grid_SpacingPips * Point) {
        OpenOrder(OP_SELL, "Grid Sell", lastTradeTimeDynamicGrid);
    }

    double gridProfit = 0;
    bool hasGridOrder = false;
    for(int i=OrdersTotal()-1; i>=0; i--) {
        if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderMagicNumber() == MagicNumber && StringFind(OrderComment(), "Grid") == 0) {
            gridProfit += OrderProfit();
            hasGridOrder = true;
        }
    }
    
    if (hasGridOrder && gridProfit >= Grid_TakeProfitPips * Point) {
        Print("Grid Take Profit raggiunto! Chiusura di tutti gli ordini della griglia.");
        for(int i=OrdersTotal()-1; i>=0; i--) {
            if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderMagicNumber() == MagicNumber && StringFind(OrderComment(), "Grid") == 0) {
                OrderClose(OrderTicket(), OrderLots(), (OrderType() == OP_BUY) ? Bid : Ask, Slippage, clrGreen);
            }
        }
        lastTradeTimeDynamicGrid = TimeCurrent(); 
    }
}


//+------------------------------------------------------------------+
//| FUNZIONI EVENTI STANDARD MQL4                                    |
//+------------------------------------------------------------------+

// OnInit - Inizializzazione dell'Expert Advisor
int OnInit()
{
   EventSetMillisecondTimer(1000); 
   ClearAllEAObjects(); 
   
   initialBalance = AccountBalance();
   lastDay = Day();

   DrawDashboard(); 
   if (!IsTesting() || !DisableGraphAnalysisInTester) { 
        DrawAnalysisObjects();
        DrawCandlestickPatterns();
   }

   return(INIT_SUCCEEDED);
}

// OnDeinit - Deinizializzazione dell'Expert Advisor
void OnDeinit(const int reason)
{
   EventKillTimer(); 
   ClearAllEAObjects(); 
}

// OnTick - Evento principale chiamato ad ogni nuovo tick di prezzo
void OnTick()
{
    CheckDailyProfitLoss(); 
    if (TimeCurrent() - lastGlobalTradeTime < MinTimeBetweenTradesMinutes * 60) {
        if (ShowDashboard) DrawDashboard(); 
        return;
    }

    ManageOrders();

    if (lastBarTime != Time[0])
    {
        if (Bars < MathMax(TF_SlowEMAPeriod + 200, MathMax(VB_LookbackPeriod + 2, Div_MaxBarsBack + 2))) { 
            if (ShowDashboard) DrawDashboard(); 
            return;
        }

        if(IsStopped() || !IsTradeAllowed() || !IsConnected()) {
            if (ShowDashboard) DrawDashboard(); 
            return;
        }
        
        StrategyTrendFollow();
        StrategyScalping(); 
        StrategySniper();
        StrategyReversalEngulfing();
        StrategyReversalPattern();
        StrategyVolatilityBreakout(); 
        StrategyDivergence();         
        StrategyDynamicGrid();        

        DrawDashboard();
        if (!IsTesting() || !DisableGraphAnalysisInTester) { 
            DrawAnalysisObjects();
            DrawCandlestickPatterns();
        }

        lastBarTime = Time[0]; 
    }
    
    if (ShowDashboard) DrawDashboard(); 
}

// OnTimer - Evento chiamato dal timer (ogni secondo) per aggiornamenti dinamici
void OnTimer()
{
    if (ShowDashboard) {
        DrawDashboard(); 
    }
}

// OnTrade - Evento chiamato quando un'operazione di trading è avvenuta (apertura, chiusura, modifica)
void OnTrade()
{
    if (ShowDashboard) {
        DrawDashboard(); 
    }
}
