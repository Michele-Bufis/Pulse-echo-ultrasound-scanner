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
