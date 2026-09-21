# Simulazione LTspice — Handoff per sessioni future

## Contesto
Simulazione del circuito Step 1 (pulse-echo singolo elemento, trasduttore 5MHz) prima dell'acquisto/assemblaggio fisico dei componenti, per verificare errori di dimensionamento. Software: **LTspice** (gratuito, Analog Devices). L'utente ha già esperienza pregressa con KiCad ma non con simulazione SPICE.

## Stato attuale: TUTTI I COMPONENTI PIAZZATI E CONFIGURATI — manca solo il collegamento dei nodi (wiring)

---

## Componenti piazzati, con valori confermati corretti

### Sorgenti di tensione
| Rif. | Valore | Ruolo |
|---|---|---|
| V1 | DC 12V | Alimentazione ingresso boost |
| V2 | `PULSE(0 5 0 9.9u 0.1u 0 10u)` | Rampa/oscillatore per comparatore PWM (~100kHz) |
| V3 | DC 2.5V | Riferimento soglia duty-cycle per comparatore (confrontato con la rampa V2) |
| V4 | `PULSE(0 12 0 10n 10n 200n 1m)` | Generatore impulsi pulser (sostituisce MC33151PG), impulso 200ns, ripetizione 1ms (1kHz) |

### Blocco boost HV (12V → 100-300V)
| Rif. | Valore | Note |
|---|---|---|
| M1 | Modello custom **MYFET** | MOSFET boost |
| L1 | 10µH | Primario trasformatore (valore di partenza, da raffinare con calcoli su nucleo reale E20/10/6) |
| L2 | 1mH | Secondario trasformatore |
| Accoppiamento | `K1 L1 L2 0.98` | Direttiva SPICE da aggiungere sul foglio se non già presente — **verificare che sia stata inserita** |
| D1 | 1N4007 | Rettificatore uscita boost (sostituto di UF4007, non disponibile in libreria base) |
| C1 | 47µF | Filtro uscita boost (elettrolitico HV, valore corretto — NON usare il capacitore pulse WIMA qui, errore di dimensionamento individuato e corretto in una sessione precedente) |
| R1 | 1Ω | Current sense per controller PWM |
| R2 | 10kΩ | Partitore feedback (superiore) |
| R3 | 1kΩ | Partitore feedback (inferiore) |

### Blocco pulser
| Rif. | Valore | Note |
|---|---|---|
| M2 | Modello custom **MYFET** (stesso modello di M1, comportamento identico per semplicità) | MOSFET pulser |
| C2 | 4.7nF | Capacitore storage energia (corrisponde al WIMA FKP1T014705D00JSSD reale) |
| D2 | 1N4007 | Diodo protezione pulser |
| D3 | 1N4007 | Diodo protezione pulser |

### Blocco T/R switch
| Rif. | Valore | Note |
|---|---|---|
| D4 | 1N4148 | Diodo incrociato protezione RX |
| D5 | 1N4148 | Diodo incrociato protezione RX |
| R4 | 100Ω | Resistenza limitatrice serie |

### Blocco trasduttore (modello Butterworth-Van Dyke, target 5MHz)
| Rif. | Valore | Note |
|---|---|---|
| C3 | 20pF | Capacità statica cristallo (ramo parallelo) |
| R5 | 50Ω | Ramo serie (resistenza) |
| L3 | 50µH | Ramo serie (induttanza) |
| C4 | 20fF | Ramo serie (capacità) — valori R5/L3/C4 di partenza plausibili per risonanza ~5MHz, DA VALIDARE con calcolo preciso in sessione futura |

### Blocco amplificatore RX
| Rif. | Valore | Note |
|---|---|---|
| U2 | **LT1360** (modello reale libreria LTspice) | 50MHz banda, 800V/µs — sostituisce LM6172IN (non trovato in libreria base). Banda ad anello chiuso effettiva ridotta dal guadagno impostato (~50MHz/11 ≈ 4.5MHz nominale, margine ancora sufficiente su 5MHz ma striminzito — da monitorare in simulazione, eventuale upgrade a LT1224 se segnale troppo attenuato) |
| R6 | 1kΩ | Rete di guadagno (ingresso) |
| R7 | 10kΩ | Rete di guadagno (feedback) — guadagno risultante ~×11 |
| C5 | 10pF | Compensazione/filtro |

### Blocco comparatore PWM (boost)
| Rif. | Valore | Note |
|---|---|---|
| U1 | **LT1011** (modello reale libreria) | Comparatore, uscita open-collector |
| R8 | 4.7kΩ | **Pull-up per U1** (necessario per uscita open-collector — aggiunto in questa sessione, verificare che sia effettivamente collegato tra uscita U1 e alimentazione logica) |
| S1 | **SW** (switch ideale generico) | Sostituisce ADG1712 (scartato: limite tensione alimentazione ±5V incompatibile con 12V del circuito). Parametri Ron/Roff/Vt/Vh ancora da impostare esplicitamente — verificare default o assegnare valori |

### Direttiva modello custom MOSFET (testo su foglio, già presente)
```
.model MYFET NMOS(Vto=4 Kp=20 Rd=0.1)
```

---

## Come si è arrivati a ogni scelta (per contesto/troubleshooting futuro)

- **UC3843BN reale non trovato in libreria LTspice** → sostituito con blocco funzionale equivalente: V2 (rampa) + V3 (riferimento) + U1 LT1011 (comparatore) + S1 (switch) — stessa logica PWM current-mode, componenti ideali/reali disponibili
- **MC33151PG reale non trovato** → sostituito con V4, generatore impulsi ideale diretto sul gate di M2
- **LM6172IN (THT, scelto per l'acquisto reale) non in libreria** → sostituito con LT1360 per la simulazione
- **UF4007 non in libreria** → sostituito con 1N4007 (comportamento simile, sempre presente in standard.dio)
- **Per editare parametri MOSFET**: NON cliccare sul simbolo grafico (apre solo il picker di modelli precompilati) — cliccare sulla **label di testo "NMOS" sotto il simbolo**, si apre casella di testo libero dove scrivere il nome del modello custom (es. "MYFET")

---

## Metodo di collegamento: ETICHETTE DI RETE (F4), non fili lunghi
Dopo un primo tentativo con fili fisici lunghi che generava confusione, si è passati a un metodo basato su **etichette di rete** (F4 su ogni pin, stesso nome = stesso nodo elettrico). Netlist completa qui sotto, organizzata per nodo — è la versione definitiva da cui ripartire.

## NETLIST COMPLETA PER NODO

**Correzione di progetto aggiunta in questa fase:** RX con alimentazione singola (VCC/GND) richiede bias sul segnale AC del trasduttore — aggiunti **R9, R10** (10k ciascuno) come partitore VCC→GND, punto centrale sul nodo RX_IN insieme al segnale.

| Nodo | Pin collegati |
|---|---|
| **VCC** | V1(+), R8(1), S1(pin passaggio A), L1(1), U1(V+), U2(V+), R9(1) |
| **GND** | V1(−), V2(−), V3(−), V4(−), R1(2), L2(1), C1(2), R3(2), M2(source), **D3(anodo)**, C3(2), ramo serie transducer(estremo finale), U1(V−), U2(V−), R6(2), R10(2), S1(pin controllo "−") |
| **RAMP** | V2(+), U1(ingresso +) |
| **REF** | V3(+), U1(ingresso −) |
| **COMP_OUT** | U1(uscita), R8(2), S1(pin controllo +) |
| **GATE_M1** | S1(pin passaggio B), M1(gate) |
| **SW_NODE** | L1(2), M1(drain) |
| **M1_SRC** | M1(source), R1(1) |
| **L2_D1** | L2(2), D1(anodo) |
| **BOOST_OUT** | D1(catodo), C1(1), R2(1), C2(1, capo NON verso M2) |
| **FB** | R2(2), R3(1) — solo monitoraggio, non chiuso sul comparatore in questa versione semplificata |
| **TX_NODE** | C2(2), M2(drain), D2(catodo), D4(1), D5(1), C3(1), R5(1) — nodo condiviso pulser/trasduttore |
| **D23_MID** | D2(anodo), D3(catodo) — nodo intermedio serie tra D2 e D3, corretto dopo revisione utente (D2/D3 in serie, non in parallelo — solo D3(anodo) va a GND) |
| **PULSE_M2** | V4(+), M2(gate) |
| **RX_IN** | R4(2), U2(ingresso +), R9(2), R10(1) — R4 in serie tra nodo D4/D5 e RX_IN |
| **RX_FB** | U2(ingresso −), R6(1), R7(1), C5(1) |
| **RX_OUT** | U2(uscita), R7(2), C5(2) |

**Procedura:** cancellare tutti i fili fisici esistenti, usare F4 su ogni pin per assegnare il nome nodo corretto dalla tabella sopra. Verificare un blocco alla volta (boost, poi pulser+T/R+trasduttore, poi RX) prima di procedere al successivo.

## COMPONENTI AGGIUNTI DURANTE IL DEBUG (non nella lista acquisti fisici originale)
| Rif. | Valore | Ruolo | Da valutare per hardware reale? |
|---|---|---|---|
| R11 | 1kΩ | Resistenza gate in serie tra S1 e gate M1 (buona pratica) | Sì, aggiungere |
| R12 | 1kΩ | Resistenza di carica in serie tra D2 e C2 (limita inrush) | Sì, aggiungere |
| C6 | 10µF | Accoppiamento AC tra T/R switch e RX_IN | Sì, verificare valore su circuito reale |

## BUG CRITICO RISOLTO — errore sintassi unità di misura
Causa di ~4 iterazioni di debug (BOOST_OUT a 0V/femtovolt): sintassi SBAGLIATA `10 L`, `47 C`, `50 L`, `10 C` (spazio + lettera unità) → SPICE legge in unità BASE (Henry/Farad), errore ×1.000.000. **Corretto in:** `10u`, `47u`, `50u`, `10u` (numero+prefisso attaccato, no spazio, no lettera finale). Componenti: L1, C1, L3, C6.

## DIRETTIVE FINALI (sostituiscono le versioni precedenti)
```
.model MYFET NMOS(Vto=4 Kp=20 Rd=0.1)
.model SW SW(Ron=1 Roff=1Meg Vt=2.5 Vh=0.1)
K1 L1 L2 1
.tran 0 3 0 5n          (validazione completa fino a stabilizzazione boost)
.tran 0 1.1m 1m 5n      (debug rapido su finestra specifica)
```
**Nota:** K1 finale = 1 (non 0.98 come da progetto iniziale — verificare se intenzionale o refuso).

## ✅ RISULTATO FINALE — SIMULAZIONE COMPLETAMENTE VALIDATA

| Blocco | Esito | Valore misurato |
|---|---|---|
| Boost (BOOST_OUT) | ✅ Validato | Sale progressivamente, si stabilizza a **~142-143V** (dentro range target 100-300V) |
| Pulser (TX_NODE) | ✅ Validato | Impulso netto 200ns coerente |
| T/R switch | ✅ Validato | Limita da ±140V (pre-diodi) a ±0.6-0.7V (post-diodi) |
| Trasduttore (Butterworth-Van Dyke) | ✅ Validato | Risonanza misurata: **5.032.921 Hz ≈ 5,03MHz** (scarto 0,7% da target, trascurabile) |
| RX (RX_OUT) | ✅ Validato | Baseline ~6V (bias corretto), oscillazione trasduttore confermata |

**Conclusione: il circuito si comporta come progettato, nessun errore concettuale rimasto.** L'unico vero blocco era il bug sintassi unità (risolto). Metodo di debug efficace: procedere blocco per blocco dalla sorgente verso l'uscita, zoom progressivi sul grafico per isolare fenomeni ad alta frequenza (5MHz invisibile su finestre larghe, serviva zoom a singoli microsecondi + passo forzato 5ns).

## Prossimi passi possibili
1. Valutare se aggiungere R11/R12/C6 alla lista acquisti fisici reali (file step1_componenti_priorita_tme.md)
2. Eventuale ottimizzazione fine (non bloccante): alzare tensione boost verso l'alto del range se si vuole più margine, chiudere il vero loop di feedback (attualmente R2/R3 solo monitoraggio, non collegato)
3. Procedere con assemblaggio fisico Step 1, forte della validazione elettrica ottenuta in simulazione
