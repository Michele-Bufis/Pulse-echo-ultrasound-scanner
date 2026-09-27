# Step 1 — Pulse-echo singolo elemento (A-mode statico)

## Obiettivo

Validare l'intera catena elettronica di base del sistema — trasduttore, pulser, protezione RX, amplificazione, ADC, firmware di acquisizione — usando **un solo trasduttore fisso** (nessuno switching, nessuna rotazione). L'output atteso è un grafico ampiezza vs tempo (o vs profondità) che mostra chiaramente gli echi di ritorno da un target di test in acqua.

Questo step non produce ancora un'immagine: è il "sistema nervoso" del progetto, quello su cui poi si costruisce tutto il resto (switching a 4 elementi, rotazione, B-mode). Se qui il segnale è pulito e il timing è affidabile, tutti gli step successivi diventano un problema di *estensione*, non di *fondamenta*.

**Criterio di successo:** vedere sullo schermo (oscilloscopio o plot software) un impulso di trasmissione netto seguito, a un tempo coerente con la distanza reale, da uno o più echi riconoscibili sopra il rumore di fondo.

---

## Introduzione ai componenti e a cosa fanno

### Trasduttore piezoelettrico (1×)
Elemento che converte energia elettrica in onda acustica (in trasmissione) e onda acustica in segnale elettrico (in ricezione). È lo stesso elemento fisico a fare entrambi i lavori in tempi diversi. Per questo step: frequenza 2-5MHz, contatto singolo (non serve array).

### Pulser
Circuito che genera l'impulso elettrico ad alta tensione (tipicamente 50-200V, durata di decine-centinaia di nanosecondi) che eccita il trasduttore in trasmissione. Più l'impulso è breve e netto, migliore sarà la risoluzione assiale del sistema.

### T/R switch (Transmit/Receive switch)
Circuito di protezione tra il pulser e lo stadio di amplificazione. Subito dopo lo sparo HV, il trasduttore diventa un ricevitore ultra-sensibile (segnali dell'ordine dei millivolt) — senza questo switch, il picco HV del pulser distruggerebbe o saturerebbe l'amplificatore. Tipicamente realizzato con diodi incrociati (limitano passivamente la tensione che arriva all'amplificatore) o uno switch analogico attivo.

### Amplificatore RX (+ eventuale TGC)
Amplifica il debole segnale di eco fino a un livello leggibile dall'ADC. In questo step puoi anche partire con un guadagno fisso (senza TGC variabile nel tempo) — il TGC lo introduci quando inizi a lavorare a profondità maggiori o con target più attenuanti.

### ADC (Analog-to-Digital Converter)
Campiona il segnale analogico amplificato trasformandolo in una sequenza di numeri che il microcontrollore può processare. Per questo step: minimo 10-12 bit, almeno 20-30 Msps.

### Microcontrollore (RP2040)
Coordina tutto: genera il segnale di trigger che fa partire pulser e finestra di campionamento in sincronia (via PIO, per timing preciso), raccoglie i dati dall'ADC, li trasferisce al PC via USB/seriale.

### PC (per elaborazione e visualizzazione)
Riceve i dati grezzi dal microcontrollore e li trasforma in un grafico leggibile (ampiezza vs tempo/profondità) tramite uno script Python.

### Vasca con acqua + target di test
Il mezzo di accoppiamento acustico (l'acqua trasmette bene gli ultrasuoni, niente gel necessario per questo test preliminare) e un oggetto/parete che genera un eco netto per la validazione (es. una parete rigida della vasca stessa, o una piastra metallica posta a distanza nota).

### Oscilloscopio (≥100MHz)
Strumento di debug fondamentale in questa fase: ti permette di vedere direttamente, con i tuoi occhi, l'impulso di trasmissione e l'eco di ritorno **prima** ancora di fidarti del firmware/software — è il modo più veloce per capire se il problema (quando qualcosa non va) è nell'elettronica analogica o nel codice.

---

## Collegamenti e interazioni

```
                    ┌─────────────┐
                    │ Microcontrol.│
                    │   (RP2040)   │
                    └──┬───────┬──┘
                       │       │
              trigger  │       │  dati campionati
              (PIO)    │       │  (SPI/parallelo)
                       ▼       ▲
                 ┌─────────┐  ┌─────────┐
                 │ Pulser  │  │  ADC    │
                 └────┬────┘  └────▲────┘
                      │            │
                      │ impulso HV │ segnale amplificato
                      ▼            │
                 ┌──────────────────────┐
                 │   T/R switch         │
                 │ (protegge il ramo RX)│
                 └──────────┬───────────┘
                            │
                    ┌───────▼────────┐
                    │  Trasduttore   │
                    │  piezoelettrico│
                    └───────┬────────┘
                            │
                       (acqua/target)
                            │
                    eco di ritorno
                            │
                    ┌───────▼────────┐
                    │ Amplificatore  │
                    │  RX (+ TGC)    │
                    └────────────────┘
```

**Sequenza logica di un singolo "sparo" (ciclo completo):**

1. Il microcontrollore (via PIO) genera il segnale di trigger
2. Il trigger attiva il pulser, che scarica l'impulso HV verso il trasduttore attraverso il T/R switch
3. Il trasduttore converte l'impulso elettrico in onda acustica, che si propaga nell'acqua
4. Contemporaneamente, il T/R switch isola l'amplificatore RX dal picco HV
5. L'onda acustica colpisce il target, torna indietro come eco, il trasduttore la riconverte in segnale elettrico debole
6. Il T/R switch ora lascia passare questo segnale verso l'amplificatore RX
7. L'amplificatore alza il livello del segnale
8. L'ADC campiona il segnale amplificato per tutta la finestra temporale di ascolto
9. Il microcontrollore raccoglie i campioni e li invia al PC
10. Il PC (Python) elabora e visualizza il grafico ampiezza/tempo

**Punti di collegamento fisico da preparare:**
- Cavo coassiale schermato tra trasduttore e circuito pulser/T-R switch (il segnale RX è debole, uno schermo scadente introduce rumore)
- Massa comune ben progettata tra pulser (ramo HV), amplificatore (ramo basso rumore) e microcontrollore — è una causa frequente di rumore/artefatti se fatta male
- Alimentazione separata (o ben filtrata) tra la sezione HV del pulser e la sezione analogica di precisione dell'amplificatore, per non iniettare disturbi

---

## Cosa aspettarti, passo dopo passo

### Fase A — Verifica statica (senza acqua, senza software)
1. Alimenti il circuito, verifichi con multimetro che le tensioni di alimentazione siano corrette (in particolare l'HV del pulser, con cautela)
2. Con oscilloscopio sul pin di trigger del microcontrollore, verifichi che il firmware base generi effettivamente un impulso quando comandato (anche solo un LED di test o un pin GPIO, prima ancora di collegare il pulser vero)

**Aspettati:** un impulso pulito e ripetibile sul pin di trigger. Se manca o è irregolare, il problema è nel firmware/PIO, non ancora nell'elettronica RF.

### Fase B — Verifica del pulser a vuoto
3. Colleghi l'oscilloscopio direttamente ai capi del trasduttore (o a un carico resistivo equivalente, se vuoi evitare di stressare il trasduttore a vuoto)
4. Attivi il trigger e osservi l'impulso HV generato dal pulser

**Aspettati:** un impulso ad alta tensione, breve, con un certo overshoot/ringing (normale nei circuiti pulser semplici) — se è assente o deformato, il problema è nel pulser stesso.

### Fase C — Verifica del ramo RX isolato
5. Simuli un segnale di eco (con un generatore di funzioni, un piccolo impulso di test) direttamente in ingresso all'amplificatore RX, bypassando temporaneamente pulser e T/R switch
6. Verifichi che l'amplificatore lo alzi di livello in modo pulito, senza saturazione né rumore eccessivo

**Aspettati:** un segnale amplificato, riconoscibile, non distorto. Se saturi o è troppo rumoroso, il guadagno va ritarato prima di andare oltre.

### Fase D — Primo test in acqua, con oscilloscopio (il momento della verità)
7. Immergi il trasduttore in una vasca d'acqua, con una parete rigida (o piastra metallica) a distanza nota (es. 5-10cm)
8. Colleghi l'oscilloscopio in uscita dall'amplificatore RX
9. Attivi il ciclo trigger→sparo

**Aspettati:** sull'oscilloscopio, un primo picco netto (impulso di trasmissione, spesso "trapela" anche sul ramo RX nonostante il T/R switch — è normale, si chiama "ringing" o "breakthrough" e va accettato entro certi limiti), seguito — a un tempo calcolabile da `t = 2×distanza/velocità_suono_acqua (~1480 m/s)` — da un eco più piccolo ma riconoscibile.

**Se non vedi nulla:** prova ad aumentare il guadagno RX, verifica l'accoppiamento acustico (niente bolle d'aria sul trasduttore), verifica che il target sia effettivamente nel percorso del fascio.

---

## Analisi di link budget — cosa aspettarsi in termini di segnale, prima di guardare l'oscilloscopio

*Analisi fatta su carta prima dell'assemblaggio, per sapere cosa aspettarsi in Fase D e riconoscere subito se qualcosa non torna rispetto alle previsioni.*

### Passo 1 — Perdita di percorso totale
Percorso: wear plate → gel → parete vasca PP → acqua → target (pelle/tessuto) → ritorno.

Impedenze acustiche usate (MRayl): cristallo ~30, wear plate ~5 (stima), gel ~1,5, parete PP ~2,0, acqua 1,48, pelle/tessuto ~1,6.

Riflessione al target (pelle vs acqua, impedenze molto vicine):
```
R = ((1,6-1,48)/(1,6+1,48))² ≈ 0,0015 (0,15%)
```
Solo lo 0,15% dell'energia che arriva sul target torna indietro — coerente con la fisica nota (contorni ecografici netti solo dove l'impedenza cambia molto, es. osso; pelle/tessuto molle riflette pochissimo).

**Perdita totale di percorso (andata+ritorno) ≈ -34,6dB.**

*Verifica di robustezza:* facendo variare l'impedenza del wear plate (stimata, non da datasheet) tra 2 e 10 MRayl, il risultato varia solo tra -34,4dB e -36,7dB — un effetto di compensazione fisica (peggiora la trasmissione in uscita, migliora quella di rientro) rende il numero robusto anche con questa incertezza.

### Passo 2 — Il vero problema: main bang acustico, non solo perdita assoluta
Oltre al breakthrough elettrico del pulser (già noto: ±0.6-0.7V residui post T/R switch, da simulazione), esiste un secondo segnale forte e distinto: il **main bang acustico** — la riflessione quasi istantanea alla prima interfaccia (wear plate→gel), che rientra nel cristallo senza percorrere la vasca.
```
Riflessione wear→gel: R = 1 - T(wear→gel) = 1 - 0,710 = 0,290 (29%)
Trasmissione di ritorno wear→cristallo: T ≈ 0,490
Main bang totale: 0,290 × 0,490 = 0,142 → circa -8,5dB
```
**Differenza main bang vs eco target: 26,1dB** — è questo il vero problema da tenere d'occhio, non il numero assoluto di perdita.

Il gating temporale (scartare l'inizio della finestra) risolve la **confusione nel tempo** tra main bang ed eco target, ma **non risolve automaticamente** un'eventuale saturazione fisica dell'amplificatore RX (LM6172IN, sigla U2 nello schema di simulazione) — quella richiede che l'op-amp abbia il tempo di uscire dalla saturazione prima che arrivi il segnale utile.

### Passo 3 — Rischio di saturazione dell'amplificatore RX (LM6172IN)
Dal datasheet LM6172 (non esiste un dato diretto di "overload recovery time"; usato il **settling time** come proxy):
```
Settling time (0.1%): 65ns @±15V, 72ns @±5V — il circuito lavora a 12V singola alimentazione,
valore intermedio stimato ~70ns
Stima recovery da saturazione vera (fattore di margine tipico 5-10× il settling lineare): 350-700ns
```

Confronto con il margine di tempo disponibile prima che arrivi l'eco del target, per diverse distanze dal bordo vasca:
| Distanza dal bordo | Tempo di volo disponibile | Margine vs recovery pessimistico (700ns) |
|---|---|---|
| 7cm (dito, caso comodo) | 94,6µs | ~135× |
| 5cm (polso, caso base) | 67,6µs | ~97× |
| 3cm (polso decentrato, caso peggiore) | 40,5µs | ~58× |

**Conclusione: il rischio di sovrapposizione temporale main bang/target è trascurabile anche nel caso peggiore.** Resta però un limite: **il dato di sensibilità RX del trasduttore (µV/Pa) non è pubblicato** dal produttore (YUSHI/XMSJ, prodotto NDT economico) — senza quel dato non è possibile calcolare la tensione assoluta che il main bang genera su U2, quindi **non si può escludere con certezza matematica** che l'amplificatore saturi, solo che se satura probabilmente recupera in tempo utile. Argomento di plausibilità aggiuntivo: R4 (100Ω) + D4/D5 sono già dimensionati per il caso peggiore assoluto (breakthrough diretto 140V) — il main bang acustico è per costruzione fisica più debole di quello, quindi non introduce un percorso di rischio nuovo rispetto a quello già gestito dal T/R switch.

**Questo punto non è chiudibile su carta con certezza — verifica empirica necessaria in Fase D**, guardando specificamente l'uscita di U2 subito dopo lo sparo, prima ancora di collegare l'ADC.

### Mitigazioni a costo zero (già adottate)
1. **Gating firmware ampio:** scartare i primi 3-5µs di ogni acquisizione (non il minimo teorico stretto) — vedi `step2-rotazione-bmode.md`, Sez. 10, punto 5
2. **Margine operativo minimo nei test:** non centrare il target a meno di ~5cm dal bordo vasca durante le prove — vedi `step2-rotazione-bmode.md`, Sez. 10, punto 6

### Piano di contingenza — se in Fase D si osserva saturazione reale di U2
Ordine di intervento consigliato, dal meno al più invasivo:
1. **Allargare ulteriormente il gating** in firmware (zero costo, primo tentativo, 5 minuti)
2. **Ridurre il guadagno RX** abbassando R7 (attualmente 10kΩ, guadagno ×11 con R6=1kΩ) — es. dimezzare a ~5kΩ per un guadagno ×6; economico, reversibile su breadboard, va bilanciato con la leggibilità dell'eco debole del target
3. **Partitore resistivo aggiuntivo** su RX_IN, a monte di U2 — attenua main bang e target insieme senza toccare il guadagno; da validare in LTspice prima di saldare, per verificare che non attenui eccessivamente anche il target
4. **Diodi di clamp aggiuntivi** (idealmente Schottky, più veloci degli 1N4148 già presenti su RX_IN) direttamente sui pin d'ingresso di U2 — tecnica da front-end professionale; attenzione al carico capacitivo aggiuntivo su una banda già "striminzita" a 5MHz (nota da `simulazione_ltspice_handoff.md`)
5. **Blanking attivo** sincronizzato col trigger (switch/transistor che cortocircuita l'ingresso di U2 per un tempo fisso dopo lo sparo) — più efficace, ma aggiunge complessità firmware e un componente da validare
6. **TGC minimo** (guadagno variabile nel tempo) — ultima risorsa, già scartata a inizio progetto per complessità/budget

### Confronto quantificato delle opzioni di correzione
| Opzione | Costo | Tempo intervento | Efficacia | Rischio collaterale |
|---|---|---|---|---|
| 1. Gating esteso | 0€ | 5 min | Alta su confusione temporale, zero su saturazione fisica | Nessuno |
| 2. Riduzione R7 | ~0,10€ | 10-15 min | Alta — riduce swing uscita U2 | Riduce anche il target, in proporzione |
| 3. Partitore aggiuntivo RX_IN | ~0,20€ | 30-40 min (+sim. LTspice) | Media-alta, attenua tutto | Rischio attenuare troppo il target |
| 4. Diodi clamp aggiuntivi (Schottky) | ~0,50-1€ | 20-30 min | Alta sui picchi | Capacità parassita su banda già stretta (5MHz) |
| 5. Blanking attivo | ~2-5€ | Ore | Molto alta, elimina il main bang | Complessità firmware, nuovo punto di guasto |
| 6. TGC minimo | ~5-15€ | Giorni | Massima, risolve strutturalmente | Sproporzionato al rischio residuo attuale |

**Nota sull'Opzione 2 (la più probabile da usare):** dimezzare R7 da 10kΩ a ~5kΩ porta il guadagno da ×11 (+20,8dB) a ×6 (+15,6dB), una riduzione di -5,2dB **su entrambi** i segnali (main bang e target, proporzionalmente) — non cambia la differenza relativa di 26,1dB tra i due. Utile solo se il sintomo è clipping netto contro i binari di alimentazione, non se il problema è distinguere target da rumore.

**Nota sull'Opzione 1 da sola:** il gating nasconde il main bang nei dati salvati ma non impedisce che U2 sia stato in saturazione poco prima — se il tempo di recovery reale fosse più lungo del previsto, il target arriverebbe comunque "sporco" da un amplificatore non ancora lineare. Va sempre verificato guardando l'oscilloscopio in tempo reale, non fidandosi solo dei dati già campionati/tagliati.

**Combinazione consigliata se serve intervenire:**
```
Saturazione lieve/moderata:    Opzione 1 + Opzione 2 (economiche, veloci, reversibili)
Saturazione severa/prolungata: + Opzione 4 (diodi clamp) come secondo livello
Ultima risorsa:                Opzione 5 o 6, solo se le precedenti risultano insufficienti
```

### Protocollo di test mirato per Fase D — isolare saturazione e tempo di recovery
*Sostituisce/espande i passi 7-9 generici della Fase D sopra, con una sequenza pensata specificamente per rispondere sì/no alla domanda "U2 satura, e per quanto tempo?".*

**Setup oscilloscopio:**
```
Canale 1 → uscita di U2 (RX_OUT, prima dell'ADC)
Canale 2 → segnale di trigger del pulser (riferimento t=0)
Trigger oscilloscopio → sul fronte di Canale 2 (sincronizzato allo sparo)
Time/div → partire largo (~20µs/div), poi restringere (~500ns/div) sulla zona critica vicino a t=0
```

**Sequenza in 4 step, ognuno con criterio di successo/fallimento oggettivo (non a giudizio):**

1. **Baseline, senza target** (vasca vuota di target) — verifica il solo main bang. L'uscita di U2 tocca i binari di alimentazione? Se sì, saturazione confermata; misura quanto tempo passa prima che il segnale torni a un andamento pulito (tempo di recovery reale, sostituisce la stima del Passo 3)
2. **Target ad alta riflettività** (piastra metallica) — verifica se l'eco compare dopo che U2 è tornato pulito, o si sovrappone ancora al transitorio
3. **Target realistico** (dito, poi attraverso parete PP+gel come nel setup Step 2 finale) — verifica se l'eco debole vero (riflettività ~0,15%) emerge sopra il rumore residuo di recovery
4. **Variazione distanza** (3cm, 5cm, 7cm dal punto di sparo) — verifica empirica della tabella di margine del Passo 3

**Criteri di decisione:**
| Risultato osservato | Azione |
|---|---|
| Nessuna saturazione visibile | Nessuna modifica, procedi come pianificato |
| Saturazione breve, recovery pulito prima dell'eco target | Nessuna modifica |
| Recovery si sovrappone all'eco solo sotto i 5cm | Margine operativo più conservativo (mai sotto 5cm), zero modifiche hardware |
| Recovery si sovrappone anche a 5-7cm | Applica piano di contingenza, parti da Opzione 1+2 |

---

### Fase E — Passaggio al software (dati via microcontrollore)
10. Ora che sai (con l'oscilloscopio) che il segnale analogico è corretto, colleghi l'uscita dell'amplificatore all'ADC
11. Il firmware campiona e invia i dati al PC
12. Script Python riceve i dati grezzi e li plotta (ampiezza vs indice campione, poi convertito in tempo/profondità)

**Aspettati:** lo stesso identico pattern visto sull'oscilloscopio, ora come array di numeri e grafico software — impulso di trasmissione + eco a distanza coerente.

### Fase F — Validazione quantitativa
13. Ripeti il test spostando il target a 2-3 distanze note diverse (es. 5cm, 10cm, 15cm)
14. Verifichi che il tempo di arrivo dell'eco calcolato dal software cambi in modo coerente con la formula `distanza = velocità × tempo / 2`

**Aspettati:** errore contenuto (idealmente sotto qualche mm/percento) tra distanza misurata dal sistema e distanza reale misurata con un righello — questo è il tuo primo dato di validazione quantitativa, utilissimo da riportare nel portfolio.

---

## Risultato finale dello Step 1

Un sistema che, dato un singolo trasduttore fisso puntato verso un target in acqua a distanza nota:
- Genera un impulso di trasmissione pulito e ripetibile
- Riceve e amplifica correttamente l'eco di ritorno senza saturazione né rumore eccessivo
- Digitalizza il segnale e lo trasferisce al PC
- Produce un grafico A-mode (ampiezza vs profondità) leggibile
- Misura la distanza del target con un errore quantificato e accettabile rispetto al valore reale

Questo risultato, da solo, è già un traguardo dimostrabile e "vero" (di fatto hai costruito un misuratore di distanza a ultrasuoni professionale, non un giocattolo) — e soprattutto è la base validata su cui costruire lo Step 2 (rotazione + scan conversion → prima immagine B-mode).

---

## Checklist strumenti necessari prima di iniziare

- [x] Oscilloscopio ≥100MHz — **POSSEDUTO**: FNIRSI DPOX180H (180MHz, 500MSa/s, 2 canali)
- [ ] Multimetro
- [ ] Alimentatore da banco (regolabile, per alimentare pulser e sezione analogica separatamente)
- [x] Generatore di funzioni — **POSSEDUTO**: integrato nel FNIRSI DPOX180H (DDS, fino 20MHz sinusoidale, uscita fissa 1Vpp — vedi nota sotto per Fase C)
- [ ] Vasca/contenitore per acqua, non metallico (per evitare riflessioni indesiderate dalle pareti se non volute)
- [ ] Target di test a distanza nota e misurabile (piastra rigida, righello/calibro)
- [ ] Cavo coassiale schermato + connettori adeguati al trasduttore
- [ ] Ambiente di sviluppo RP2040 già configurato (VS Code + Pico SDK, o Arduino IDE + core RP2040)
- [ ] Python con NumPy/SciPy/Matplotlib installati sul PC

**Nota su Fase C con il generatore integrato FNIRSI:** l'uscita è fissa a 1Vpp, non regolabile — troppo alta per simulare un eco realistico (ordine dei mV). Serve un partitore resistivo/attenuatore passivo (vedi lista componenti, Blocco 2) tra l'uscita del generatore e l'ingresso dell'amplificatore RX per scalare il segnale a un livello realistico prima del test.

## Errori comuni da aspettarsi (per non scoraggiarti se capitano)

| Sintomo | Causa probabile |
|---|---|
| Nessun eco visibile, solo rumore | Guadagno RX troppo basso, bolle d'aria sul trasduttore, target fuori dal fascio |
| Segnale saturato/piatto in alto | Guadagno RX troppo alto, oppure T/R switch non isola bene il breakthrough del pulser |
| Eco a distanza sbagliata rispetto al reale | Errore nel calcolo t=0 (offset del trigger), velocità del suono usata nel calcolo non corretta per la temperatura dell'acqua |
| Rumore periodico ad alta frequenza sovrapposto al segnale | Massa/schermatura mal progettata, interferenza tra ramo HV e ramo analogico |
| Impulso di trasmissione "sporco"/con lungo ringing | Normale entro limiti, ma se troppo lungo mangia risoluzione assiale — verificare damping del trasduttore/pulser |

## Nota sul passaggio allo Step 2

Non modificare hardware per lo Step 2 finché lo Step 1 non è solido e ripetibile su più misure — aggiungere la rotazione meccanica su un sistema di sensing ancora incerto rende impossibile capire se un problema nell'immagine finale viene dalla meccanica o dall'elettronica di base.