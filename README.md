> [!NOTE]
> **Personal Project**
> 
> This repository contains an independent, self-directed learning initiative. It is not affiliated with any university coursework, academic research, or corporate entity.

# Sistema di Imaging a Ultrasuoni DIY — Pulse-Echo B-mode

Obiettivo: produrre un'immagine B-mode reale (non solo segnali A-mode) su fantocci tissue-mimicking e target biologici semplici (dito/polso in acqua), documentando l'intero processo ingegneristico per portfolio tecnico.

## Stato del progetto

**Step 1 (pulse-echo singolo elemento, A-mode statico): in fase avanzata, con punti aperti dopo revisione**
- ✅ Architettura definita e documentata
- 🔄 Circuito progettato e impostato in simulazione LTspice — **validazione solo parziale** (una revisione del 29/09/2026 ha trovato un partitore di feedback boost mai dimensionato per 142V e un possibile bug sul guadagno RX, non ancora verificati con una simulazione `.op` reale — vedi `revisione-progetto-29-09-2026.md`)
- 🔄 Lista componenti **congelata in attesa dei ricalcoli** — il totale ~84,70€ include valori ora superati (R2/R3, possibile rimozione LM317). Ordine consigliato: solo il trasduttore per ora (55,69€)
- 🔄 Assemblaggio fisico: non ancora iniziato

**Step 2 (rotazione meccanica → B-mode): design deciso, assemblaggio fisico non ancora iniziato**
- ✅ Configurazione vasca/sonda decisa: vasca cilindrica in Polipropilene, Ø18cm, sonda **esterna** (mai a contatto con l'acqua, gel accoppiante sulla parete) — motivo: trasduttore scelto non impermeabile. Dettagli e calcoli completi in `step2-rotazione-bmode.md`
- ✅ Numero acquisizioni per giro deciso: 200 (una per step motore NEMA17 full-step), verificato sufficiente per il raggio scelto
- 🔄 Componenti meccanici (motore, anello, driver): da ordinare
- 🔄 Assemblaggio fisico: non ancora iniziato

## Obiettivo e aspettative realistiche

Il sistema produce un'immagine **B-mode in scala di grigi**, con speckle (rumore a grana tipico degli ultrasuoni), contorni netti solo dove c'è forte differenza di impedenza acustica (pelle, osso), ombra acustica dietro l'osso. **Non è raggiungibile una qualità tipo RM o TAC** — tecnologie fisicamente diverse, irraggiungibili a livello hobbistico.

> ⚠️ **Aspettativa ulteriormente ridimensionata in revisione (29/09/2026)**: il trasduttore (Ø20mm, 5MHz) ha un campo vicino di ≈338mm — l'intera vasca (18cm) resta dentro questa zona, quindi il fascio non è mai focalizzato e la risoluzione laterale realistica è dell'ordine della **larghezza del fascio (~20mm)**, non del millimetro. Un dito (~15mm) risulta una macchia indistinta più che una forma dettagliata. Il riferimento "ecografia muscoloscheletrica clinica" (sonde 10-18MHz con array focalizzati elettronicamente) resta utile per spiegare l'obiettivo, ma la qualità raggiungibile da questo sistema a elemento singolo è più lontana da quel riferimento di quanto suggerito in precedenza. Dettagli in `step2-rotazione-bmode.md`, Sez. 4bis.

**Nota sulla configurazione reale:** a differenza del concetto iniziale (anello di trasduttori immerso), la sonda nello Step 2 resta **esterna** alla vasca (contact-type non impermeabile), a contatto con la parete esterna tramite gel — solo il target (dito/polso) è immerso. Vedi `step2-rotazione-bmode.md` per la cronologia della scelta.

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

⚠️ **Validazione solo parziale** (rivisto 29/09/2026 — vedi `revisione-progetto-29-09-2026.md`), non "completa" come affermato in precedenza:

| Blocco | Esito |
|---|---|
| Boost | 🔄 Duty fisso osservato a ~142V, **ma il loop di feedback reale non era mai stato chiuso correttamente** — ricalcolato (R2≈560kΩ, R3=10kΩ) |
| Pulser | 🔄 Impulso netto 200ns osservato, ma la topologia C2/D2/D3 non è stata riverificata in dettaglio |
| T/R switch | ✅ Limita da ±140V a ±0.6-0.7V — confermato, non impattato dalla revisione |
| Trasduttore (Butterworth-Van Dyke) | 🔄 Risonanza 5.03MHz osservata, ma un valore del modello (C4=20fF) è incoerente con L3=50µH per quella frequenza — origine della risonanza da chiarire con uno sweep `.ac` |
| RX | 🔄 **Sospetto bug non verificato**: un resistore (R6) senza condensatore in serie potrebbe causare saturazione dell'uscita — richiede un `.op` su `Eco.asc` |

Bug di sintassi unità di misura (`10 L` invece di `10u`) resta risolto correttamente. Dettagli completi, netlist aggiornata, e cronologia della revisione in `simulazione_ltspice_handoff.md` e `revisione-progetto-29-09-2026.md`.

## File nella repository

| File | Contenuto |
|---|---|
| `revisione-progetto-29-09-2026.md` | **Revisione tecnica approfondita** (metodo multi-prospettiva + peer review): registro problemi trovati, livelli di certezza [V]/[C]/[I]/[?], dimensionamenti ricalcolati, procedura di misura del trasduttore, ordine di lavoro concordato |
| `progetto_ultrasuoni_handoff.md` | Visione generale del progetto: obiettivo, architettura, roadmap, stack software, riferimenti (un0rick, Murgen) |
| `pre-progetto-ultrasuoni-DIY.md` | Contesto tecnico/fisico minimo prima di affrontare lo Step 1 (fisica base, sicurezza HV, checklist pre-acquisto) |
| `step1_pulse_echo_singolo_elemento.md` | Dettaglio Step 1: obiettivo, componenti, collegamenti, procedura di test passo-passo (Fasi A-F), analisi link budget e piano di contingenza saturazione RX, timing PIO (generazione clock ADC + cattura sincronizzata), procedura di misura del trasduttore, checklist strumenti, errori comuni |
| `step1_componenti_priorita_tme.md` | **Lista componenti Step 1** (ordine congelato in attesa dei ricalcoli di revisione), con codici precisi TME/AliExpress, stato acquisto (🟢/🟡/🔴), bilancio di potenza ATX (LM317 probabilmente non necessario) |
| `codici-distributore-verificati.md` | Codici TME verificati con stato stock, per i componenti Step 1 (valori aggiornati dopo revisione) |
| `lista-acquisti-completa.md` | Riepilogo acquisti dall'inizio alla fine del progetto (Step 1+2), con subtotali e budget complessivo — totale non più definitivo dopo revisione |
| `simulazione_ltspice_handoff.md` | Documentazione della simulazione: componenti, netlist per nodo, sostituzioni fatte, bug risolti, **punti aperti dopo revisione** (feedback boost, sospetto bug RX, modello BVD) |
| `step2-rotazione-bmode.md` | Dettaglio Step 2: configurazione vasca/sonda (cronologia scelte), materiale e dimensionamento vasca, calcolo acquisizioni/giro, anatomia sonda, costruzione morsetto, firmware, software di ricostruzione (formula scan conversion corretta, backprojection di inviluppo) |
| `step3-switching-matrix.md` | Switching matrix a 4 elementi — accantonato per budget, documentato come stretch goal futuro |
| `step4-fusione-multistatica.md` | Fusione multistatica (Delay-and-Sum) — accantonato per budget, dipende da Step 3; principio DAS ora applicato anche in Step 2 come backprojection a singolo elemento |

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

## Prossimi passi — AGGIORNATO dopo revisione del 29/09/2026

1. **Eseguire `.op` su `Eco.asc`** per chiarire il sospetto bug sul blocco RX (vedi `revisione-progetto-29-09-2026.md`)
2. **Ordinare per ora solo il trasduttore 5P20N** — il resto del carrello TME resta congelato
3. Alla consegna del trasduttore: misurare C0 e curva di impedenza reale (procedura in `step1_pulse_echo_singolo_elemento.md`)
4. Aggiornare `Eco.asc` con i valori misurati, chiudere il vero loop di feedback boost, ricostruire il modello BVD
5. Calcolare/avvolgere il trasformatore con i nuovi valori, progettare l'interfaccia U2→ADC, definire il clock PIO a 31,25MHz
6. **Solo a quel punto** confermare e finalizzare l'ordine TME aggiornato
7. Assemblaggio fisico del circuito Step 1
8. Setup firmware RP2040 (Pico SDK + PIO)
9. Test pratici secondo le fasi A-F documentate (incluso il protocollo di test mirato per saturazione RX)
10. Step 2: ordinare componenti meccanici e assemblare — design deciso, vedi `step2-rotazione-bmode.md`

---

*Progetto documentato con l'assistenza di Claude (Anthropic) e Gemini (Google) — inclusa la simulazione circuitale in LTspice e la selezione componenti.*
