# MyLibrary

Libreria TwinCAT 3 (Structured Text) con funzioni e function block riutilizzabili nei progetti PLC.

| Proprietà | Valore |
|---|---|
| Nome (placeholder) | `MyLibrary` |
| Namespace | `ML` (es. `ML.F_Saturation(...)`) |
| Company | `FabioBolognesi` |
| Versione | 0.1.0.0 (vedi [CHANGELOG.md](CHANGELOG.md)) |
| TwinCAT | 3.1.4024 o successivo |
| Dipendenze | `Tc2_Standard`, `Tc2_System`, `Tc2_Utilities`, `Tc3_Module` |

## Struttura del repository

```
MyLibrary.sln                     Solution TwinCAT (aprire questa)
MyLibrary/
├── MyLibrary.tsproj              Progetto di sistema TwinCAT
├── MyLibrary/                    PROGETTO DELLA LIBRERIA
│   ├── MyLibrary.plcproj
│   ├── ENUMERATIONs/             E_...   enumerazioni
│   ├── FUNCTIONs/                F_...   funzioni
│   ├── FUNCTION_BLOCKs/          FB_...  function block
│   ├── GLOBAL_VARIABLEs/         GVL_... costanti e parameter list
│   └── STRUCTUREs/               ST_...  strutture
└── MyLibrary_Test/               PROGETTO DI TEST (TcUnit), non fa parte della libreria
    ├── MyLibrary_Test.plcproj
    ├── PlcTask.TcTTO             task dei test (10 ms) che chiama MAIN
    ├── PROGRAMs/MAIN.TcPOU       istanzia le test suite e chiama TcUnit.RUN()
    └── TESTs/                    FB_<Oggetto>_Test, una test suite per ogni oggetto
README.md                         questo file
CHANGELOG.md                      storico delle modifiche
```

## Contenuto della libreria

### Funzioni

| Nome | Descrizione |
|---|---|
| `F_Atan2(rX, rY) : REAL` | Angolo [rad] del punto (rX, rY) nell'intervallo (-π, π]; 0 per l'origine. **Attenzione:** l'ordine è (X, Y), non (Y, X) come in C/Python: usare sempre i nomi dei parametri. |
| `F_InterpolateReal(rX0, rX1, rY0, rY1, rX) : REAL` | Interpolazione lineare tra (rX0, rY0) e (rX1, rY1). Fuori dall'intervallo satura al valore dell'estremo più vicino; funziona anche con rX0 > rX1; se rX0 = rX1 restituisce rY0. |
| `F_InterpolateTime(tX0, tX1, rY0, rY1, tX) : REAL` | Come `F_InterpolateReal`, con ascisse `TIME`. |
| `F_Saturation(rInput, rMinimumValue, rMaximumValue) : REAL` | Limita `rInput` nell'intervallo dato. |
| `F_SecondsToTime(rSeconds) : TIME` | Secondi (REAL) → `TIME`. |
| `F_TimeToSeconds(tTime) : REAL` | `TIME` → secondi (REAL). Precisione REAL: esatta fino a circa 4,6 ore. |

### Function block

**`FB_Timer`**: cronometro basato su `TIME()`. Si usa solo tramite metodi e action, senza chiamare il corpo del blocco.

| Membro | Descrizione |
|---|---|
| `Reset()` (action) | Fa ripartire il cronometro da zero. |
| `ResetWithOffset(tOffset)` | Riparte come se fosse già trascorso `tOffset`. |
| `ResetCondition(bCondition) : BOOL` | Reset solo se `bCondition`; restituisce TRUE se il reset è avvenuto. |
| `ChangeTimer(tTimerNew)` | Imposta la durata usata da `VerifySelfElapsed`. |
| `ComputeElapsed() : TIME` / `ComputeElapsedSeconds() : REAL` | Tempo trascorso dall'ultimo reset. |
| `VerifyElapsed(tInputTime) : BOOL` | TRUE se è trascorso più di `tInputTime`. |
| `VerifySelfElapsed() : BOOL` | TRUE se è trascorso più della durata impostata. |

**`FB_Sweep`**: sweep sinusoidale (chirp lineare) di ampiezza 1.

| Membro | Descrizione |
|---|---|
| `Initialize(rInputStartFrequency, rInputEndFrequency, rInputDuration) : BOOL` | Imposta frequenze [Hz] e durata [s]. Restituisce FALSE, senza modificare nulla, se la durata non è positiva. |
| `Start()` (action) | Fa partire lo sweep. |
| `Step() : REAL` | Valore attuale (tra -1 e +1; 0 a sweep finito). Da chiamare a ogni ciclo. |
| `ComputeFrequency() : REAL` | Frequenza istantanea [Hz]. |

**`FB_StringBuilder`**: costruisce stringhe lunghe fino a `GVL_Parameters.MAX_CHARACTERS` caratteri, aggiungendo un pezzo alla volta.

| Membro | Descrizione |
|---|---|
| `AddChar(eChar : E_Ascii) : BOOL` | Aggiunge un carattere dell'elenco `E_Ascii`, es. `AddChar(ML.E_Ascii.Comma)`. |
| `AddByte(byValue) : BOOL` | Aggiunge un byte qualsiasi. |
| `AddString(psString) : BOOL` | Aggiunge una stringa: `AddString(ADR(sTesto))`. |
| `AddKeyValue(psKey, psValue) : BOOL` | Aggiunge `chiave:valore,`. |
| `BuildString(psString, udiStringSize) : BOOL` | Copia il risultato, con terminatore, nella stringa di destinazione: `BuildString(ADR(sRisultato), SIZEOF(sRisultato))`. Non modifica il buffer. |
| `DeleteChars(udiNumChars) : BOOL` | Cancella gli ultimi caratteri; FALSE se ce n'erano meno di quelli richiesti (in tal caso li cancella tutti). |
| `ResetBuffer()` (action) | Salva il testo in un buffer di riserva e svuota il buffer. |
| `CheckLength()`, `CheckFinalLength(udiLen)`, `CheckMinimumLength()` | C'è spazio per 1 / per `udiLen` caratteri; il buffer non è vuoto. |

Esempio:

```
VAR
	fbBuilder	:	ML.FB_StringBuilder;
	sName		:	STRING	:=	'temperatura';
	sValue		:	STRING	:=	'21.5';
	sResult		:	STRING(255);
END_VAR

fbBuilder.AddChar(ML.E_Ascii.OpenBrace);
fbBuilder.AddKeyValue(ADR(sName), ADR(sValue));
fbBuilder.DeleteChars(1);								// tolgo l'ultima virgola
fbBuilder.AddChar(ML.E_Ascii.CloseBrace);
IF fbBuilder.BuildString(ADR(sResult), SIZEOF(sResult)) THEN
	// sResult = '{temperatura:21.5}'
END_IF
fbBuilder.ResetBuffer();
```

**Valori di ritorno BOOL.** In tutta la libreria un metodo che restituisce BOOL segue una di due regole:
- per le **azioni** (`Add...`, `BuildString`, `DeleteChars`, `Initialize`, `ResetCondition`): TRUE = eseguita, FALSE = non eseguita. Se un'azione `Add...` o `BuildString` restituisce FALSE, il buffer e la destinazione non sono stati modificati;
- per le **domande** (`Check...`, `Verify...`): la risposta alla domanda.

I metodi che non possono fallire (`ChangeTimer`, `ResetWithOffset`, `IncrementIndexVariable`) non restituiscono nulla.

### Tipi e variabili globali

| Nome | Descrizione |
|---|---|
| `E_Ascii` (BYTE) | Codici ASCII dei caratteri più usati (Comma, Colon, Space, OpenBrace, ...), da passare ad `AddChar`. |
| `ST_TypeSizes` | Dimensioni in byte dei tipi (strumento di debug da leggere online). |
| `GVL_Constants` | `PI`, `G` (accelerazione di gravità). |
| `GVL_Conversions` | `TIME_TO_SECONDS`, `SECONDS_TO_TIME`. |
| `GVL_Parameters` (parameter list) | `MAX_CHARACTERS` = 5000: dimensione del buffer di `FB_StringBuilder`. Il progetto che usa la libreria la può cambiare dal Library Manager. |

Tutte le GVL e `E_Ascii` sono `qualified_only`: si scrive sempre il nome completo, per esempio `ML.GVL_Constants.PI` o `ML.E_Ascii.Comma`.

## Convenzioni di nome

Ogni nuovo oggetto deve seguire queste regole, per mantenere la libreria uniforme.

**Oggetti (POU e tipi):** prefisso + nome in PascalCase, in inglese.

| Oggetto | Prefisso | Esempio |
|---|---|---|
| Funzione | `F_` | `F_InterpolateReal`, `F_TimeToSeconds` (conversioni con `To`, non `2`) |
| Function block | `FB_` | `FB_StringBuilder` |
| Enumerazione | `E_` | `E_Ascii` |
| Struttura | `ST_` | `ST_TypeSizes` |
| Interfaccia | `I_` | `I_Logger` |
| Lista di variabili globali | `GVL_` | `GVL_Constants` |
| Test suite (progetto di test) | `FB_<Oggetto>_Test` | `FB_StringBuilder_Test` |

**Metodi e action:** PascalCase, iniziano con un verbo (`AddChar`, `ComputeElapsed`, `VerifySelfElapsed`). I valori delle enumerazioni sono in PascalCase (`E_Ascii.OpenBrace`).

**Costanti** (`VAR_GLOBAL CONSTANT`, parameter list): MAIUSCOLO_CON_UNDERSCORE, senza prefisso di tipo (`PI`, `MAX_CHARACTERS`).

**Variabili:** prefisso del tipo in minuscolo + nome in PascalCase (`rStartFrequency`, `udiIndex`).

| Tipo | Prefisso | | Tipo | Prefisso |
|---|---|---|---|---|
| BOOL | `b` | | REAL | `r` |
| BYTE | `by` | | LREAL | `lr` |
| WORD / DWORD / LWORD | `w` / `dw` / `lw` | | TIME / LTIME | `t` / `lt` |
| SINT / USINT | `si` / `usi` | | STRING | `s` |
| INT / UINT | `i` / `ui` | | enumerazione | `e` |
| DINT / UDINT | `di` / `udi` | | struttura | `st` |
| LINT / ULINT | `li` / `uli` | | istanza di FB | `fb` |
| ARRAY | `a` + prefisso dell'elemento (`abyBuffer`) | | POINTER TO / REFERENCE TO | `p` / `ref` + prefisso (`psString`) |

**Cartelle:** una per tipo di oggetto, nome in MAIUSCOLO al plurale con la `s` finale minuscola e `_` al posto degli spazi (`FUNCTION_BLOCKs`).

**Commenti:** ogni POU inizia con un commento che spiega a cosa serve; ogni metodo ha un commento con scopo e significato del valore restituito. I commenti sopra le dichiarazioni compaiono nella documentazione del Library Manager.

## Installare e usare la libreria

1. Aprire `MyLibrary.sln` in TwinCAT XAE.
2. Tasto destro sul progetto PLC `MyLibrary` → **Save as library and install...**: la libreria viene installata nel repository delle librerie del PC.
3. Nel progetto che la usa: **References** → **Add library** → cercare `MyLibrary` (company `FabioBolognesi`).
4. Usare gli oggetti con il namespace: `ML.F_Saturation(...)`, `fbBuilder : ML.FB_StringBuilder;`.

A ogni rilascio: aggiornare la versione (proprietà del progetto PLC → `Version`), il [CHANGELOG.md](CHANGELOG.md) e reinstallare la libreria.

## Test (TcUnit)

I test stanno nel progetto PLC `MyLibrary_Test` e usano il framework [TcUnit](https://tcunit.org). Il progetto referenzia la libreria **installata**, non i sorgenti: dopo ogni modifica alla libreria bisogna reinstallarla (passo 2 sopra) prima di rilanciare i test.

**Preparazione** (una sola volta):
1. Installare TcUnit: da TwinCAT 3.1.4026 con il TwinCAT Package Manager; nelle versioni precedenti scaricando `TcUnit.library` dalle [release di TcUnit](https://github.com/tcunit/TcUnit/releases) e installandola da **PLC → Library Repository → Install**.
2. Aggiungere il progetto di test alla solution: tasto destro sul nodo **PLC** → **Add Existing Item...** → `MyLibrary/MyLibrary_Test/MyLibrary_Test.plcproj`. Se TwinCAT crea un task nuovo invece di usare `PlcTask`, va bene comunque: basta che `MAIN` sia chiamato da un task.

**Esecuzione:**
1. Installare la libreria (**Save as library and install...**).
2. Compilare `MyLibrary_Test`, attivare la configurazione sul runtime locale (o sull'usermode runtime), fare login e Start.
3. I risultati compaiono nella **Error List** di XAE, per esempio `| Test suite ID=0 'MAIN.fbMathTest' ... | Test class 'FB_Math_Test' ... SUCCESS`. Un test fallito mostra il messaggio passato all'`Assert...`.

**Aggiungere un test:**
1. Nella test suite dell'oggetto (`TESTs/FB_<Oggetto>_Test`) aggiungere un metodo `PRIVATE` con questa forma:
   ```
   TEST('NomeDelTest');
   AssertEquals_REAL(Expected := ..., Actual := ..., Delta := 0.0001, Message := 'cosa si controlla');
   TEST_FINISHED();
   ```
2. Chiamare il metodo dal corpo della test suite.
3. Per un oggetto nuovo: creare una nuova suite `FB_<Oggetto>_Test EXTENDS TcUnit.FB_TestSuite` e dichiararne un'istanza in `MAIN`.

Regola: ogni nuova funzione o function block entra nella libreria insieme ai suoi test.

## Stato

| Oggetto | Test |
|---|---|
| `F_Atan2`, `F_Saturation` | `FB_Math_Test` |
| `F_InterpolateReal`, `F_InterpolateTime` | `FB_Interpolate_Test` |
| `F_TimeToSeconds`, `F_SecondsToTime` | `FB_Conversion_Test` |
| `FB_Timer` | `FB_Timer_Test` |
| `FB_Sweep` | `FB_Sweep_Test` (inizializzazione e valori iniziali) |
| `FB_StringBuilder`, `E_Ascii` | `FB_StringBuilder_Test` |

Le cartelle `INTERFACEs`, `PROGRAMs` e `VISUs` del progetto della libreria sono ancora vuote.
