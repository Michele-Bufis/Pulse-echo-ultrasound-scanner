# Progetto: Sistema di imaging a ultrasuoni (pulse-echo, B-mode) — Handoff

## Obiettivo
Costruire in casa un sistema di imaging a ultrasuoni che produca un'**immagine 2D reale** (B-mode, non solo segnali A-mode), partendo da esperienza pregressa in ECG DIY. Obiettivo finale: imaging su fantocci tissue-mimicking e su target biologici semplici (dito/polso in acqua), per portfolio tecnico da mostrare ad aziende del settore biomedicale.

**Chiarimento importante — completezza vs qualità:** con rotazione a 360° (non un ventaglio parziale) lo Step 2 produce concettualmente una **"fetta" 2D completa dell'oggetto**, strutturalmente paragonabile a una sezione TAC/RM (vista completa a giro d'orizzonte, non solo un lato parziale come un'ecografia clinica classica). Quello che resta diverso da TAC/RM è il **dettaglio interno**: contorni netti solo dove c'è forte differenza di impedenza acustica (pelle, osso), zone di tessuto omogeneo restano "grigie e granulose" (speckle), ombre acustiche dietro l'osso restano buchi nella fetta. La rotazione risolve la *forma* del risultato (sezione completa), non la *ricchezza di dettaglio* al suo interno.

**Riferimento concettuale (non di scala):** l'idea originale di un anello di trasduttori rotante in vasca d'acqua è concettualmente la stessa famiglia tecnica del progetto **Midjourney Medical** (annunciato giugno 2026, scanner full-body a ultrasuoni con anello di migliaia di trasduttori in vasca d'acqua, sviluppato con Butterfly Network, ricostruzione con 2 petaflop di calcolo) — stesso principio (multistatico, imaging in vasca, tecniche derivate da time-reversal/synthetic aperture), scala enormemente più piccola. Da citare in portfolio come narrativa ("prototipo hobbistico che dimostra lo stesso principio, a scala ridotta"), non da inseguire come target di qualità immagine.

Nessun vincolo di sicurezza clinica: il dispositivo è un dimostratore, non verrà mai usato su altre persone in modo clinico, quindi non serve la trafila di certificazione medicale — resta comunque da trattare con normale attenzione da elettronica RF/HV.

## Architettura scelta — AGGIORNATA
**Approccio A — Singolo elemento + rotazione meccanica** (tornato da Approccio C per motivi di budget: lo switching matrix multistatico a 4 elementi risultava troppo costoso)

L'idea a 4 elementi rotanti (Approccio C, esplorata e poi accantonata) resta valida come possibile "stretch goal" futuro se il budget lo permetterà, con narrativa esplicita di richiamo al principio Midjourney Medical — ma non è nel piano attuale.

## Roadmap incrementale consigliata (NON partire dal sistema completo)
1. **Step 1** (in corso, lista componenti definita): 1 solo elemento, pulse-echo statico → valida hardware/firmware base (A-mode)
2. **Step 2** (prossimo): 1 elemento + rotazione meccanica → B-mode a scansione meccanica pura (scan conversion semplice) — **questo è ora l'obiettivo finale del piano attuale**
3. ~~Step 3-4 (4 elementi + switching matrix + eventuale fusione multistatica)~~ — accantonati per budget, possibile stretch goal futuro

## Blocchi hardware necessari (Approccio A — singolo elemento)
- **Trasduttore piezoelettrico** (1×, singolo elemento — scelto 5MHz, priorità data all'uso finale su dito/polso, funziona bene anche per i test su fantoccio/target in acqua dello Step 1)
- **Pulser** (generatore impulsi HV, 100-300V, breve durata) — MOSFET HV (STP8NK100Z) + gate driver (MC33151PG), capacitore storage (WIMA FKP1)
- **Boost HV autocostruito** (12V→100-300V, troppo caro il modulo pronto XP Power ~150-200€): UC3843BN + trasformatore custom (nucleo E20/10/6-3C94 + coil former EF20-K-H-8P, filo di rame recuperato) + MOSFET dedicato boost (basse tensioni, ancora da scegliere) + capacitore filtro elettrolitico HV (ancora da scegliere) + resistenza current sense (ancora da scegliere)
- **T/R switch** (diodi incrociati 1N4148 + resistenza limitatrice, ancora da scegliere il valore)
- ~~Switching matrix~~ — non più necessaria (approccio a singolo elemento)
- **Amplificatore RX**: LM6172IN (THT, DIP-8, ~100MHz banda passante, nessun TGC nello Step 1, guadagno fisso)
- **ADC**: AD9280ARSZ (8-bit, 32Msps, stesso chip usato da un0rick/pic0rick)
- **Microcontrollore**: Raspberry Pi Pico H (RP2040) — PIO state machine per timing preciso
- **Motore + driver + encoder** per rotazione (da definire nello Step 2, non ancora nella lista Step 1)
- Struttura meccanica di rotazione: da definire nello Step 2

## Budget stimato — AGGIORNATO (dato reale da carrello TME confermato + acquisti rimanenti)
Lista dettagliata con codici precisi in file separato (checklist componenti Step 1). **Carrello TME confermato: ~70,75€.** A questo si aggiungono i pezzi ancora da ordinare (trasduttore ~50€ su AliExpress, resistori/capacitori passivi vari, breakout ATX+LM317+dissipatore, adattatore ADC, vasca/target/calibro) — **totale realistico Step 1: ~130-160€**, comunque molto inferiore alla stima iniziale per l'Approccio C (120-320€), grazie a: niente switching matrix, boost HV autocostruito invece di modulo pronto XP Power, trasformatore con filo recuperato invece di acquistato, molti materiali di supporto già posseduti.
- Oscilloscopio: **già posseduto** (FNIRSI DPOX180H, 180MHz, 500MSa/s, con generatore di funzioni integrato)

## Progetti open source di riferimento (fondamentali da studiare prima di progettare da zero)
- **un0rick** (un0rick.cc) — hardware pulse-echo open source, scheda attuale "pic0rick" su RP2040, 60 Msps 10-bit. Dichiaratamente un dev-kit pedagogico/NDT, non un dispositivo medicale. Supporta A-mode e imaging con scansione meccanica.
  - Paper di riferimento: "Arduino-like development kit for single-element ultrasound imaging" (Journal of Open Hardware, 2016)
- **Murgen** (hackaday.io/project/9281) — progetto simile, log di sviluppo dettagliati, modulo di elaborazione analogica acquistabile su Tindie, usa piezo mosso da servo per scanning, visualizzazione con server Streamlit

Strategia: NON copiare, ma studiare la loro architettura (in particolare soluzione T/R switch, pulser, TGC) e riadattarla con scelte componenti proprie, documentando le differenze — approccio più credibile e maturo per un portfolio.

## Stack software

| Livello | Linguaggio | Note |
|---|---|---|
| Timing critico (trigger pulser/ADC) | **PIO Assembly** (RP2040) | Solo poche righe, timing hardware deterministico a livello di ciclo di clock |
| Logica firmware generale (switching, motore, comunicazione) | **C/C++** | 99% del codice firmware reale |
| DSP (envelope detection, compressione log, TGC software, filtri) | **Python** (NumPy/SciPy) | Su PC, dopo trasferimento dati |
| Ricostruzione immagine (scan conversion, geometria, eventuale fusione multistatica) | **Python** (NumPy) | Parte più complessa concettualmente — trigonometria + interpolazione |
| Visualizzazione/demo | **Python** (Matplotlib/Streamlit) | Streamlit usato anche da Murgen |

Nota: AVR (Arduino classico) scartato — RP2040 scelto per le PIO state machine, clock più alto, compatibilità con un0rick/Murgen.

## Ambiente di sviluppo / toolchain
- **Consigliato**: VS Code + estensione ufficiale "Raspberry Pi Pico" (installa Pico SDK, CMake, compilatore ARM GCC in automatico) — accesso nativo a PIO
- Alternativa più semplice ma meno potente: Arduino IDE + core RP2040 (Earle Philhower o ufficiale)
- **Flash**: tenere BOOTSEL premuto collegando USB → Pico appare come drive di massa → si copia/carica file `.uf2` → riavvio automatico
- **Debug hardware** (opzionale ma utile): secondo Pico configurato come debug probe via SWD (~10-15€), o Raspberry Pi Debug Probe ufficiale

## Concetti fisici chiave da tenere a mente in fase di progettazione
- **Riflessione vs scattering**: superfici lisce (specular) richiedono angolazione quasi perpendicola per essere rilevate — da qui l'importanza della copertura angolare (rotazione). Tessuti biologici/materiali "scatteranti" sono meno sensibili a questo problema.
- **Ombra acustica**: interfacce ad altissima riflettività (es. osso) bloccano quasi tutta l'energia — nulla visibile dietro, è un limite fisico non risolvibile con più hardware.
- **Risoluzione assiale**: dipende da durata impulso (frequenza trasduttore) — non dalla copertura angolare.
- **Risoluzione laterale**: dipende da dimensione elemento/focalizzazione del fascio.
- Target di test consigliati: fantocci tissue-mimicking (gelatina + cellulosa, con inclusioni per generare echi interni) e successivamente dito/polso in acqua (buona eterogeneità: pelle, tessuti molli, tendini, osso con ombra acustica — risultato visivamente riconoscibile e paragonabile a ecografia muscoloscheletrica reale). **Trasduttore scelto: 5MHz** (priorità data all'uso finale su dito/polso; funziona bene anche sui target Step 1 a distanze brevi 5-15cm).

## Prossimi passi pratici da affrontare nella prossima sessione
1. Chiudere gli ultimi elementi 🔴 rimasti della checklist componenti Step 1 (vedi file separato: MOSFET boost, capacitore filtro boost, resistenza current sense, resistenza T/R switch, connettore BNC femmina, cavo USB Pico, ordine trasduttore su AliExpress, vasca/piastra target/calibro)
2. Recuperare filo di rame da un componente dismesso per l'avvolgimento del trasformatore
3. Assemblaggio fisico e collegamenti del circuito Step 1 (pulser, T/R switch, RX, ADC)
4. Setup ambiente di sviluppo RP2040 (VS Code + Pico SDK) — non ancora iniziato
5. Test pratici Step 1 (Fasi A-F già definite in dettaglio in documento separato)
6. Solo dopo Step 1 completo e validato: passare allo Step 2 (motore + encoder per rotazione, ancora da progettare/scegliere componenti)
