# Step 2 — Rotazione meccanica + primo B-mode: hardware, assemblaggio, acquisti

*Nota: contenuto aggiornato dopo la decisione finale sulla configurazione vasca/sonda (vedi Sezione 2 per la cronologia completa delle scelte). Riferimenti Step 1 coerenti con i componenti effettivamente scelti (UC3843BN/MC33151PG/STP8NK100Z per pulser/boost, LM6172IN per RX, AD9280ARSZ come ADC, trasduttore 5MHz singolo, contact-type YUSHI/XMSJ 5P20N).*

Obiettivo: prendere il sistema pulse-echo statico dello Step 1 (validato anche in simulazione LTspice) e aggiungere rotazione step-and-shoot del trasduttore, per la prima immagine 2D B-mode.

---

## 1. Schema concettuale

```
RP2040 --step/dir--> [Driver motore A4988] --> [Motore NEMA17] --> Riduttore/accoppiamento --> Anello rotante
                            ^                                              |
                            |                                    trasduttore montato sull'anello,
                            |                                    ESTERNO alla vasca (vedi Sez. 2)
                     [Encoder] <---------------------------------------(feedback posizione)
                            |
RP2040: per ogni step angolare -> trigger acquisizione pulse-echo (blocco Step 1) -> salva linea A-mode con angolo noto
                            |
                            v
PC (Python): scan conversion (coordinate polari -> immagine cartesiana) -> B-mode
```

**Step-and-shoot**, non rotazione continua — elimina la necessità di slip ring.

---

## 2. Configurazione vasca/sonda — cronologia delle scelte e motivazioni

### 2.1 Piano A (iniziale, scartato) — sonda e target entrambi immersi, a contatto
Il piano originale prevedeva il trasduttore montato sull'anello **immerso in acqua insieme al target** (dito/polso), a contatto diretto con la pelle durante la rotazione.

**Motivo di scarto: il trasduttore scelto (5P20N, YUSHI/XMSJ) non è impermeabile.** Verificato tramite:
- Analisi visiva del prodotto: ghiera zigrinata svitabile (tipica dei probe contact per sostituzione wear-face), connettore BNC fissato con viti visibili sul corpo (giunto meccanico, non annegato in resina) — entrambi indizi di costruzione non pensata per tenuta stagna
- Descrizione prodotto: esplicitamente per "superfici piane e lisce" e ispezione saldature — applicazione a contatto manuale con gel, mai in immersione
- Confronto con letteratura NDT professionale: i veri probe da immersione hanno housing in acciaio inox, connettore UHF (non BNC) e lente epossidica — fascia di prezzo 80-150€+, fuori budget per un dimostratore hobbistico

Un piano di ripiego (impermeabilizzazione fai-da-te con guaina termorestringente + sigillante) è stato considerato ma scartato per l'incertezza sulla tenuta reale nel tempo, specialmente sul giunto cristallo-corpo (non verificabile dall'esterno).

**Nota tecnica collaterale:** questo piano avrebbe comunque richiesto un meccanismo meccanico aggiuntivo — una slitta radiale a molla ("trombone slide") per compensare le variazioni di raggio locale del dito/polso (non un cerchio perfetto) mantenendo il contatto sonda-pelle sempre garantito. Complessità che il Piano B elimina come effetto collaterale.

### 2.2 Piano B (scelto) — sonda esterna alla vasca, mai a contatto con l'acqua
**Tecnica di riferimento:** stessa famiglia dei sensori ultrasonici "clamp-on" industriali (es. misuratori di flusso che si fissano dall'esterno di un tubo pieno di liquido, senza bucarlo).

**Configurazione:**
```
   Sonda (fissa su anello, gira attorno, esterna)
        │
        │  [gel accoppiante ecografico, stesso ruolo che avrebbe su pelle]
        ▼
   ┌─────────────────────────┐
   │  Parete esterna vasca    │  ← punto critico: materiale e spessore
   ├─────────────────────────┤
   │        ACQUA             │
   │      (dito/polso)         │
   └─────────────────────────┘
```

La sonda scorre sulla superficie esterna della vasca esattamente come farebbe su un paziente, con gel accoppiante tra faccia della sonda e parete. Il corpo/connettore BNC restano sempre completamente asciutti.

**Vantaggi ottenuti, risolti insieme:**
1. **Tenuta stagna:** nessun rischio di infiltrazione, il problema del trasduttore non impermeabile sparisce del tutto
2. **Meccanica semplificata:** il raggio di rotazione ora è fisso e costante (dettato dalla vasca cilindrica), non dal profilo irregolare del dito/polso — l'anello Step 2 originale a raggio fisso, senza slitte/molle, torna sufficiente
3. **Tolleranza al centraggio:** essendo il target (dito/polso) circondato da acqua e non a contatto diretto, piccole variazioni di posizione del target dentro la vasca non causano perdita di segnale brusca (l'acqua resta comunque il mezzo accoppiante, l'eco si affievolisce gradualmente con la distanza, non sparisce di colpo)

---

## 3. Materiale della vasca — analisi e scelta

### 3.1 Il problema fisico: impedenza acustica e spessore
Ogni interfaccia tra materiali con impedenza acustica diversa riflette parte dell'energia (Z = densità × velocità del suono nel materiale). A 5MHz la lunghezza d'onda in acqua è:
```
λ = v / f = 1480 m/s / 5.000.000 Hz ≈ 0,296mm (~0,3mm)
```
Più la parete è spessa rispetto a questa lunghezza d'onda, più si accumulano attenuazione e riflessioni multiple — quindi, a parità di materiale, **più sottile è meglio è**.

### 3.2 Confronto materiali (impedenza acustica Z, riferimento acqua ≈ 1,5 MRayl)
| Materiale | Z (MRayl) | Match con l'acqua | Reperibilità |
|---|---|---|---|
| Pellicola alimentare (LDPE) | ~1,8-2,0 | Ottimo | Supermercato — ma NON rigida, scartata per requisito di robustezza (Sez. 3.3) |
| Sacchetto freezer (LDPE/HDPE) | ~1,8-2,3 | Ottimo | Supermercato — stesso limite di robustezza |
| **Polipropilene (PP)** | ~1,9-2,4 | **Molto buono, il migliore tra i materiali rigidi** | Contenitori alimentari, vaschette gelato, barattoli dispensa |
| HDPE (bottiglie) | ~1,8-2,3 | Buono | Casa |
| PET (bottiglie acqua) | ~3,6 | Discreto (riflette ~41% a singola interfaccia, ma spessore ridotto aiuta) | Casa |
| Acrilico/plexiglass standard (3-5mm) | ~3,2 | Scarso se spesso | Ferramenta |
| Vetro | ~13-14 | Pessimo, da evitare | — |

*(Dato di verifica: letteratura scientifica conferma che il polipropilene ha impedenza acustica molto vicina a quella dell'acqua — motivo per cui è usato anche in ambito di ricerca per contenitori campione esposti a ultrasuoni.)*

### 3.3 Requisito aggiuntivo: robustezza meccanica
Le opzioni a film sottile (pellicola, sacchetto) sono acusticamente ottimali ma **non reggono da sole il peso dell'acqua né la pressione della sonda che scorre ripetutamente sopra** — si deformerebbero o si romperebbero, causando perdita di contatto costante.

### 3.4 Decisione finale: contenitore rigido in Polipropilene (PP)
Non un film teso su un telaio, ma un **contenitore già pronto, rigido di suo** — es. vaschette da gelato vuote, contenitori alimentari trasparenti da asporto, barattoli da dispensa in PP. Compromesso ottimale tra:
- Impedenza acustica quasi ideale (il migliore tra i materiali rigidi comuni)
- Rigidità sufficiente a reggere acqua + sonda in movimento
- Reperibilità immediata, costo nullo o quasi (riciclo/riuso)

**Compromesso accettato:** pareti più spesse (0,5-1mm) rispetto all'ideale teorico (pellicola, 0,01mm) → leggera perdita di segnale aggiuntiva, verificabile direttamente in Fase D del test Step 1 (ampiezza eco su oscilloscopio). Se insufficiente, possibile ottimizzazione futura: assottigliare localmente la parete (carta vetrata fine) solo nella fascia dove scorre la sonda — da valutare solo se necessario, non pianificato a priori.

---

## 4. Dimensionamento della vasca — diametro 18cm

### 4.1 Fattori da bilanciare
Il raggio non è "più piccolo possibile" né "più grande possibile" — è un compromesso tra vincoli opposti:

**Motivi per non andare troppo piccolo:**
1. **Zona morta (dead zone):** subito dopo lo sparo il ramo RX è ancora disturbato dal breakthrough del pulser (residuo ±0.6-0.7V anche dopo il T/R switch, da validazione LTspice) — serve margine perché l'eco utile non si sovrapponga a questo transitorio
2. **Separazione dal riverbero della parete opposta:** con vasca troppo stretta, l'eco che rimbalza sulla parete dall'altra parte del target arriva quasi nello stesso istante dell'eco del target stesso — impossibile distinguerli nel software. Con margine sufficiente, l'eco di parete lontana arriva chiaramente più tardi e può essere tagliato via con una finestra temporale (gating) software
3. **Tolleranza al centraggio:** margine per un posizionamento non perfetto del target a mano

**Motivi per non andare troppo grande:**
1. **Nessun vantaggio reale:** l'attenuazione dell'acqua a 5MHz è trascurabile su queste distanze (frazioni di dB su pochi cm) — allargare oltre il necessario non guadagna nulla in qualità segnale
2. **Tempo di volo più lungo:** solo un dettaglio firmware (finestra di campionamento più ampia), ma inutile allungarlo senza motivo
3. **Ingombro meccanico:** anello più grande, più materiale, più spazio fisico

### 4.2 Calcolo
```
Raggio target più grande (polso, ~5-6cm diametro) ≈ 3cm
+ margine di sicurezza (zona morta + separazione riverbero parete) ≈ 4-5cm
= Raggio vasca ≈ 7,5-9cm
```

### 4.3 Decisione finale: Diametro 18cm (raggio 9cm)
Scelto il lato alto del range calcolato (15-18cm), per maggiore margine di sicurezza contro ambiguità di segnale al primo tentativo pratico — più facile ignorare in software un eco di parete "in ritardo comodo" che diagnosticare due echi sovrapposti per vasca troppo stretta.

---

## 5. Numero di acquisizioni per giro — 200 (una per ogni step motore)

### 5.1 Perché il raggio maggiore richiede più acquisizioni
Con la sonda che ruota sulla parete esterna della vasca, il raggio di rotazione rilevante ai fini del campionamento angolare è quello della vasca (9cm). La distanza fisica coperta tra un'acquisizione e la successiva è:
```
Distanza tra acquisizioni = Raggio × Angolo di step (in radianti)
```
A parità di step angolare, un raggio maggiore lascia "buchi" di copertura più grandi tra un'acquisizione e l'altra — serve infittire gli step.

### 5.2 Calcolo del minimo necessario
Riferimento: dimensione del fascio della sonda. Cristallo 20mm, nel campo vicino il fascio resta circa collimato a questa larghezza. Per evitare buchi di copertura, spaziatura massima consigliata ≈ metà larghezza fascio ≈ **10mm** tra acquisizioni.
```
Circonferenza vasca (raggio 9cm) = 2π × 90mm ≈ 565mm
N acquisizioni minime = 565mm / 10mm ≈ 57
```

### 5.3 Verifica: il motore già previsto è sufficiente
NEMA17 full-step (senza microstepping) = 1.8°/step = 200 step/giro completo.
```
Spaziatura reale con 200 step = 2π × 90mm / 200 ≈ 2,8mm tra acquisizioni
```
**2,8mm è ben sotto il minimo richiesto (10mm)** — nessuna modifica hardware necessaria rispetto al piano originale, il NEMA17 con guida A4988 già in lista acquisti basta.

### 5.4 Decisione finale: 200 acquisizioni/giro
Un'acquisizione per ogni step motore (opzione più semplice in firmware: trigger a ogni step, nessuna logica di conteggio aggiuntiva). Alternativa scartata: sotto-campionare a ~65-70 acquisizioni (1 ogni 3 step) per ridurre tempo/volume dati — margine di sicurezza inutile da gestire ora, essendo pulse-echo comunque veloce (microsecondi per sparo); da valutare eventualmente in futuro se serve velocizzare la scansione.

---

## 6. Limite fisico accettato — niente riflessioni deviate/specular

Un sistema pulse-echo a singolo elemento (Step 1+2) può ricevere **solo** l'eco che torna direttamente verso la sonda lungo il percorso di andata. Se un'onda colpisce una superficie angolata e si riflette specularmente in un'altra direzione, quell'energia non arriva mai al cristallo — non è un filtro software, è un limite fisico strutturale: il trasduttore può convertire in segnale elettrico solo ciò che fisicamente lo colpisce.

**Non è un difetto del piano:** è il limite intrinseco di qualsiasi sistema monostatico a singolo elemento, già documentato da inizio progetto (vedi `progetto_ultrasuoni_handoff.md` — superfici lisce/specular richiedono angolazione quasi perpendicolare per essere rilevate, i tessuti biologici "scatteranti" sono meno sensibili a questo problema). È lo stesso principio su cui si basa il progetto di riferimento un0rick, ed è come funzionavano i primi ecografi meccanici a scansione singola.

**Soluzione esistente ma accantonata:** catturare anche le riflessioni deviate richiederebbe più ricevitori indipendenti in posizioni diverse — esattamente lo Step 3-4 (switching matrix multistatica + algoritmo Delay-and-Sum), già valutato e accantonato per budget a inizio progetto. La scelta di procedere solo con Step 1-2 conferma implicitamente che il compromesso costo/complessità di Step 3-4 resta corretto.

---

## 7. Componenti meccanici ed elettronici nuovi

| Blocco | Componente | Note |
|---|---|---|
| Motore | NEMA17 stepper, 1.8°/step (200 step/giro) | Con A4988, sufficiente per 200 acquisizioni/giro senza microstepping (vedi Sez. 5.3) |
| Driver motore | A4988 (o TMC2208 per meno rumore) | Alimentazione motore separata da quella logica RP2040 |
| Encoder (opzionale) | AS5600 magnetico, o solo conteggio step | Per Step 2 si può partire senza, contando gli step inviati |
| Vasca | Contenitore rigido in Polipropilene (PP), Ø18cm | Vaschetta gelato/contenitore alimentare — vedi Sez. 3.4 |
| Gel accoppiante | Gel ecografico o equivalente | Tra faccia sonda e parete esterna vasca |
| Struttura anello | Stampa 3D + cuscinetto 608ZZ | Raggio dimensionato su Ø18cm vasca (Sez. 4.3) |
| Accoppiamento | Cinghia GT2 + pulegge | Più semplice e tollerante di accoppiamento rigido |
| Cavi verso trasduttore | Cavo coassiale flessibile con margine | Range di rotazione entro ±180° evita necessità di slip ring |

---

## 8. Assemblaggio meccanico — sequenza consigliata

1. Reperire/verificare il contenitore PP scelto (Sez. 3.4) — testare tenuta d'acqua riempiendolo prima di montare qualunque elettronica
2. Stampa 3D supporto anello, dimensionato per raggio 9cm (vasca Ø18cm) (tolleranze da aggiustare, normale ristampare 1-2 volte)
3. Montaggio cuscinetto (press-fit, colla/carta vetrata per aggiustare)
4. Fissaggio motore su supporto separato, allineato per trasmissione a cinghia
5. Montaggio cinghia GT2, tensione moderata
6. Montaggio del supporto sonda sull'anello, verificando che la faccia della sonda resti sempre a contatto con la parete esterna della vasca durante tutta la rotazione
7. Test meccanico puro (anello a mano, senza elettronica)
8. Collegamento motore-driver-RP2040 (alimentazione motore separata, mai dalla stessa uscita logica)
9. Test elettrico motore isolato prima di integrare con acquisizione pulse-echo

---

## 9. Firmware — aggiunte rispetto allo Step 1

1. Generazione impulsi STEP/DIR per A4988 (GPIO semplice, non serve PIO)
2. Sequenza step-and-shoot: step motore → settling time (decine di ms) → trigger pulse-echo (blocco Step 1 già validato) → salva linea con angolo → ripeti, per 200 acquisizioni/giro (Sez. 5.4)
3. Finestra di campionamento dimensionata sul tempo di volo per raggio 9cm — da verificare/estendere rispetto al piano originale (raggio maggiore = tempo di volo maggiore)
4. Trasferimento dati: ogni pacchetto USB seriale include angolo + campioni

---

## 10. Software di ricostruzione

1. Ogni linea A-mode = vettore ampiezze vs tempo(=profondità), con angolo noto
2. **Scan conversion**: `x = profondità * cos(angolo)`, `y = profondità * sin(angolo)`
3. **Interpolazione**: `scipy.interpolate.griddata` per riempire i buchi tra linee discrete
4. **Envelope detection + compressione logaritmica**: applicata per linea prima della scan conversion
5. **Gating temporale**: finestra per escludere il riverbero della parete opposta della vasca (vedi Sez. 4.1, punto 2) dal segnale utile
6. **Visualizzazione**: `matplotlib.pyplot.imshow`, Streamlit come miglioramento successivo

---

## 11. Lista acquisti Step 2 (in aggiunta allo Step 1)

| Cosa | Termine ricerca | Fascia prezzo | Fonte |
|---|---|---|---|
| Motore + driver kit | `NEMA17 stepper motor A4988 driver kit` | 8-15€ | AliExpress |
| Cinghia e pulegge GT2 | `GT2 belt pulley kit 3D printer` | 4-8€ | AliExpress |
| Cuscinetto 608ZZ ×2-3 | `608ZZ bearing` (o ferramenta locale) | 1-3€/pz | AliExpress/ferramenta |
| Filamento PLA (se mancante) | `PLA filament 1kg 1.75mm` | 12-18€/kg | AliExpress |
| Alimentatore 12V per motore | `12V 2A power supply adapter` | 5-10€ | AliExpress |
| Encoder AS5600 (opzionale) | `AS5600 magnetic encoder module` | 2-4€ | AliExpress |
| Vasca in PP, Ø18cm | Riciclo (vaschetta gelato) o `contenitore alimentare PP trasparente` | 0-5€ | Casa/supermercato |
| Gel accoppiante ecografico | `ultrasound gel transmission` | 5-10€ | Amazon/farmacia |

**Totale indicativo Step 2: ~30-55€**, in aggiunta al totale Step 1 (~130-160€ già stimato).

---

## 12. Checklist prima di considerare completato lo Step 2

- [ ] Vasca PP Ø18cm reperita, tenuta d'acqua verificata
- [ ] Anello (raggio 9cm) ruota liberamente senza attrito, sonda a contatto costante con parete esterna vasca durante tutta la rotazione
- [ ] Motore risponde correttamente a STEP/DIR (test isolato)
- [ ] Sequenza step-and-shoot completa: 200 acquisizioni/giro, rotazione + acquisizione + salvataggio con angolo
- [ ] Finestra di campionamento estesa correttamente per il tempo di volo su raggio 9cm
- [ ] Prima immagine B-mode ricostruita in Python su target semplice, con gating per escludere riverbero parete opposta
- [ ] Posizione ricostruita confrontata con misura reale a righello/calibro