# Simulazione LTspice — Handoff per sessioni future

## Contesto
Simulazione del circuito Step 1 (pulse-echo singolo elemento, trasduttore 5MHz) prima dell'acquisto/assemblaggio fisico dei componenti, per verificare errori di dimensionamento. Software: **LTspice** (gratuito, Analog Devices). L'utente ha già esperienza pregressa con KiCad ma non con simulazione SPICE.

> ⚠️ **Revisione del 29/09/2026** (vedi `revisione-progetto-29-09-2026.md`): una revisione approfondita ha individuato diversi problemi in questo file, non ancora verificati su `Eco.asc` con una simulazione `.op`/`.ac` reale — sono calcoli a mano [C] o ipotesi [I], non risultati di simulazione. Legenda usata da qui in poi: **[V]** verificato sul file reale, **[C]** calcolato a mano (non simulato), **[I]** ipotesi/stima, **[?]** aperto, richiede verifica. I dettagli completi sono nel file di revisione.

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
| L1 | ~~10µH~~ → **≈87µH [C]** | Primario trasformatore — ricalcolato (29/09/2026) da dimensionamento DCM reale: f=100kHz, Vin=12V, D=0.4, Vout=142V, Pout≈1W, η=75%. Il valore 10µH era una stima di partenza mai verificata sul nucleo reale. Vedi `revisione-progetto-29-09-2026.md` Sez. 5.1 per il calcolo completo |
| L2 | ~~1mH~~ → **≈4,3mH [C]** | Secondario trasformatore — rapporto spire Np:Ns ≈ 7:1 (20:140 spire stimate), non più 10:1 |
| Accoppiamento | `K1 L1 L2 0.98` → **K≈0,95-0,98 [C]** | Il valore K=1 usato nelle direttive finali (sotto) era probabilmente un refuso — K=1 è fisicamente irraggiungibile (accoppiamento perfetto), va corretto |
| D1 | 1N4007 | Rettificatore uscita boost (sostituto di UF4007, non disponibile in libreria base) — **[?] nota aperta**: a 100kHz un 1N4007 reale (recovery lento) non sarebbe rappresentativo; nel circuito fisico va usato UF4007 (già in carrello), la simulazione con 1N4007 sottostima il problema di recovery |
| C1 | 47µF | Filtro uscita boost (elettrolitico HV, valore corretto — NON usare il capacitore pulse WIMA qui, errore di dimensionamento individuato e corretto in una sessione precedente) |
| R1 | 1Ω | Current sense per controller PWM — **[C] verificato compatibile**: Ipk primario stimato ≈0,55A → caduta ≈0,55V, sotto la soglia tipica di corrente UC3843 |
| R2 | ~~10kΩ~~ → **≈560kΩ (o 2×270kΩ in serie) [C]** | Partitore feedback (superiore) — **ricalcolato**, vedi nota critica sotto |
| R3 | ~~1kΩ~~ → **10kΩ [C]** | Partitore feedback (inferiore) — ricalcolato insieme a R2 |

### ⚠️ Correzione critica — partitore di feedback boost mai dimensionato per 142V
Il valore precedente (R2=10kΩ, R3=1kΩ) **non produce 142V** una volta chiuso il loop di feedback reale sull'UC3843BN: con riferimento interno ≈2,5V, `Vout = 2,5×(1+R2/R3) = 2,5×(1+10) = 27,5V`, non 142V. Il "boost stabile a 142V" osservato in simulazione era conseguenza del **duty cycle fisso** imposto da V2/V3/U1 (il loop non era mai chiuso — R2/R3 erano solo monitoraggio, vedi netlist sotto), non di una regolazione funzionante.

**Valori corretti per Vout≈142V:** R3=10kΩ, R2≈560kΩ (split in 2×270kΩ per tenuta di tensione) → dissipazione ≈35mW (trascurabile, contro i ≈0,35W della combinazione errata R2=56kΩ/R3=1kΩ che si otterrebbe scalando ingenuamente il rapporto). **Aggiungere uno zener di clamp sul pin feedback** come sicurezza contro fuga di tensione se il partitore si scollega accidentalmente.

**Impatto sulla lista acquisti:** il codice MF01SFF1002A10 (10kΩ) a 0 stock serve ora per R3 + R7 + R9 + R10 (4 pezzi, non più R2). R2 richiede invece 2× resistori 270kΩ, non ancora in lista.

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
| C3 | 20pF | Capacità statica cristallo (ramo parallelo) — **[I] irrealistico**: per un disco PZT Ø20mm a 5MHz l'ordine di grandezza atteso è nF (stima 1-12nF secondo εr/spessore/area), non pF. Da misurare sul trasduttore reale appena arriva (procedura in `revisione-progetto-29-09-2026.md` Sez. 6) |
| R5 | 50Ω | Ramo serie (resistenza) |
| L3 | 50µH | Ramo serie (induttanza) |
| C4 | ~~20fF~~ | Ramo serie (capacità) — **⚠️ [?] incoerenza non risolta**: con L3=50µH e C4=20fF la risonanza serie teorica sarebbe ≈159MHz, non i 5,03MHz misurati in simulazione. Il valore di risonanza osservato è numericamente coerente con L=50µH e **C=20pF** (non 20fF) — ma non è chiaro se la risonanza osservata provenga davvero dal ramo serie o da un'interazione con C3 (che è in parallelo, non in serie a L3). **Richiede uno sweep `.ac` dedicato** sulla sola impedenza del trasduttore per chiarire l'origine reale prima di fidarsi del modello |

### Blocco amplificatore RX
| Rif. | Valore | Note |
|---|---|---|
| U2 | **LT1360** (modello reale libreria LTspice) | 50MHz banda, 800V/µs — sostituisce LM6172IN (non trovato in libreria base). Banda ad anello chiuso effettiva ridotta dal guadagno impostato (~50MHz/11 ≈ 4.5MHz nominale, margine ancora sufficiente su 5MHz ma striminzito — da monitorare in simulazione, eventuale upgrade a LT1224 se segnale troppo attenuato) |
| R6 | 1kΩ | Rete di guadagno (ingresso) — **⚠️ [?] bug potenziale, non verificato**: nella netlist sotto, R6(2) va a GND senza capacitore in serie. Con RX_IN polarizzato a ≈6V (dal partitore R9/R10), un guadagno DC ×11 su quel bias richiederebbe un'uscita teorica di ~66V — impossibile, l'op-amp satura sul binario (~11-12V). **Se confermato con `.op` su `Eco.asc`** (leggere V(RX_IN), V(RX_FB), V(RX_OUT)), la correzione è: inserire un condensatore 100nF-1µF **in serie a R6** (tra nodo RX_FB e R6), che sposta il guadagno a ×1 in DC e ×11 solo in AC (frequenza di taglio ≈160Hz-1,6kHz, lontanissima da 5MHz) |
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
| **TX_NODE** | C2(2), M2(drain), D2(catodo), C3(1), R5(1), R4(1) — nodo condiviso pulser/trasduttore. Da qui esce SOLO R4 verso il T/R switch |
| **D23_MID** | D2(anodo), D3(catodo) — nodo intermedio serie tra D2 e D3, corretto dopo revisione utente (D2/D3 in serie, non in parallelo — solo D3(anodo) va a GND) |
| **PULSE_M2** | V4(+), M2(gate) |
| **RX_IN** | R4(2), D4(1), D5(1), U2(ingresso +), R9(2), R10(1) — R4 in serie tra TX_NODE e RX_IN; D4/D5 (clamp verso GND) collegati su RX_IN, A VALLE di R4, non su TX_NODE |
| **RX_FB** | U2(ingresso −), R6(1), R7(1), C5(1) |
| **RX_OUT** | U2(uscita), R7(2), C5(2) |

**Procedura:** cancellare tutti i fili fisici esistenti, usare F4 su ogni pin per assegnare il nome nodo corretto dalla tabella sopra. Verificare un blocco alla volta (boost, poi pulser+T/R+trasduttore, poi RX) prima di procedere al successivo.

## ⚠️ CORREZIONE — errore logico nella netlist T/R switch (versione precedente di questo file)
Una versione precedente di questa tabella documentava erroneamente D4/D5 collegati direttamente su `TX_NODE`, con R4 a valle verso `RX_IN`. **Era un errore di documentazione, non del circuito reale** (lo schema LTspice fisico ha sempre avuto il wiring corretto). Sequenza CORRETTA, obbligatoria:

1. `TX_NODE` → un capo di **R4**
2. L'altro capo di R4 → nodo `RX_IN`
3. Su `RX_IN`, i diodi incrociati **D4/D5** verso GND (clamp)
4. Da `RX_IN` → ingresso positivo dell'amplificatore (U2)

**Motivo fisico:** se D4/D5 fossero collegati direttamente su TX_NODE (senza R4 a monte), dovrebbero assorbire l'intera corrente di scarica del pulser (~140V su C2 4.7nF) senza alcun limite — distruzione istantanea dei diodi, nessun impulso trasmesso al trasduttore. Con R4 (100Ω) prima dei diodi, il picco HV viene limitato in corrente (I ≈ 130V/100Ω ≈ 1.3A) prima che i diodi clampino il residuo a ±0.6-0.7V — è anche il motivo per cui R4 deve essere un resistore 1-2W film metallico (tenuta al picco istantaneo), non un 1/4W standard. In ricezione, l'eco (mV) attraversa R4 con perdita trascurabile verso l'alta impedenza d'ingresso dell'op-amp, mentre i diodi restano spenti sotto la soglia di conduzione (~0.7V).

## ⚠️ BOM incompleta per UC3843BN (nota aggiunta in revisione 29/09/2026)
Nella simulazione, i componenti di contorno tipici dell'UC3843BN (RT/CT per fissare la frequenza di oscillazione, rete di compensazione sul pin COMP, filtro RC sul pin current sense, resistenza di gate + pulldown) sono sostituiti da generatori/blocchi ideali (V2/V3/U1/S1). Nel circuito fisico questi componenti **mancano dalla checklist acquisti** — da aggiungere prima di ordinare (vedi `step1_componenti_priorita_tme.md`).

## COMPONENTI AGGIUNTI DURANTE IL DEBUG (non nella lista acquisti fisici originale)
| Rif. | Valore | Ruolo | Da valutare per hardware reale? |
|---|---|---|---|
| R11 | 1kΩ | Resistenza gate in serie tra S1 e gate M1 (buona pratica) | Sì, aggiungere |
| R12 | 1kΩ | Resistenza di carica in serie tra D2 e C2 (limita inrush) | Sì, aggiungere |
| C6 | 10µF | Accoppiamento AC tra T/R switch e RX_IN | Sì, verificare valore su circuito reale |

## BUG CRITICO RISOLTO — errore sintassi unità di misura
Causa di ~4 iterazioni di debug (BOOST_OUT a 0V/femtovolt): sintassi SBAGLIATA `10 L`, `47 C`, `50 L`, `10 C` (spazio + lettera unità) → SPICE legge in unità BASE (Henry/Farad), errore ×1.000.000. **Corretto in:** `10u`, `47u`, `50u`, `10u` (numero+prefisso attaccato, no spazio, no lettera finale). Componenti: L1, C1, L3, C6.

## DIRETTIVE FINALI (sostituiscono le versioni precedenti) — ⚠️ da aggiornare con i valori ricalcolati
```
.model MYFET NMOS(Vto=4 Kp=20 Rd=0.1)
.model SW SW(Ron=1 Roff=1Meg Vt=2.5 Vh=0.1)
K1 L1 L2 0.95          ← CORRETTO (29/09/2026): K=1 era un refuso, fisicamente irraggiungibile
.tran 0 3 0 5n          (validazione completa fino a stabilizzazione boost)
.tran 0 1.1m 1m 5n      (debug rapido su finestra specifica)
```
**Nota risolta:** K1=1 nella versione precedente era effettivamente un refuso (confermato in revisione) — un accoppiamento perfetto (K=1) non è fisicamente ottenibile in un trasformatore reale. Usare K≈0,95-0,98. **Da applicare insieme ai nuovi valori L1≈87µH, L2≈4,3mH** (vedi tabella Blocco boost HV sopra) — i tre cambiamenti vanno fatti insieme prima di rilanciare la simulazione.

## ⚠️ RISULTATO FINALE — SIMULAZIONE VALIDATA SOLO PARZIALMENTE (rivisto 29/09/2026)

*Titolo precedente ("COMPLETAMENTE VALIDATA") non più accurato — una revisione ha individuato problemi non verificati. Vedi `revisione-progetto-29-09-2026.md` per l'analisi completa.*

| Blocco | Esito | Valore misurato | Stato dopo revisione |
|---|---|---|---|
| Boost (BOOST_OUT) | Duty fisso, non regolazione reale | Si stabilizza a **~142-143V** | **[?] Il loop di feedback non era mai chiuso** (R2/R3 solo monitoraggio) — il valore 142V è conseguenza del duty cycle fisso imposto da V2/V3, non di un partitore funzionante. Con R2/R3 originali e loop chiuso si otterrebbero solo 27,5V. Ricalcolati R2≈560kΩ, R3=10kΩ (vedi sopra) |
| Pulser (TX_NODE) | Plausibile, non rivalidato | Impulso netto 200ns coerente | **[?]** La topologia C2/D2/D3/M2/R12 non è stata riverificata in dettaglio — non è chiaro dalla netlist come C2 si carichi esattamente. Da rivedere |
| T/R switch | ✅ Validato | Limita da ±140V (pre-diodi) a ±0.6-0.7V (post-diodi) | Risultato confermato, non impattato dalla revisione |
| Trasduttore (Butterworth-Van Dyke) | Numericamente plausibile, origine non chiarita | Risonanza misurata: **5.032.921 Hz ≈ 5,03MHz** | **[?]** Coerente con L=50µH e C=20pF, ma il componente C4 nel modello è 20fF (non 20pF) — non chiaro se la risonanza osservata provenga dal ramo serie o da un'interazione con C3. Richiede sweep `.ac` dedicato |
| RX (RX_OUT) | **Sospetto bug non verificato** | Baseline ~6V (bias corretto) | **[?]** Se R6 non ha condensatore in serie (vedi tabella Blocco RX sopra), un guadagno DC ×11 sul bias di 6V dovrebbe saturare l'uscita (~66V teorici vs ~11-12V di binario) — il valore "6V" osservato non è ovviamente coerente con questo scenario. Possibili spiegazioni non verificate: misura presa su RX_IN invece di RX_OUT, condensatore presente ma non documentato, transitorio non assestato. **Richiede `.op` su `Eco.asc`** per chiarire, prima di qualsiasi altra modifica al blocco RX |

**Conclusione aggiornata:** il circuito **non è pienamente validato** come affermato in precedenza — restano almeno due punti [?] da chiarire con una simulazione `.op`/`.ac` reale (feedback boost, blocco RX) prima di procedere con fiducia all'assemblaggio fisico di quei blocchi specifici. Il T/R switch resta confermato. Il bug di sintassi unità di misura (sotto) resta comunque risolto correttamente. Metodo di debug efficace confermato: procedere blocco per blocco, zoom progressivi sul grafico per isolare fenomeni ad alta frequenza.

## Prossimi passi possibili — AGGIORNATO (ordine di lavoro concordato in revisione)
1. **Eseguire `.op` su `Eco.asc`**; leggere V(RX_IN), V(RX_FB), V(RX_OUT) — chiude il dubbio sul bug R6
2. Ordinare **solo il trasduttore 5P20N** nel frattempo (il resto del carrello TME resta congelato in attesa dei ricalcoli)
3. Alla consegna del trasduttore: misurare C0 e curva di impedenza reale (procedura in `revisione-progetto-29-09-2026.md` Sez. 6), poi eco da piastra attraverso parete PP+gel
4. Aggiornare `Eco.asc` con i valori misurati; chiudere davvero il loop di feedback boost (R2≈560kΩ, R3=10kΩ); ricostruire il modello BVD con i parametri reali
5. Calcolare/avvolgere il trasformatore secondo i nuovi valori (L1≈87µH, L2≈4,3mH, 20:140 spire); progettare l'interfaccia U2→ADC (non ancora esistente in nessun file, vedi nota sotto); definire il clock PIO a 31,25MHz (non 32MHz, vedi `step1_pulse_echo_singolo_elemento.md`)
6. Solo a quel punto confermare la distinta acquisti TME aggiornata

## ⚠️ Nota aperta — interfaccia U2 → ADC mai progettata
In nessun file del progetto l'uscita di U2 (RX_OUT, bias ≈6V) risulta collegata all'ingresso dell'AD9280 (AVDD tipicamente 2,7-3,6V, span atteso 0-2V). Un collegamento diretto rischia sovratensione sull'ingresso dell'ADC. Serve progettare: accoppiamento AC dall'uscita di U2, ribias a ≈1V (centro dello span 0-2V, es. partitore ~22k/10k da un riferimento 3,3V), clamp (Schottky) verso AVDD/GND dell'ADC, filtro anti-aliasing. Dettagli e valori indicativi in `revisione-progetto-29-09-2026.md` Sez. 5.4 — non ancora un progetto definitivo, richiede verifica di span/REFSENSE sul datasheet AD9280.