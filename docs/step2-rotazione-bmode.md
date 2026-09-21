# Step 2 — Rotazione meccanica + primo B-mode: hardware, assemblaggio, acquisti

*Nota: questo step non è ancora stato affrontato in dettaglio nelle sessioni di lavoro finora — il contenuto sotto resta il piano di riferimento, con solo le referenze allo Step 1 corrette per coerenza con i componenti effettivamente scelti (UC3843BN/MC33151PG/STP8NK100Z per pulser/boost, LM6172IN per RX, AD9280ARSZ come ADC, trasduttore 5MHz singolo).*

Obiettivo: prendere il sistema pulse-echo statico dello Step 1 (validato anche in simulazione LTspice) e aggiungere rotazione step-and-shoot del trasduttore, per la prima immagine 2D B-mode.

---

## 1. Schema concettuale

```
RP2040 --step/dir--> [Driver motore A4988] --> [Motore NEMA17] --> Riduttore/accoppiamento --> Anello rotante
                            ^                                              |
                            |                                    trasduttore montato sull'anello
                     [Encoder] <---------------------------------------(feedback posizione)
                            |
RP2040: per ogni step angolare -> trigger acquisizione pulse-echo (blocco Step 1) -> salva linea A-mode con angolo noto
                            |
                            v
PC (Python): scan conversion (coordinate polari -> immagine cartesiana) -> B-mode
```

**Step-and-shoot**, non rotazione continua — elimina la necessità di slip ring.

---

## 2. Componenti meccanici ed elettronici nuovi

| Blocco | Componente | Note |
|---|---|---|
| Motore | NEMA17 stepper, 1.8°/step | Con microstepping via A4988, risoluzione angolare più fine |
| Driver motore | A4988 (o TMC2208 per meno rumore) | Alimentazione motore separata da quella logica RP2040 |
| Encoder (opzionale) | AS5600 magnetico, o solo conteggio step | Per Step 2 si può partire senza, contando gli step inviati |
| Struttura anello | Stampa 3D + cuscinetto 608ZZ | Anche se non serve ora, valutare diametro pensando a eventuali estensioni future |
| Accoppiamento | Cinghia GT2 + pulegge | Più semplice e tollerante di accoppiamento rigido |
| Cavi verso trasduttore | Cavo coassiale flessibile con margine | Range di rotazione entro ±180° evita necessità di slip ring |

---

## 3. Assemblaggio meccanico — sequenza consigliata

1. Stampa 3D supporto anello (tolleranze da aggiustare, normale ristampare 1-2 volte)
2. Montaggio cuscinetto (press-fit, colla/carta vetrata per aggiustare)
3. Fissaggio motore su supporto separato, allineato per trasmissione a cinghia
4. Montaggio cinghia GT2, tensione moderata
5. Test meccanico puro (anello a mano, senza elettronica)
6. Collegamento motore-driver-RP2040 (alimentazione motore separata, mai dalla stessa uscita logica)
7. Test elettrico motore isolato prima di integrare con acquisizione pulse-echo

---

## 4. Firmware — aggiunte rispetto allo Step 1

1. Generazione impulsi STEP/DIR per A4988 (GPIO semplice, non serve PIO)
2. Sequenza step-and-shoot: step motore → settling time (decine di ms) → trigger pulse-echo (blocco Step 1 già validato) → salva linea con angolo → ripeti
3. Trasferimento dati: ogni pacchetto USB seriale include angolo + campioni

---

## 5. Software di ricostruzione

1. Ogni linea A-mode = vettore ampiezze vs tempo(=profondità), con angolo noto
2. **Scan conversion**: `x = profondità * cos(angolo)`, `y = profondità * sin(angolo)`
3. **Interpolazione**: `scipy.interpolate.griddata` per riempire i buchi tra linee discrete
4. **Envelope detection + compressione logaritmica**: applicata per linea prima della scan conversion
5. **Visualizzazione**: `matplotlib.pyplot.imshow`, Streamlit come miglioramento successivo

---

## 6. Lista acquisti Step 2 (in aggiunta allo Step 1)

| Cosa | Termine ricerca | Fascia prezzo | Fonte |
|---|---|---|---|
| Motore + driver kit | `NEMA17 stepper motor A4988 driver kit` | 8-15€ | AliExpress |
| Cinghia e pulegge GT2 | `GT2 belt pulley kit 3D printer` | 4-8€ | AliExpress |
| Cuscinetto 608ZZ ×2-3 | `608ZZ bearing` (o ferramenta locale) | 1-3€/pz | AliExpress/ferramenta |
| Filamento PLA (se mancante) | `PLA filament 1kg 1.75mm` | 12-18€/kg | AliExpress |
| Alimentatore 12V per motore | `12V 2A power supply adapter` | 5-10€ | AliExpress |
| Encoder AS5600 (opzionale) | `AS5600 magnetic encoder module` | 2-4€ | AliExpress |

**Totale indicativo Step 2: ~30-50€**, in aggiunta al totale Step 1 (~130-160€ già stimato).

---

## 7. Checklist prima di considerare completato lo Step 2

- [ ] Anello ruota liberamente senza attrito
- [ ] Motore risponde correttamente a STEP/DIR (test isolato)
- [ ] Sequenza step-and-shoot completa (rotazione + acquisizione + salvataggio con angolo)
- [ ] Prima immagine B-mode ricostruita in Python su target semplice
- [ ] Posizione ricostruita confrontata con misura reale a righello/calibro
