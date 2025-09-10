# FB_[Gerätename] - Dokumentation

## Übersicht

**Funktionsbaustein:** `FB_[Gerätename]`  
**Version:** [x.x]  
**Letzte Änderung:** [TT.MM.JJJJ]  
**Autor:** [Autor]  

### Kurzbeschreibung
[Kurze Beschreibung der Funktionalität des Bausteins]

### Abhängigkeiten
- [Abhängigkeit 1]
- [Abhängigkeit 2]
- FB_BasisMappingProgramAlarm (Standard)

## Versionshistorie

| Version | Datum      | Autor       | Kommentar |
|---------|------------|-------------|-----------|
| v[x.x]  | [TT.MM.JJJJ] | [Autor]   | [Änderung] |

## Architektur und Regionen

### REGION-Struktur nach JennyScience-Standard

#### 1. Power ON [Gerät] (REGION Power ON)
**Zweck:** Sequenzielle Einschaltung und Initialisierung des Geräts

**Zustandsmaschine: power_on_state**

| Zustand | Beschreibung | Eingangsbedingungen | Ausgangsaktion | Timeout |
|---------|--------------|-------------------|----------------|---------|
| 0 | Idle/Einschaltsperre | [Bedingungen] | [Aktionen] | - |
| 1 | [Zustand 1] | [Bedingungen] | [Aktionen] | [Zeit] |
| 2 | [Zustand 2] | [Bedingungen] | [Aktionen] | [Zeit] |
| 100 | Normal-Betrieb | [Bedingungen] | Überwachung aktiv | - |
| -1 | Fehler beim Einschalten | Timeout/Fehler | Reset-Sequenz | - |
| -2 | Betriebsfehler | Gerätestörung | Fehlerbehandlung | - |

**Kommentierung:**
```scl
(*
REGION Power ON [Gerät]
Sequenzielle Einschaltung mit Watchdog-Überwachung
S0: [Beschreibung]
S1: [Beschreibung] 
...
*)
```

#### 2. Sicherheitskreis NIO (REGION Sicherheitskreis NIO)
**Zweck:** Behandlung von Sicherheitskreis-Ausfällen

**Funktionalität:**
- Sofortige Deaktivierung aller Bewegungsbefehle
- Status-Meldung setzen
- Sichere Abschaltung

#### 3. [Spezifische Funktion 1] (REGION [Funktionsname])
**Zweck:** [Beschreibung der Hauptfunktion]

**Zustandsmaschine: [state_variable]**

| Zustand | Beschreibung | Timeout | Fehlercodes |
|---------|--------------|---------|-------------|
| 0 | Idle | - | - |
| 1 | [Zustand] | [Zeit] | [Codes] |
| -1 | Fehlerzustand | - | [Codes] |

#### 4. [Spezifische Funktion 2] (REGION [Funktionsname])
**Zweck:** [Beschreibung]

[Weitere Regionen nach gleichem Schema...]

#### 5. [Gerät] Halt (REGION [Gerät] Halt)
**Zweck:** Kontrolliertes Anhalten aller Bewegungen

#### 6. Eingänge (REGION Eingänge)
**Zweck:** Verarbeitung von Eingangssignalen und Quittierungen

#### 7. Ausgänge (REGION Ausgänge)
**Zweck:** Setzen der Ausgangssignale

#### 8. HMI Interface (REGION HMI Interface)
**Zweck:** Datenaustausch mit HMI-Struktur

#### 9. [Spezielle Features] (REGION [Feature])
**Zweck:** [Beschreibung spezieller Funktionen]

#### 10. Fehlerhandling [Gerät] (REGION Fehlerhandling)
**Zweck:** Zentrale Fehlerbehandlung mit Program_Alarm

## Schnittstellen-Architektur

### Drei-Schnittstellen-Konzept

Das standardisierte Schnittstellenkonzept basiert auf drei klar getrennten Kommunikationsebenen:

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Peripherie    │◄──►│  Funktionsblock  │◄──►│ Anwenderprogramm│
│    (Hardware)   │    │   (FB_[Gerät])   │    │     (Main)      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         ▲                        ▲                        ▲
         │                        │                        │
    ┌────▼────┐              ┌────▼────┐              ┌────▼────┐
    │ DrvToPlc│              │ PlcToDrv│              │[Gerät]  │
    │Interface│              │Interface│              │_HMI     │
    └─────────┘              └─────────┘              └─────────┘
```

## Schnittstelle 1: Peripherie → PLC (DrvToPlc)

### Zweck
**Statusdaten und Rückmeldungen** von der Hardware zur SPS

### Standard-Struktur
```scl
TYPE "ST_[Gerät]_DrvToPlc"
VERSION : 0.1
   STRUCT
      StatusWord : Struct
         Ready : Bool;                    // Gerät betriebsbereit
         Error : Bool;                    // Störung aktiv
         Warning : Bool;                  // Warnung aktiv
         InMotion : Bool;                 // Bewegung aktiv
         PositionReached : Bool;          // Zielposition erreicht
         Referenced : Bool;               // Referenzierung abgeschlossen
         // [Gerätespezifische Status-Bits]
      END_STRUCT;
      ActualPosition : DInt;              // Ist-Position
      ActualVelocity : DInt;              // Ist-Geschwindigkeit
      ActualForce : DInt;                 // Ist-Kraft
      DiagnosisCode : Word;               // Diagnose-Code
      // [Weitere Ist-Werte]
   END_STRUCT;
END_TYPE
```

### Implementierung im FB
```scl
VAR_INPUT
    i_[Gerät]DrvToPlc : "ST_[Gerät]_DrvToPlc";    // Eingangsdaten von Hardware
END_VAR

// In HMI Interface Region:
#[Gerät]_HMI.Ausgang.Ready := #i_[Gerät]DrvToPlc.StatusWord.Ready;
#[Gerät]_HMI.Ausgang.Error := #i_[Gerät]DrvToPlc.StatusWord.Error;
#[Gerät]_HMI.Ausgang.PositionActual := #i_[Gerät]DrvToPlc.ActualPosition;
```

## Schnittstelle 2: PLC → Peripherie (PlcToDrv)

### Zweck
**Befehle und Sollwerte** von der SPS zur Hardware

### Standard-Struktur
```scl
TYPE "ST_[Gerät]_PlcToDrv"
VERSION : 0.1
   STRUCT
      ControlWord : Struct
         Enable : Bool;                   // Freigabe
         Start : Bool;                    // Start-Befehl
         Stop : Bool;                     // Stop-Befehl
         Reset : Bool;                    // Reset-Befehl
         Home : Bool;                     // Referenzfahrt
         // [Gerätespezifische Befehls-Bits]
      END_STRUCT;
      TargetPosition : DInt;              // Soll-Position
      TargetVelocity : DInt;              // Soll-Geschwindigkeit
      TargetForce : DInt;                 // Soll-Kraft
      OperationMode : Byte;               // Betriebsmodus
      // [Weitere Soll-Werte]
   END_STRUCT;
END_TYPE
```

### Implementierung im FB
```scl
VAR_OUTPUT
    q_[Gerät]PlcToDrv : "ST_[Gerät]_PlcToDrv";    // Ausgangsdaten zur Hardware
END_VAR

// In Power ON Region:
#q_[Gerät]PlcToDrv.ControlWord.Enable := TRUE;

// In Funktions-Regionen:
#q_[Gerät]PlcToDrv.TargetPosition := #[Gerät]_HMI.Eingang.TargetPosition;
#q_[Gerät]PlcToDrv.ControlWord.Start := (#[funktion]_state = 2);
```

## Schnittstelle 3: HMI-Interface (Anwenderschnittstelle)

### Zweck
**Bedienschnittstelle** zwischen Funktionsblock und Anwenderprogramm

### Standard-Struktur
```scl
TYPE "ST_[Gerät]_HMI"
VERSION : 0.1
   STRUCT
      Eingang : Struct   // Befehle vom Anwenderprogramm
         Execute : Bool;                  // Funktion ausführen
         Home : Bool;                     // Referenzfahrt
         Stop : Bool;                     // Stopp
         Reset : Bool;                    // Reset/Quittierung
         TargetPosition : DInt;           // Ziel-Position
         Velocity : DInt;                 // Geschwindigkeit
         Acceleration : DInt;             // Beschleunigung
         // [Weitere Eingabeparameter]
      END_STRUCT;
      Ausgang : Struct   // Status zum Anwenderprogramm
         Status : String;                 // Status-Text
         StepBusy : Bool;                 // FB beschäftigt
         StepDone : Bool;                 // FB bereit für neuen Befehl
         InPosition : Bool;               // Position erreicht
         Ready : Bool;                    // Gerät betriebsbereit
         Error : Bool;                    // Fehler aktiv
         ErrorCode : Int;                 // Fehlercode
         PositionActual : DInt;           // Ist-Position
         // [Weitere Statuswerte]
      END_STRUCT;
   END_STRUCT;
END_TYPE
```

### Implementierung im FB
```scl
VAR_IN_OUT
    [Gerät]_HMI : "ST_[Gerät]_HMI";               // HMI-Schnittstelle
END_VAR

// Eingänge verarbeiten:
IF #[Gerät]_HMI.Eingang.Execute AND NOT #[Gerät]_HMI.Ausgang.StepBusy THEN
    #[funktion]_state := 1;
END_IF;

// Ausgänge setzen:
#[Gerät]_HMI.Ausgang.Ready := #power_on_state = 100;
#[Gerät]_HMI.Ausgang.ErrorCode := #error_state;
```

## Detaillierte Schnittstellen-Spezifikationen

### Standard-Eingänge (VAR_INPUT)
| Variable | Typ | Beschreibung | Verwendung |
|----------|-----|--------------|------------|
| ip_Maschine | Int | Maschinennummer | Alarm-System |
| ip_Station | Int | Stationsnummer | Alarm-System |
| ip_Bmk | String[30] | Begleitwert | Alarm-System |
| i_StoerungQuittieren | Bool | Störung quittieren | Reset error_state |
| i_NotHalt_IO | Bool | Steuerung Ein | Power-ON Bedingung |
| i_Sicherheitskreis_IO | Bool | Sicherheitskreis OK | Power-ON Bedingung |
| i_FreigabeBewegung | Bool | Bewegungsfreigabe | Funktions-Bedingung |
| i_FreigabeHMI | Bool | HMI-Freigabe | Bedienfreigabe |
| **i_[Gerät]DrvToPlc** | **ST_[Gerät]_DrvToPlc** | **Hardware-Eingänge** | **Status-Auswertung** |

### Standard-Ausgänge (VAR_OUTPUT)
| Variable | Typ | Beschreibung | Verwendung |
|----------|-----|--------------|------------|
| q_[Gerät]Ready | Bool | Gerät betriebsbereit | Anwenderprogramm |
| **q_[Gerät]PlcToDrv** | **ST_[Gerät]_PlcToDrv** | **Hardware-Ausgänge** | **Befehls-Übertragung** |

### Standard In/Out-Parameter (VAR_IN_OUT)
| Variable | Typ | Beschreibung | Verwendung |
|----------|-----|--------------|------------|
| **[Gerät]_HMI** | **ST_[Gerät]_HMI** | **HMI-Interface** | **Anwender-Kommunikation** |
| iq_Stoermeldeindikator | Bool | Störmeldeindikator | Alarm-System |

### VAR (Interne Variablen)
| Variable | Typ | Beschreibung |
|----------|-----|--------------|
| power_on_state | Int | Zustand Einschaltsequenz |
| [funktion]_state | Int | Zustand [Funktionsname] |
| error_state | Int | Fehlerstatus |
| hm[Funktion] | Bool | Hilfsmerkbit [Funktion] |

### VAR DB_SPECIFIC (Timer)
| Variable | Typ | Beschreibung |
|----------|-----|--------------|
| wthDog | TON_TIME | Watchdog-Timer |
| tmr_[Funktion] | TON_TIME | Timer für [Funktion] |

## HMI-Struktur (ST_[Gerät]_HMI)

### Eingang-Struktur
| Variable | Typ | Beschreibung | Standardwert |
|----------|-----|--------------|--------------|
| [Befehl1] | Bool | [Beschreibung] | FALSE |
| [Parameter1] | [Typ] | [Beschreibung] | [Wert] |

### Ausgang-Struktur
| Variable | Typ | Beschreibung |
|----------|-----|--------------|
| Status | String | Aktueller Status-Text |
| StepBusy | Bool | Baustein beschäftigt |
| StepDone | Bool | Baustein bereit |
| ErrorState | Int | Fehlerstatus-Code |
| [Statuswerte] | [Typ] | [Beschreibung] |

## Fehlercodes (error_state)

| Code | Beschreibung | Ursache | Behebung |
|------|--------------|---------|----------|
| 0 | Kein Fehler | - | - |
| 1 | [Fehler 1] | [Ursache] | [Lösung] |
| 2 | [Fehler 2] | [Ursache] | [Lösung] |
| ... | ... | ... | ... |

## Zeitverhalten

### Standard-Timeouts
| Vorgang | Timeout | Beschreibung |
|---------|---------|--------------|
| Watchdog allgemein | 500ms | Standard-Überwachung |
| [Funktion 1] | [Zeit] | [Beschreibung] |
| [Funktion 2] | [Zeit] | [Beschreibung] |

### Timer-Verwendung
- **wthDog**: Universeller Watchdog für kritische Vorgänge
- **tmr_[Name]**: Spezifische Timer für Funktionsüberwachung

## Code-Beispiele und Patterns

### Vollständiges Funktionsblock-Template
```scl
FUNCTION_BLOCK "FB_[Gerätename]"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      ip_Maschine : Int := 1;
      ip_Station : Int;
      ip_Bmk : String[30];
      i_StoerungQuittieren : Bool;
      i_NotHalt_IO : Bool;
      i_Sicherheitskreis_IO : Bool;
      i_FreigabeBewegung : Bool;
      i_FreigabeHMI : Bool;
      // [Gerätespezifische Eingänge hier einfügen]
   END_VAR

   VAR_OUTPUT 
      q_[Gerät]Ready : Bool;
      // [Gerätespezifische Ausgänge hier einfügen]
   END_VAR

   VAR_IN_OUT 
      [Gerät]_HMI : "ST_[Gerät]_HMI";
      iq_Stoermeldeindikator { ExternalVisible := 'False'} : Bool;
   END_VAR

   VAR 
      power_on_state : Int;
      [funktion]_state : Int;
      error_state : Int;
      hm[Funktion] : Bool;
   END_VAR
   
   VAR DB_SPECIFIC
      wthDog {InstructionName := 'TON_TIME'; LibVersion := '1.0'} : TON_TIME;
      tmr_[Funktion] {InstructionName := 'TON_TIME'; LibVersion := '1.0'} : TON_TIME;
   END_VAR
   
   VAR 
      iDB_BasisMappingProgramAlarm : "FB_BasisMappingProgramAlarm";
      Meldungen_[Gerät] {InstructionName := 'Program_Alarm'; LibVersion := '1.0'} : Program_Alarm;
   END_VAR

BEGIN
    (* Kommentarblock hier einfügen *)
    
    // [REGIONEN hier implementieren]
    
END_FUNCTION_BLOCK
```

### HMI-Struktur Template
```scl
TYPE "ST_[Gerät]_HMI"
VERSION : 0.1
   STRUCT
      Eingang : Struct
         Execute : Bool;
         [Befehl1] : Bool;
         [Parameter1] { S7_SetPoint := 'True'} : [Typ] := [Standardwert];
         // [Weitere Parameter]
      END_STRUCT;
      Ausgang : Struct
         Status : String;
         StepBusy : Bool;
         StepDone : Bool;
         ErrorState : Int;
         [Statuswert1] : Bool;
         // [Weitere Statuswerte]
      END_STRUCT;
   END_STRUCT;
END_TYPE
```

## Programmierrichtlinien

### Kommentierungsstandard
```scl
(*
version |   Datum   | Autor       | Kommentar
--------|-----------|-------------|--------------------------------------------------
v[x.x]  |[TT.MM.JJJJ]| [Autor]    | [Änderung]
--------|-----------|-------------|--------------------------------------------------

--- Kurzbeschreibung ---
[Beschreibung der Funktionalität]

--- Abhängig von ---
[Abhängigkeiten auflisten]

--- Alarmbeschreibung ---
error_state:
[Fehlercodes mit Beschreibung]
*)
```

### REGION-Struktur Templates

#### Standard Power-ON Pattern
```scl
REGION Power ON [Gerät]
    IF #i_NotHalt_IO AND #i_Sicherheitskreis_IO THEN
        CASE #power_on_state OF
            0: // S0 Führung gefordert
                #[Gerät]_HMI.Ausgang.Status := '[Gerät] Führung gefordert';
                IF [Führungsbedingung] THEN
                    [Führung aktivieren]
                    #power_on_state := 1;
                END_IF;
                
            1: // S1 Einschaltbereit
                #[Gerät]_HMI.Ausgang.Status := '[Gerät] Einschaltbereit';
                #wthDog(IN := TRUE, PT := T#500ms);
                IF [Bereitschaftsbedingung] THEN
                    [Einschaltbefehl]
                    #wthDog(IN := FALSE, PT := T#500ms);
                    #power_on_state := 2;
                ELSIF #wthDog.Q THEN
                    #power_on_state := -1;
                END_IF;
                
            100: // Normal-Betrieb
                #[Gerät]_HMI.Ausgang.Status := '[Gerät] Betriebsbereit';
                IF [Störungsbedingung] THEN
                    #power_on_state := -2;
                END_IF;
                
            -1: // Einschaltfehler
                #[Gerät]_HMI.Ausgang.Status := 'Fehler beim Einschalten';
                IF [Reset-Bedingung] THEN
                    #power_on_state := 0;
                END_IF;
        END_CASE;
    ELSE
        // Sicherheit nicht OK
        #power_on_state := 0;
        #[Gerät]_HMI.Ausgang.Status := 'Sicherheitskreis NIO';
    END_IF;
END_REGION
```

#### Standard Funktions-Pattern
```scl
REGION [Funktionsname]
    IF #hm[Hauptfunktion]
        AND #[Gerät]_HMI.Eingang.[Befehl]
        AND #error_state = 0
        AND #power_on_state = 100
    THEN
        CASE #[funktion]_state OF
            0: // Idle/Vorbereitung
                #[Gerät]_HMI.Ausgang.Status := '[Funktion] starten';
                #[Gerät]_HMI.Ausgang.StepBusy := TRUE;
                #[Gerät]_HMI.Ausgang.StepDone := FALSE;
                [Vorbereitungsaktionen]
                #[funktion]_state := 1;
                
            1: // Ausführung
                #[Gerät]_HMI.Ausgang.Status := '[Funktion] läuft';
                #wthDog(IN := TRUE, PT := T#[Timeout]);
                IF [Erfolgsbedingung] THEN
                    #wthDog(IN := FALSE, PT := T#[Timeout]);
                    #[funktion]_state := 100;
                ELSIF #wthDog.Q THEN
                    #error_state := [Fehlercode];
                    #[funktion]_state := -1;
                END_IF;
                
            100: // Abschluss
                #[Gerät]_HMI.Ausgang.Status := '[Funktion] abgeschlossen';
                #[Gerät]_HMI.Eingang.[Befehl] := FALSE; // Auto-Reset
                #[Gerät]_HMI.Ausgang.StepBusy := FALSE;
                #[Gerät]_HMI.Ausgang.StepDone := TRUE;
                #[funktion]_state := 0;
                
            -1: // Fehler
                #[Gerät]_HMI.Ausgang.Status := 'Fehler [Funktion]';
                #[Gerät]_HMI.Ausgang.StepBusy := FALSE;
                #[Gerät]_HMI.Ausgang.StepDone := TRUE;
                #[funktion]_state := 0;
        END_CASE;
    END_IF;
END_REGION
```

#### Standard Halt-Pattern
```scl
REGION [Gerät] Halt
    IF #[Gerät]_HMI.Eingang.Halt THEN
        #[Gerät]_HMI.Eingang.Execute := FALSE;
        #[Gerät]_HMI.Eingang.[Weitere Befehle] := FALSE;
        
        CASE #halt_state OF
            0: // Halt einleiten
                #[Gerät]_HMI.Ausgang.Status := '[Gerät] wird angehalten';
                [Halt-Befehle senden]
                #halt_state := 1;
                
            1: // Warten auf Stillstand
                IF [Stillstandsbedingung] THEN
                    #[Gerät]_HMI.Ausgang.Status := '[Gerät] steht';
                    #[Gerät]_HMI.Eingang.Halt := FALSE;
                    #halt_state := 0;
                    #error_state := 0; // Fehler zurücksetzen
                END_IF;
        END_CASE;
    END_IF;
END_REGION
```

#### Standard Fehlerhandling-Pattern
```scl
REGION Fehlerhandling [Gerät]
    #iDB_BasisMappingProgramAlarm
    (i_Signal := [Störungsbedingung] OR #error_state <> 0,
     i_Maschine := #ip_Maschine,
     i_Station := #ip_Station,
     i_Bmk := #ip_Bmk,
     i_FehlernummerObjekt := #error_state,
     i_IstWarnung := FALSE,
     iq_Meldung := #Meldungen_[Gerät]);
    
    // Störungsbehandlung
    IF #iDB_BasisMappingProgramAlarm.q_Stoerung THEN
        #iq_Stoermeldeindikator := TRUE;
        // Alle Bewegungsbefehle zurücksetzen
        #[Gerät]_HMI.Eingang.Execute := FALSE;
        #[Gerät]_HMI.Eingang.[Weitere Befehle] := FALSE;
    END_IF;
    
    // Quittierung
    IF #i_StoerungQuittieren THEN
        #error_state := 0;
        #iq_Stoermeldeindikator := FALSE;
    END_IF;
END_REGION
```

### Variablen-Namenskonvention
- **Zustandsvariablen**: `[funktion]_state`
- **Hilfsmerkbits**: `hm[Funktion]`
- **Timer**: `tmr_[Funktion]` oder `wthDog`
- **HMI-Interface**: `[Gerät]_HMI`

### Status-Meldungen
- Einheitliche Formulierung
- Aussagekräftige Beschreibung des aktuellen Zustands
- Fehlermeldungen mit Lösungshinweisen

## Inbetriebnahme-Checkliste

### Vorbereitung
- [ ] Sicherheitskreis funktionsfähig
- [ ] NotHalt-Funktion getestet
- [ ] Kommunikation zum Gerät etabliert
- [ ] Parameter konfiguriert

### Funktionstest
- [ ] Einschaltsequenz durchlaufen
- [ ] [Spezifische Funktion 1] getestet
- [ ] [Spezifische Funktion 2] getestet
- [ ] Fehlerverhalten geprüft
- [ ] HMI-Bedienung validiert

### Abnahme
- [ ] Alle Funktionen dokumentiert
- [ ] Fehlerbehandlung verifiziert
- [ ] Performance-Tests durchgeführt
- [ ] Betriebsdokumentation erstellt

## Wartung und Erweiterung

### Anpassungsmöglichkeiten
- Timeout-Werte in Timer-Aufrufen
- Fehlercodes in error_state-Behandlung
- HMI-Texte in Status-Zuweisungen
- Parameter-Bereiche in VAR-Deklarationen

### Erweiterungsrichtlinien
1. Neue REGION am Ende einfügen
2. Zustandsvariable für komplexe Funktionen hinzufügen
3. Timer für zeitkritische Vorgänge definieren
4. HMI-Interface entsprechend erweitern
5. Fehlercodes dokumentieren und zuweisen

### Debugging-Hilfen
- Status-Text für aktuellen Zustand
- error_state für systematische Fehleranalyse
- Watchdog-Timer für Timeout-Erkennung
- HMI-Interface für Live-Diagnose

---

## UDT-Erstellungsanleitung

### Die 3 Standard-UDTs erstellen

#### UDT 1: ST_[Gerät]_DrvToPlc
```scl
TYPE "ST_[Gerät]_DrvToPlc"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   STRUCT
      StatusWord : Struct
         Ready { S7_SetPoint := 'False'} : Bool := false;                    // Bit 0: Gerät betriebsbereit
         Error { S7_SetPoint := 'False'} : Bool := false;                    // Bit 1: Störung aktiv  
         Warning { S7_SetPoint := 'False'} : Bool := false;                  // Bit 2: Warnung aktiv
         InMotion { S7_SetPoint := 'False'} : Bool := false;                 // Bit 3: Bewegung aktiv
         PositionReached { S7_SetPoint := 'False'} : Bool := false;          // Bit 4: Zielposition erreicht
         Referenced { S7_SetPoint := 'False'} : Bool := false;               // Bit 5: Referenzierung abgeschlossen
         HomePosition { S7_SetPoint := 'False'} : Bool := false;             // Bit 6: Home-Position
         LimitSwitchPos { S7_SetPoint := 'False'} : Bool := false;           // Bit 7: Endschalter positiv
         LimitSwitchNeg { S7_SetPoint := 'False'} : Bool := false;           // Bit 8: Endschalter negativ
         // Reserve Bits 9-15 für gerätespezifische Status
      END_STRUCT;
      ActualPosition { S7_SetPoint := 'False'} : DInt := 0;                  // Ist-Position in [Einheit]
      ActualVelocity { S7_SetPoint := 'False'} : DInt := 0;                  // Ist-Geschwindigkeit in [Einheit]/s
      ActualForce { S7_SetPoint := 'False'} : DInt := 0;                     // Ist-Kraft in [Einheit]
      DiagnosisCode { S7_SetPoint := 'False'} : Word := 16#0000;             // Diagnose-Code (Herstellerspezifisch)
      DeviceID { S7_SetPoint := 'False'} : Word := 16#0000;                  // Geräte-Identifikation
      FirmwareVersion { S7_SetPoint := 'False'} : Word := 16#0000;           // Firmware-Version
      // [Weitere gerätespezifische Ist-Werte hier hinzufügen]
   END_STRUCT;
END_TYPE
```

#### UDT 2: ST_[Gerät]_PlcToDrv
```scl
TYPE "ST_[Gerät]_PlcToDrv"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   STRUCT
      ControlWord : Struct
         Enable { S7_SetPoint := 'True'} : Bool := false;                    // Bit 0: Freigabe
         Start { S7_SetPoint := 'True'} : Bool := false;                     // Bit 1: Start-Befehl
         Stop { S7_SetPoint := 'True'} : Bool := false;                      // Bit 2: Stop-Befehl  
         Reset { S7_SetPoint := 'True'} : Bool := false;                     // Bit 3: Reset-Befehl
         Home { S7_SetPoint := 'True'} : Bool := false;                      // Bit 4: Referenzfahrt
         Acknowledge { S7_SetPoint := 'True'} : Bool := false;               // Bit 5: Quittierung
         ModeSwitch { S7_SetPoint := 'True'} : Bool := false;                // Bit 6: Modus-Umschaltung
         // Reserve Bits 7-15 für gerätespezifische Befehle
      END_STRUCT;
      TargetPosition { S7_SetPoint := 'True'} : DInt := 0;                   // Soll-Position in [Einheit]
      TargetVelocity { S7_SetPoint := 'True'} : DInt := 1000;                // Soll-Geschwindigkeit in [Einheit]/s
      TargetAcceleration { S7_SetPoint := 'True'} : DInt := 1000;            // Soll-Beschleunigung in [Einheit]/s²
      TargetForce { S7_SetPoint := 'True'} : DInt := 100;                    // Soll-Kraft in [Einheit]
      OperationMode { S7_SetPoint := 'True'} : Byte := 1;                    // Betriebsmodus (1=Position, 2=Velocity, 3=Force)
      ParameterSet { S7_SetPoint := 'True'} : Byte := 1;                     // Parametersatz-Nummer
      // [Weitere gerätespezifische Soll-Werte hier hinzufügen]
   END_STRUCT;
END_TYPE
```

#### UDT 3: ST_[Gerät]_HMI
```scl
TYPE "ST_[Gerät]_HMI"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   STRUCT
      Eingang : Struct   // Befehle VOM Anwenderprogramm ZUM Funktionsblock
         Execute { S7_SetPoint := 'True'} : Bool := false;                   // Funktion ausführen
         Home { S7_SetPoint := 'True'} : Bool := false;                      // Referenzfahrt starten
         Stop { S7_SetPoint := 'True'} : Bool := false;                      // Stopp-Befehl
         Reset { S7_SetPoint := 'True'} : Bool := false;                     // Reset/Quittierung
         JogPlus { S7_SetPoint := 'True'} : Bool := false;                   // Jog vorwärts
         JogMinus { S7_SetPoint := 'True'} : Bool := false;                  // Jog rückwärts
         TargetPosition { S7_SetPoint := 'True'} : DInt := 0;                // Ziel-Position in [Einheit]
         Velocity { S7_SetPoint := 'True'} : DInt := 50;                     // Geschwindigkeit in % (1-100)
         Acceleration { S7_SetPoint := 'True'} : DInt := 25;                 // Beschleunigung in % (1-100)  
         Force { S7_SetPoint := 'True'} : DInt := 50;                        // Kraftbegrenzung in % (1-100)
         OperationMode { S7_SetPoint := 'True'} : Int := 1;                  // Betriebsmodus
         Timeout { S7_SetPoint := 'True'} : Time := T#10s;                   // Timeout für Operationen
         // [Weitere Eingabeparameter hier hinzufügen]
      END_STRUCT;
      Ausgang : Struct   // Status VOM Funktionsblock ZUM Anwenderprogramm
         Status { S7_SetPoint := 'False'} : String := '';                    // Status-Text (max. 254 Zeichen)
         StepBusy { S7_SetPoint := 'False'} : Bool := false;                 // FB beschäftigt
         StepDone { S7_SetPoint := 'False'} : Bool := true;                  // FB bereit für neuen Befehl
         InPosition { S7_SetPoint := 'False'} : Bool := false;               // Position erreicht
         Ready { S7_SetPoint := 'False'} : Bool := false;                    // Gerät betriebsbereit
         Error { S7_SetPoint := 'False'} : Bool := false;                    // Fehler aktiv
         Warning { S7_SetPoint := 'False'} : Bool := false;                  // Warnung aktiv
         Referenced { S7_SetPoint := 'False'} : Bool := false;               // Referenziert
         ErrorCode { S7_SetPoint := 'False'} : Int := 0;                     // Fehlercode (siehe Fehlertabelle)
         PositionActual { S7_SetPoint := 'False'} : DInt := 0;               // Ist-Position in [Einheit]
         VelocityActual { S7_SetPoint := 'False'} : DInt := 0;               // Ist-Geschwindigkeit
         ForceActual { S7_SetPoint := 'False'} : DInt := 0;                  // Ist-Kraft
         Progress { S7_SetPoint := 'False'} : Int := 0;                      // Fortschritt in % (0-100)
         // [Weitere Statuswerte hier hinzufügen]
      END_STRUCT;
   END_STRUCT;
END_TYPE
```

## Herstellerdokumentation als Basis für UDT-Entwicklung

### 📚 **Primäre Informationsquellen**

#### 1. Hersteller-Beispielprogramme
```
🔍 Typische Dateien suchen:
├── Example_[Gerätename].scl
├── Sample_[Gerätename]_Control.scl  
├── Demo_[Gerätename].zip
└── [Gerätename]_Library.zip

📁 Wichtige Inhalte extrahieren:
- Telegramm-Strukturen (Input/Output)
- StatusWord/ControlWord Bit-Definitionen
- Beispiel-Zustandsmaschinen
- Timing-Parameter und Timeouts
```

#### 2. PDF-Handbücher analysieren
```
📖 Relevante Kapitel identifizieren:
- "Kommunikation" / "Communication Interface"
- "Telegramm-Aufbau" / "Telegram Structure"  
- "Statuswörter" / "Status Words"
- "Fehlercodes" / "Error Codes"
- "Programmierbeispiele" / "Programming Examples"

🔍 Spezifische Tabellen suchen:
- Bit-Zuordnungstabellen (StatusWord/ControlWord)
- Datentyp-Spezifikationen
- Wertebereiche und Einheiten
- Timeout-Empfehlungen
```

### 🛠️ **Herstellerspezifische UDT-Anpassung**

#### Template-Anpassung basierend auf Herstellerdokumentation:

```scl
// SCHRITT 1: Hersteller-Telegramm analysieren (aus PDF/Beispielen)
// Beispiel: Siemens Sinamics Telegramm 111
TYPE "ST_Sinamics_DrvToPlc"  // ← Herstellername + Geräteserie
   STRUCT
      // StatusWord aus Handbuch Kapitel "Telegramm 111"
      ZSW1 : Struct  // ← Original-Bezeichnung beibehalten
         Bereit : Bool;                    // Bit 0 - aus Hersteller-Bit-Tabelle
         Freigabe : Bool;                  // Bit 1 - aus Hersteller-Bit-Tabelle  
         Referenziert : Bool;              // Bit 11 - aus Hersteller-Bit-Tabelle
         // [Weitere Bits entsprechend Hersteller-Dokumentation]
      END_STRUCT;
      XIST1 : DInt;    // ← Original-Bezeichnung aus Handbuch
      NIST : DInt;     // ← Original-Bezeichnung aus Handbuch
      // [Weitere Werte entsprechend Telegramm-Definition]
   END_STRUCT;
END_TYPE
```

#### Beispiel-Workflow: Siemens Sinamics
```scl
// 1. Aus Sinamics Handbuch "Telegramm 111" extrahiert:
StatusWord1.Bit0 = Betriebsbereit
StatusWord1.Bit1 = Keine Störung  
StatusWord1.Bit11 = Referenzpunkt gesetzt

// 2. In UDT übertragen:
TYPE "ST_Sinamics_DrvToPlc"
   STRUCT
      ZSW1 : Struct
         Betriebsbereit : Bool;           // Bit 0 (Hersteller-Bezeichnung)
         Keine_Stoerung : Bool;           // Bit 1  
         Referenzpunkt_gesetzt : Bool;    // Bit 11
      END_STRUCT;
   END_STRUCT;
END_TYPE
```

### 📋 **Dokumentations-Checkliste für UDT-Erstellung**

#### Vor UDT-Erstellung sammeln:
- [ ] **Hersteller-Beispielprogramm** heruntergeladen und analysiert
- [ ] **Kommunikationshandbuch** mit Telegramm-Definitionen vorliegen
- [ ] **Bit-Zuordnungstabellen** für StatusWord/ControlWord extrahiert
- [ ] **Datentyp-Spezifikationen** (Einheiten, Wertebereiche) dokumentiert
- [ ] **Fehlercodes-Tabelle** aus Handbuch übernommen
- [ ] **Timing-Parameter** (Timeouts, Zykluszeiten) ermittelt

#### UDT-Anpassungsschritte:
1. **Template kopieren** und Gerätename einsetzen
2. **StatusWord-Bits** entsprechend Hersteller-Dokumentation anpassen
3. **ControlWord-Bits** entsprechend Hersteller-Dokumentation anpassen  
4. **Datentypen und Einheiten** aus Handbuch übernehmen
5. **Original-Bezeichnungen** des Herstellers beibehalten (ZSW1, STW1, etc.)
6. **Standardwerte** entsprechend Hersteller-Empfehlungen setzen

### UDT-Erstellungsreihenfolge
1. **Herstellerdokumentation studieren** - Telegramm-Aufbau verstehen
2. **DrvToPlc erstellen** - Status-Telegramm implementieren
3. **PlcToDrv erstellen** - Befehl-Telegramm implementieren  
4. **HMI anpassen** - Anwenderschnittstelle definieren

### Wichtige UDT-Attribute
- **Hersteller-Bezeichnungen beibehalten**: ZSW1, STW1, XIST1, etc.
- **Original-Bit-Nummerierung**: Entsprechend Hersteller-Dokumentation
- **Einheiten aus Handbuch**: mm, °, mN, etc.
- **Wertebereiche validieren**: Min/Max-Werte aus Spezifikation

## Schritt-für-Schritt Implementierungsanleitung

### Phase 1: UDTs und Grundstruktur erstellen
1. **Drei UDTs erstellen** in der oben gezeigten Reihenfolge
2. **FB-Deklaration kopieren** und Platzhalter `[Gerätename]` ersetzen
3. **UDT-Referenzen einbinden** in VAR_INPUT/OUTPUT
4. **Variablen anpassen**: Zustandsvariablen für jede Hauptfunktion
5. **Timer hinzufügen** für zeitkritische Vorgänge

### Phase 2: Herstellerspezifische Implementierung
1. **Hersteller-Beispielcode analysieren** und State-Machine extrahieren
2. **Power-ON Sequenz** entsprechend Hersteller-Vorgaben implementieren
3. **Original-Zeitvorgaben** aus Handbuch übernehmen (nicht schätzen!)
4. **Hersteller-Fehlercodes** in error_state-Behandlung integrieren
5. **Sicherheitskonzept** entsprechend Gerätespezifikation umsetzen

### Phase 3: Hauptfunktionen hinzufügen
1. **Funktions-Regionen** nach Standard-Pattern
2. **Zustandsmaschinen** mit klaren Übergangsbedingungen
3. **Timeout-Überwachung** für alle zeitkritischen Vorgänge
4. **Status-Meldungen** für jede Zustandsänderung

### Phase 4: Integration und Test
1. **HMI-Interface** vollständig verknüpfen
2. **Fehlerhandling** testen und verfeinern
3. **Dokumentation** aktualisieren
4. **Code-Review** durchführen

## Checkliste für neue Funktionsblöcke

### UDT-Checkliste
- [ ] ST_[Gerät]_DrvToPlc mit StatusWord und Ist-Werten erstellt
- [ ] ST_[Gerät]_PlcToDrv mit ControlWord und Soll-Werten erstellt  
- [ ] ST_[Gerät]_HMI mit Eingang/Ausgang-Strukturen erstellt
- [ ] S7_SetPoint Attribute korrekt gesetzt (Input=False, Output=True)
- [ ] Standardwerte für alle Parameter definiert
- [ ] Kommentare mit Einheiten für alle Elemente vorhanden

### Struktur-Checkliste
- [ ] Kommentarblock mit Versionshistorie vorhanden
- [ ] Alle Standard-Eingänge implementiert (NotHalt, Sicherheitskreis, etc.)
- [ ] Drei UDTs korrekt referenziert in FB-Deklaration
- [ ] HMI-Struktur vollständig definiert
- [ ] Watchdog-Timer für kritische Vorgänge vorhanden
- [ ] Fehlerhandling mit Program_Alarm implementiert

### Code-Qualität-Checkliste
- [ ] Einheitliche Namenskonvention befolgt
- [ ] Alle Zustandsübergänge dokumentiert
- [ ] Timeout-Werte angemessen gewählt
- [ ] Status-Meldungen aussagekräftig
- [ ] Fehlercodes systematisch vergeben

### Test-Checkliste
- [ ] Power-ON-Sequenz getestet
- [ ] Alle Hauptfunktionen validiert
- [ ] Fehlerverhalten geprüft
- [ ] Halt-Funktion verifiziert
- [ ] HMI-Bedienung getestet

## Schnittstellen-Integration in Regionen

### Standard-Pattern für Hardware-Kommunikation
```scl
REGION HMI Interface
    // 1. Hardware-Status zu HMI weiterleiten
    #[Gerät]_HMI.Ausgang.Ready := #i_[Gerät]DrvToPlc.StatusWord.Ready;
    #[Gerät]_HMI.Ausgang.Error := #i_[Gerät]DrvToPlc.StatusWord.Error;
    #[Gerät]_HMI.Ausgang.InPosition := #i_[Gerät]DrvToPlc.StatusWord.PositionReached;
    #[Gerät]_HMI.Ausgang.PositionActual := #i_[Gerät]DrvToPlc.ActualPosition;
    
    // 2. HMI-Parameter zu Hardware weiterleiten  
    #q_[Gerät]PlcToDrv.TargetPosition := #[Gerät]_HMI.Eingang.TargetPosition;
    #q_[Gerät]PlcToDrv.TargetVelocity := #[Gerät]_HMI.Eingang.Velocity;
    
    // 3. Interne Status zu HMI
    #[Gerät]_HMI.Ausgang.StepBusy := (#power_on_state <> 100) OR (#[funktion]_state <> 0);
    #[Gerät]_HMI.Ausgang.StepDone := (#power_on_state = 100) AND (#[funktion]_state = 0);
    #[Gerät]_HMI.Ausgang.ErrorCode := #error_state;
END_REGION
```

### Schnittstellen-Validierung
```scl
REGION Schnittstellen Validierung
    // Eingangsdaten-Plausibilität prüfen
    IF #i_[Gerät]DrvToPlc.StatusWord.Error AND #i_[Gerät]DrvToPlc.StatusWord.Ready THEN
        #error_state := 99; // Inkonsistente Hardware-Meldung
    END_IF;
    
    // Ausgangsdaten-Konsistenz sicherstellen
    IF NOT #i_NotHalt_IO OR NOT #i_Sicherheitskreis_IO THEN
        #q_[Gerät]PlcToDrv.ControlWord.Enable := FALSE;
        #q_[Gerät]PlcToDrv.ControlWord.Start := FALSE;
    END_IF;
END_REGION
```

## Gerätespezifische Schnittstellen-Erweiterungen

### Servo-Antriebe
```scl
// DrvToPlc Erweiterung
TYPE "ST_ServoMotor_DrvToPlc"
   STRUCT
      StatusWord : Struct
         Ready : Bool;
         Error : Bool;
         Referenced : Bool;
         InPosition : Bool;
         Following : Bool;              // Nachführung aktiv
         TargetReached : Bool;          // Ziel erreicht
      END_STRUCT;
      ActualPosition : DInt;
      ActualVelocity : DInt;
      ActualTorque : Int;               // Ist-Drehmoment
      FollowingError : DInt;            // Schleppfehler
      DriveTemperature : Int;           // Antriebstemperatur
   END_STRUCT;
END_TYPE

// PlcToDrv Erweiterung  
TYPE "ST_ServoMotor_PlcToDrv"
   STRUCT
      ControlWord : Struct
         Enable : Bool;
         Start : Bool;
         Home : Bool;
         Halt : Bool;
         Reset : Bool;
      END_STRUCT;
      TargetPosition : DInt;
      TargetVelocity : DInt;
      MaxTorque : Int;                  // Max. Drehmoment
      OperationMode : Byte;             // 1=Position, 2=Velocity, 3=Torque
      ProfileVelocity : DInt;           // Profilgeschwindigkeit
      ProfileAcceleration : DInt;       // Profilbeschleunigung
   END_STRUCT;
END_TYPE
```

### Pneumatik-Aktoren
```scl
// DrvToPlc für Pneumatik
TYPE "ST_Pneumatic_DrvToPlc"
   STRUCT
      StatusWord : Struct
         Ready : Bool;
         Error : Bool;
         PositionA : Bool;              // Position A erreicht
         PositionB : Bool;              // Position B erreicht
         InMotion : Bool;
         PressureOK : Bool;             // Betriebsdruck OK
      END_STRUCT;
      ActualPressure : Int;             // Ist-Druck
      CycleCounter : DInt;              // Zyklenzähler
   END_STRUCT;
END_TYPE

// PlcToDrv für Pneumatik
TYPE "ST_Pneumatic_PlcToDrv"
   STRUCT
      ControlWord : Struct
         Enable : Bool;
         MoveToA : Bool;                // Fahre zu Position A
         MoveToB : Bool;                // Fahre zu Position B
         Reset : Bool;
      END_STRUCT;
      PressureSetpoint : Int;           // Soll-Druck
      SpeedControl : Byte;              // Geschwindigkeitsregelung
   END_STRUCT;
END_TYPE
```

### Sensor-Systeme
```scl
// DrvToPlc für Sensoren
TYPE "ST_Sensor_DrvToPlc"
   STRUCT
      StatusWord : Struct
         Ready : Bool;
         Error : Bool;
         DataValid : Bool;              // Messwert gültig
         CalibrationOK : Bool;          // Kalibrierung OK
         OutOfRange : Bool;             // Messbereich überschritten
      END_STRUCT;
      MeasuredValue : Real;             // Messwert
      Quality : Byte;                   // Messwert-Qualität 0-100%
      Temperature : Int;                // Sensor-Temperatur
   END_STRUCT;
END_TYPE

// PlcToDrv für Sensoren
TYPE "ST_Sensor_PlcToDrv"
   STRUCT
      ControlWord : Struct
         Enable : Bool;
         StartMeasurement : Bool;       // Messung starten
         Calibrate : Bool;              // Kalibrierung
         Reset : Bool;
      END_STRUCT;
      MeasurementMode : Byte;           // Messmodus
      SamplingRate : Int;               // Abtastrate
      FilterSetting : Byte;             // Filtereinstellung
   END_STRUCT;
END_TYPE
```

## Schnittstellen-Mapping im Anwenderprogramm

### Standard-Verdrahtung
```scl
// Im Main-Programm (OB1):
"FB_[Gerät]_Instance"(
    // Standard-Eingänge
    ip_Maschine := 1,
    ip_Station := 10,
    ip_Bmk := 'Station_10_[Gerät]',
    i_StoerungQuittieren := "HMI_Global".Reset,
    i_NotHalt_IO := "Safety_IO".NotHalt,
    i_Sicherheitskreis_IO := "Safety_IO".SafetyCircuit,
    i_FreigabeBewegung := "Process_Enable".Movement,
    i_FreigabeHMI := "Process_Enable".HMI,
    
    // Hardware-Schnittstellen
    i_[Gerät]DrvToPlc := "IO_[Gerät]_Input",        // Eingangsbaugruppe
    q_[Gerät]PlcToDrv => "IO_[Gerät]_Output",       // Ausgangsbaugruppe
    
    // Standard-Ausgänge
    q_[Gerät]Ready => "[Gerät]_Status".Ready,
    
    // HMI-Schnittstelle
    [Gerät]_HMI := "[Gerät]_HMI_Data",              // Globale HMI-Struktur
    iq_Stoermeldeindikator := "[Gerät]_Alarm".Active
);
```

### HMI-Datenstruktur anlegen
```scl
// Globale Datenbaustein für HMI
DATA_BLOCK "[Gerät]_HMI_Data"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
NON_RETAIN
"ST_[Gerät]_HMI"
BEGIN
   Eingang.TargetPosition := 1000;
   Eingang.Velocity := 50;
   Eingang.Acceleration := 25;
END_DATA_BLOCK
```

## Häufige Hersteller-Patterns

### Siemens Antriebe (Sinamics, etc.)
```scl
// Typische Strukturen aus Siemens-Beispielen:
ZSW1, ZSW2 : Word;        // Zustandswort 1+2 (StatusWord)
STW1, STW2 : Word;        // Steuerwort 1+2 (ControlWord)  
XIST1 : DInt;             // Ist-Position
NSOLL_A : DInt;           // Soll-Drehzahl
```

### Beckhoff Antriebe (AX5000, etc.)
```scl
// Typische Strukturen aus Beckhoff-Beispielen:
Status : WORD;            // Status Register
Control : WORD;           // Control Register
ActPos : DINT;            // Actual Position
SetPos : DINT;            // Set Position
```

### SEW Antriebe (Movimot, etc.)
```scl
// Typische Strukturen aus SEW-Beispielen:
StatusRegister : WORD;
CommandRegister : WORD;
ActualPosition : DINT;
TargetPosition : DINT;
```

### Festo Pneumatik/Servo
```scl
// Typische Strukturen aus Festo-Beispielen:
StatusByte : BYTE;
ControlByte : BYTE;
PositionActual : INT;
PositionSetpoint : INT;
```

## Template-Verwendung

Diese erweiterte Vorlage bietet:

### ✅ **Herstellerspezifische Anpassung**
- Systematische Nutzung von Hersteller-Dokumentation
- Beibehaltung von Original-Bezeichnungen und Bit-Zuordnungen  
- Validierte Timing-Parameter aus Handbüchern

### ✅ **Bewährte Code-Patterns**
- Vollständige FB-Struktur basierend auf Hersteller-Beispielen
- Standard-Regionen angepasst an typische Geräte-Workflows
- Zustandsmaschinen entsprechend Hersteller-Spezifikationen

### ✅ **Qualitätssicherung durch Dokumentation**
- Pflichtanalyse der Hersteller-Unterlagen vor Implementierung
- Checklisten für vollständige Dokumentations-Erfassung
- Validierung gegen Original-Spezifikationen

**Die Vorlage berücksichtigt jetzt die wichtige Rolle der Herstellerdokumentation und leitet zur systematischen Nutzung dieser Informationsquellen an.**

*Diese Vorlage basiert auf der bewährten Architektur des FB_JennyScience_Epos und gewährleistet einheitliche Struktur und Wartbarkeit.*
