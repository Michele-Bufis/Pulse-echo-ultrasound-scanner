# Sistema di Imaging a Ultrasuoni DIY — Pulse-Echo B-mode

Progetto hobbistico per costruire in casa un sistema di imaging a ultrasuoni pulse-echo, partendo da esperienza pregressa in ECG DIY. Obiettivo: produrre un'immagine B-mode reale (non solo segnali A-mode) su fantocci tissue-mimicking e target biologici semplici (dito/polso in acqua), documentando l'intero processo ingegneristico per portfolio tecnico.

## Stato del progetto

**Step 1 (pulse-echo singolo elemento, A-mode statico): in fase avanzata**
- ✅ Architettura definita e documentata
- ✅ Circuito completo progettato e **validato in simulazione LTspice**
- ✅ Lista componenti quasi completa — carrello TME confermato ~84.70€, totale realistico Step 1 ~130-160€
- 🔄 Assemblaggio fisico: non ancora iniziato

**Step 2 (rotazione meccanica → B-mode): pianificato, non ancora affrontato in dettaglio**

## Obiettivo e aspettative realistiche

Il sistema produce un'immagine **B-mode in scala di grigi**, con speckle (rumore a grana tipico degli ultrasuoni), contorni netti solo dove c'è forte differenza di impedenza acustica (pelle, osso), ombra acustica dietro l'osso. **Non è raggiungibile una qualità tipo RM o TAC** — tecnologie fisicamente diverse, irraggiungibili a livello hobbistico. Riferimento onesto: ecografia muscoloscheletrica clinica (polso/tendini).

**Riferimento concettuale** (non di scala): il principio di un anello di trasduttori in vasca d'acqua richiama lo stesso approccio del progetto **Midjourney Medical** (scanner full-body a ultrasuoni, migliaia di elementi, 2 petaflop di calcolo) — stessa famiglia tecnica, scala hobbistica enormemente più piccola. Buona narrativa di portfolio, non un target di qualità da inseguire.

## Architettura — Step 1

**Approccio A: singolo elemento trasduttore, pulse-echo statico** (in futuro: + rotazione meccanica per B-mode, Step 2). Un precedente approccio a 4 elementi rotanti con switching matrix multistatica (ispirato a Midjourney Medical) è stato valutato e accantonato per costi eccessivi — resta possibile come stretch goal futuro.

### Blocchi del circuito
1. **Boost HV** (12V → ~140V): UC3843BN (controller PWM, sostituito in sim da comparatore LT1011+rampa) + trasformatore custom (nucleo E20/10/6-3C94) + MOSFET STP8NK100Z
2. **Pulser**: MOSFET STP8NK100Z + capacitore storage WIMA FKP1 4.7nF, pilotato da MC33151PG
3. **T/R switch**: diodi incrociati 1N4148 + resistenza limitatrice, protegge lo stadio RX dal picco HV
4. **Trasduttore**: NDT multi-variante, 5MHz scelto (priorità su uso finale dito/polso)
5. **Amplificatore RX**: LM6172IN (op-amp THT, ~100MHz banda)
6. **Digitalizzazione**: AD9280ARSZ (ADC 8-bit, 32Msps) + Raspberry Pi Pico H (RP2040, PIO per timing preciso)

## Simulazione LTspice

Circuito completo validato in LTspice prima dell'assemblaggio fisico. Risultati:

| Blocco | Esito |
|---|---|
| Boost | ✅ Si stabilizza a ~142V (range target 100-300V) |
| Pulser | ✅ Impulso netto 200ns |
| T/R switch | ✅ Limita da ±140V a ±0.6-0.7V |
| Trasduttore (Butterworth-Van Dyke) | ✅ Risonanza 5.03MHz (target 5MHz, scarto 0.7%) |
| RX | ✅ Segnale amplificato coerente |

Bug principale trovato e risolto durante il debug: errore di sintassi nelle unità di misura (`10 L` invece di `10u` — letto come 10 Henry invece di 10µH, fattore di errore 1.000.000×). Dettagli completi, netlist, e cronologia del debug in `simulazione_ltspice_handoff.md`.

## File nella repository

| File | Contenuto |
|---|---|
| `progetto_ultrasuoni_handoff.md` | Visione generale del progetto: obiettivo, architettura, roadmap, stack software, riferimenti (un0rick, Murgen) |
| `step1_pulse_echo_singolo_elemento.md` | Dettaglio Step 1: obiettivo, componenti, collegamenti, procedura di test passo-passo (Fasi A-F), checklist strumenti, errori comuni |
| `step1_lista_componenti.md` | Prima versione della lista componenti (generica, propedeutica) |
| `step1_componenti_priorita_tme.md` | **Lista componenti definitiva**, con codici precisi TME/AliExpress, stato acquisto (🟢/🔴), organizzata per ordine di acquisizione |
| `simulazione_ltspice_handoff.md` | Documentazione completa della simulazione: componenti, netlist per nodo, sostituzioni fatte, bug risolti, risultati di validazione |

## Stack software (pianificato)

| Livello | Linguaggio |
|---|---|
| Timing critico (trigger pulser/ADC) | PIO Assembly (RP2040) |
| Logica firmware generale | C/C++ |
| DSP (envelope detection, filtri) | Python (NumPy/SciPy) |
| Ricostruzione immagine / scan conversion | Python (NumPy) |
| Visualizzazione | Python (Matplotlib/Streamlit) |

## Riferimenti open source

- **un0rick** (un0rick.cc) — hardware pulse-echo open source, RP2040, dev-kit pedagogico/NDT
- **Murgen** (hackaday.io/project/9281) — progetto simile, scansione meccanica con servo

## Strumentazione posseduta

Oscilloscopio FNIRSI DPOX180H (180MHz, 500MSa/s) con generatore di funzioni integrato, Raspberry Pi 4B (elaborazione), multimetro, saldatore.

## Prossimi passi

1. Chiudere gli ultimi acquisti 🔴 in `step1_componenti_priorita_tme.md`
2. Assemblaggio fisico del circuito Step 1
3. Setup firmware RP2040 (Pico SDK + PIO)
4. Test pratici secondo le fasi A-F documentate
5. Progettazione Step 2 (rotazione meccanica, motore+encoder)

---

*Progetto documentato con l'assistenza di Claude (Anthropic) e Gemini (Google) — inclusa la simulazione circuitale in LTspice e la selezione componenti.*