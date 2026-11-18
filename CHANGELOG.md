# Changelog

Tutte le modifiche rilevanti della libreria. Formato ispirato a [Keep a Changelog](https://keepachangelog.com/it/1.1.0/);
le versioni seguono lo schema `MAJOR.MINOR.PATCH` (MAJOR = modifiche incompatibili per chi usa la libreria).

## [0.1.0] - 2026-11-18

Prima versione con convenzioni di nome uniformi, test e documentazione. **Incompatibile** con lo stato
precedente (nomi degli oggetti e dei parametri cambiati).

### Aggiunto
- Progetto di test `MyLibrary_Test` (TcUnit) con una test suite per ogni oggetto della libreria.
- Enumerazione `E_Ascii` con i codici dei caratteri più usati, da passare a `FB_StringBuilder.AddChar`.
- `FB_StringBuilder.AddByte(byValue)` per aggiungere un byte qualsiasi.
- `README.md` (contenuto, convenzioni di nome, installazione, test) e questo `CHANGELOG.md`.
- Versione del progetto della libreria impostata a 0.1.0.0.

### Modificato
- Nomi uniformati (vedi README, "Convenzioni di nome"):
  - `FB_stringBuilder` → `FB_StringBuilder`, `FB_sweep` → `FB_Sweep`, `FB_timer` → `FB_Timer`;
  - `F_atan2` → `F_Atan2`, `F_interpolateReal` → `F_InterpolateReal`, `F_interpolateTime` → `F_InterpolateTime`,
    `F_saturation` → `F_Saturation`, `F_seconds2time` → `F_SecondsToTime`, `F_time2seconds` → `F_TimeToSeconds`;
  - `CONSTANTS` → `GVL_Constants`, `CONVERSIONS` → `GVL_Conversions` (`rTime2Seconds` → `TIME_TO_SECONDS`,
    `rSeconds2Time` → `SECONDS_TO_TIME`), `PARAMETERs` → `GVL_Parameters` (`uiMaxCharacters` → `MAX_CHARACTERS`);
  - variabili interne di `FB_Sweep` (`rStartFrequency`, `rEndFrequency`, `rDuration`, `rSlope`; ingresso
    `rInputLength` → `rInputDuration`) e di `FB_StringBuilder` (`abyBuffer`, `abyBufferOld`);
  - campi di `ST_TypeSizes` (`uiSint`, `uiInt`, ...; aggiunto `uiSweep`).
- `F_InterpolateReal` / `F_InterpolateTime`: parametri rinominati in `X0, X1, Y0, Y1`; se `X0 = X1` restituiscono
  `Y0` invece di 0; la saturazione fuori intervallo funziona anche con `X0 > X1`.
- `FB_StringBuilder.AddChar` accetta un `E_Ascii` invece di un `BYTE`; le 13 action `AddComma`, `AddColon`,
  `AddSpace`, ... sono sostituite da `AddChar(E_Ascii.Comma)` ecc.
- Valori di ritorno BOOL resi coerenti (TRUE = operazione eseguita):
  - `FB_StringBuilder.AddString` e `AddKeyValue` sono "tutto o niente" e restituiscono FALSE anche con puntatore nullo;
  - `FB_StringBuilder.BuildString` riceve la dimensione della destinazione (`udiStringSize`), restituisce FALSE se
    la destinazione è troppo piccola o nulla (prima poteva scrivere oltre la fine della stringa) e non modifica più
    il buffer, quindi si può chiamare più volte;
  - `FB_StringBuilder.DeleteChars` restituisce FALSE se c'erano meno caratteri di quelli richiesti;
  - `FB_Sweep.Initialize` restituisce FALSE con durata non positiva (prima divideva per zero);
  - `FB_Timer.ResetCondition` restituisce TRUE se il reset è stato eseguito;
  - `FB_Timer.ChangeTimer`, `FB_Timer.ResetWithOffset` e `FB_StringBuilder.IncrementIndexVariable` non
    restituiscono più nulla (prima restituivano sempre FALSE).
- `FB_StringBuilder`: la capacità è esattamente `MAX_CHARACTERS` caratteri, uguale per `AddChar` e `AddString`
  (prima i due controlli differivano di un carattere); `ResetBuffer` salva in `abyBufferOld` tutto il testo.
- `F_Atan2`: tolte le conversioni `LREAL_TO_REAL` inutili (PI è REAL).

### Rimosso
- Action `AddNullChar` di `FB_StringBuilder`: il terminatore viene scritto da `BuildString`
  (se serve un carattere nullo nel testo: `AddChar(E_Ascii.Null)`).
- Puntatori `pbBuffer` / `pbBufferOld` inizializzati con `ADR()` nella dichiarazione: dopo un online change
  potevano puntare alla vecchia posizione del blocco. Ora si usa `ADR(abyBuffer)` direttamente.

### Corretto
- `FB_Sweep`: istanza del timer dichiarata come `FB_Timer` (era `FB_Timer(T#0S)`, che richiede un `FB_init`).

## [0.0.0] - 2026-11-18

- Prima stesura: `FB_stringBuilder`, `FB_sweep`, `FB_timer`, funzioni matematiche e di conversione, GVL di costanti
  e parametri.
