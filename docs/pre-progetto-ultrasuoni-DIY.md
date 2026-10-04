# Prima di iniziare: guida completa per il progetto ultrasuoni DIY

*Aggiornato per riflettere le scelte effettive del progetto (fonte TME, componenti specifici, strumentazione già posseduta).*

Obiettivo: contesto tecnico per affrontare lo Step 1 (singolo elemento, pulse-echo statico) senza sorprese.

---

## 1. Fisica minima indispensabile

- **Velocità del suono in acqua**: ~1480 m/s. Formula chiave: `profondità = (tempo di volo × velocità) / 2`.
- **Impedenza acustica e riflessione**: eco = interfaccia tra materiali con impedenza diversa (Z = densità × velocità). L'osso (alta Z) genera ombra acustica.
- **Attenuazione con la profondità**: proporzionale alla frequenza. Frequenze più alte = risoluzione migliore, penetrazione minore.
- **Risoluzione assiale vs laterale**: assiale dipende dalla durata impulso (frequenza trasduttore); laterale da dimensione/focalizzazione fascio.

---

## 2. Elettronica: sicurezza e concetti chiave

### 2.1 Alta tensione — il circuito reale lavora a ~140V
Nel progetto effettivo, il boost (UC3843BN + trasformatore custom su nucleo E20/10/6-3C94) si stabilizza intorno a **~142V** (validato in simulazione LTspice).
- I condensatori HV (C1, 47µF) restano carichi a circuito spento — **resistenza di bleeder (1MΩ, 1-2W)** in parallelo per scaricarli in sicurezza.
- Una mano sola vicino al circuito HV quando alimentato.
- Alimentatore ATX con limite di corrente per i primi test.
- Occhiali di protezione durante i test HV.

### 2.2 RF/segnali veloci
- **T/R switch**: diodi incrociati 1N4148 + resistenza limitatrice — **attenzione al wattaggio**: non serve per il calore (potenza media minima, duty cycle ~0.02%) ma per la tenuta dielettrica contro il picco di ~130V istantaneo. Usare resistori THT 1-2W (film metallico), non 1/4W standard.
- **Cavo coassiale 50Ω** tra trasduttore e front-end.

### 2.3 Strumentazione — già posseduta
**Oscilloscopio FNIRSI DPOX180H** (180MHz, 500MSa/s, 2 canali, con generatore di funzioni integrato DDS fino 20MHz) — già in possesso, nessun acquisto necessario. Nota pratica: il generatore integrato ha uscita fissa a 1Vpp, non regolabile — serve un partitore resistivo per simulare segnali realistici (mV) nei test del ramo RX.

**Alimentatore da banco**: soluzione scelta = riciclo alimentatore ATX da PC dismesso + breakout board + LM317 (più economico e veloce di un alimentatore da banco commerciale o di un modulo regolatore dedicato).

### 2.4 Dove si è risparmiato senza perdere valore didattico
- Boost HV autocostruito (UC3843BN + trasformatore avvolto a mano) invece di modulo pronto commerciale (risparmio ~150-200€)
- Filo di rame per il trasformatore: recuperato da un componente dismesso, non acquistato
- Zoccoli DIP8 invece di saldare direttamente i chip (protezione + riutilizzo)

---

## 3. Firmware e software — ordine di apprendimento

| Ordine | Cosa | Note |
|---|---|---|
| 1 | C/C++ base RP2040 | Logica non critica: comunicazione seriale, controllo, lettura ADC |
| 2 | RP2040 e PIO | Timing deterministico per trigger pulser/ADC — documentazione ufficiale RP2040, riferimento pic0rick |
| 3 | Comunicazione USB seriale | Trasferimento dati ADC → PC |
| 4 | Python/NumPy per DSP | Envelope detection, compressione logaritmica, filtri |
| 5 | Scan conversion (Step 2) | Trigonometria + interpolazione, coordinate polari → immagine 2D |

---

## 4. Validazione prima dell'hardware: simulazione LTspice

Passo aggiuntivo rispetto al piano originale: l'intero circuito Step 1 è stato impostato in simulazione LTspice prima dell'acquisto componenti. Risultati iniziali: boost stabile a ~142V, pulser con impulso netto 200ns, T/R switch che limita correttamente, trasduttore (modello Butterworth-Van Dyke) risonante a 5.03MHz (target 5MHz).

> ⚠️ **Limiti della validazione — aggiornato dopo revisione del 29/09/2026** (vedi `revisione-progetto-29-09-2026.md`): questi risultati **non sono una validazione completa** come affermato in precedenza. In particolare: (1) il loop di feedback del boost non era mai stato chiuso con un partitore dimensionato correttamente — il valore 142V osservato era conseguenza di un duty cycle fisso, non di regolazione reale; (2) il modello del trasduttore (Butterworth-Van Dyke) usa valori (C3=20pF, C4=20fF) probabilmente irrealistici, mai confrontati con una misura reale sul componente fisico; (3) un possibile bug sul guadagno RX (resistore R6 senza condensatore in serie) non è stato verificato con una simulazione `.op` reale. Dettagli completi, calcoli di correzione, e procedura di misura del trasduttore in `simulazione_ltspice_handoff.md` e `revisione-progetto-29-09-2026.md`.

**Calcolo aggiuntivo — campo vicino del trasduttore [C]:** `N = D²/(4λ) = (20mm)²/(4×0,296mm) ≈ 338mm` — con cristallo Ø20mm a 5MHz in acqua, l'intera vasca Step 2 (18cm) sta dentro il campo vicino: il fascio non è mai focalizzato, resta largo quanto il cristallo in ogni punto della vasca. Questo pone un limite reale alla risoluzione laterale raggiungibile (~20mm, non il millimetro) — dettagli in `step2-rotazione-bmode.md`, Sez. 4bis.

**Lezione chiave dal debug**: attenzione alla sintassi delle unità di misura in LTspice — `10 L` (con spazio) viene letto come 10 Henry, non 10µH. Usare sempre `10u` (numero+prefisso attaccato).

---

## 5. Checklist prima di ordinare

- [x] Oscilloscopio ≥100MHz — posseduto (FNIRSI DPOX180H)
- [ ] Ho capito la formula profondità = tempo di volo × velocità / 2
- [ ] So spiegare cosa fa un T/R switch e perché serve
- [ ] ~~Circuito validato in simulazione LTspice prima dell'acquisto~~ → **Circuito impostato in simulazione, ma non completamente validato** (vedi nota sopra — punti aperti su boost e RX da chiarire con `.op` reale)
- [ ] So come scaricare in sicurezza un condensatore HV (resistenza di bleeder)
- [x] Frequenza trasduttore decisa: **5MHz** (priorità su uso finale dito/polso, funziona bene anche su target Step 1)
- [ ] **Misurare C0 e curva di impedenza del trasduttore reale** appena arriva (procedura in `step1_pulse_echo_singolo_elemento.md`) — aggiunto in revisione 29/09/2026, necessario prima di fidarsi del modello BVD

Quando la checklist è completa, procedi con l'assemblaggio fisico secondo `step1_componenti_priorita_tme.md`.