# Lista acquisti completa — dall'inizio alla fine del progetto

*Riscritta per riflettere le scelte effettive: fonte principale TME (non AliExpress/distributori generici), componenti realmente scelti e validati in simulazione LTspice, strumentazione già posseduta.*

---

## STRUMENTAZIONE — già posseduta, nessun acquisto

Oscilloscopio FNIRSI DPOX180H (180MHz, 500MSa/s, 2 canali, generatore di funzioni integrato DDS 20MHz), sonde ×2, PC per elaborazione Python, saldatore+stagno+flussante, multimetro, breadboard, PCB proto, kit cavi dupont, cavo coassiale BNC-BNC, filo rigido per collegamenti.

**Risparmio rispetto a un progetto da zero: ~185-235€ non spesi.**

---

## STEP 1 — Componenti (dettaglio completo con codici in `step1_componenti_priorita_tme.md` e `codici-distributore-verificati.md`)

**Fonte principale: TME.eu** (sede italiana, Grassobbio BG — spedizione nazionale, niente dazi). Eccezione: trasduttore da AliExpress.

| Blocco | Voci principali | Subtotale |
|---|---|---|
| Digitale + alimentazione | Raspberry Pi Pico H, zoccoli DIP8 ×5, ATX breakout 24pin (AliExpress, confermato), LM317, dissipatore TO-220 (AliExpress, confermato), bleeder resistor, occhiali protezione | ~15-20€ |
| Pulser + Boost HV | MC33151PG, STP8NK100Z ×2, WIMA FKP1 ×2, UF4007 ×3, UC3843BN, nucleo E20/10/6-3C94, coil former ×2, capacitore 47µF 400V, resistori vari | ~25-30€ |
| Ricezione + trasduttore | Trasduttore NDT 5MHz 20mm YUSHI/XMSJ 5P20N (AliExpress, confermato, 55,69€), 1N4148 ×2, R4 1-2W, LM6172IN, cavo BNC extra, connettore BNC femmina, resistori/capacitori RX (incl. C6) | ~75-85€ |
| Digitalizzazione + test | AD9280ARSZ, adattatore SSOP28→DIP28 (ancora da trovare), vasca, piastra target, calibro | ~35-40€ |

**Totale Step 1: ~130-160€** — ⚠️ **non più definitivo dopo la revisione del 29/09/2026** (vedi `revisione-progetto-29-09-2026.md`): il carrello TME ~84,70€ include R2/R3 con valori ora superati (ricalcolati per 142V reali), possibile rimozione LM317+dissipatore, 4 componenti UC3843 mancanti da aggiungere, quantità nucleo trasformatore da verificare. Dettagli completi in `step1_componenti_priorita_tme.md`.

---

## STEP 2 — Rotazione meccanica + primo B-mode (design deciso, dettagli completi in `step2-rotazione-bmode.md`)

Configurazione finale: sonda esterna alla vasca (mai a contatto con l'acqua, gel accoppiante sulla parete), vasca cilindrica in Polipropilene Ø18cm, 200 acquisizioni/giro.

| Cosa | Termine ricerca | Prezzo | Fonte |
|---|---|---|---|
| Motore + driver kit (NEMA17+A4988) | `NEMA17 stepper motor A4988 driver kit` | 8-15€ | AliExpress |
| Cinghia e pulegge GT2 | `GT2 belt pulley kit 3D printer` | 4-8€ | AliExpress |
| Cuscinetto 608ZZ ×2-3 | `608ZZ bearing` | 1-3€/pz | AliExpress/ferramenta |
| Filamento PLA (se mancante) | `PLA filament 1kg 1.75mm` | 12-18€/kg | AliExpress |
| Alimentatore 12V motore | `12V 2A power supply adapter` | 5-10€ | AliExpress |
| Encoder AS5600 (opzionale) | `AS5600 magnetic encoder module` | 2-4€ | AliExpress |
| Vasca in PP, Ø18cm | Riciclo (vaschetta gelato) o `contenitore alimentare PP trasparente` | 0-5€ | Casa/supermercato |
| Gel accoppiante ecografico | `ultrasound gel transmission` | 5-10€ | Amazon/farmacia |

**Subtotale Step 2: ~30-55€**

---

## STEP 3-4 — ACCANTONATI (switching matrix 4 elementi + fusione multistatica)

Valutati e messi da parte per budget durante la pianificazione (modulo boost HV professionale da solo costava 150-200€). Restano documentati come stretch goal futuro in `step3-switching-matrix.md` e `step4-fusione-multistatica.md` — se ripresi, costo indicativo aggiornato ~180-220€ (i 3 trasduttori aggiuntivi costerebbero ~50€/pz al prezzo reale scelto, non i pochi euro dei dischi grezzi ipotizzati inizialmente).

---

## Totale complessivo realistico (progetto attuale, Step 1+2)

| Voce | Costo |
|---|---|
| Strumentazione | 0€ (già posseduta) |
| Step 1 | 130-160€ |
| Step 2 | 30-55€ |
| **Totale attuale (Step 1+2)** | **~160-215€** |
| Step 3-4 (se ripresi in futuro) | +180-220€ |

---

## Ordine di acquisto consigliato — AGGIORNATO dopo revisione 29/09/2026

> ⚠️ Il circuito Step 1 **non è ancora pienamente validato** come affermato in precedenza — restano punti aperti su boost/RX da chiarire con una simulazione `.op`/`.ac` reale su `Eco.asc`. Vedi `revisione-progetto-29-09-2026.md` per il dettaglio completo.

1. **Eseguire `.op` su `Eco.asc`** per chiarire il sospetto bug sul blocco RX (R6 senza condensatore in serie)
2. **Ordinare per ora solo il trasduttore 5P20N** (55,69€) — il resto del carrello TME resta congelato, non finalizzare l'ordine
3. Alla consegna del trasduttore: misurare capacità C0 e curva di impedenza reale, poi test eco su piastra attraverso parete PP+gel
4. Aggiornare `Eco.asc` con i valori misurati, chiudere il vero loop di feedback boost (R2≈560kΩ, R3=10kΩ), ricostruire il modello BVD
5. Calcolare/avvolgere il trasformatore con i nuovi valori, progettare l'interfaccia U2→ADC (ancora mancante), definire il clock PIO a 31,25MHz
6. **Solo a quel punto** confermare e finalizzare l'ordine TME consolidato (componenti rimanenti + eventuali nuovi: resistori 270kΩ, RT/CT UC3843, rete compensazione, filtro current sense)
7. Assemblare e validare fisicamente lo Step 1
8. Solo dopo Step 1 validato con hardware reale: ordinare componenti Step 2
9. Step 3-4: da rivalutare in futuro come estensione, non nel piano attuale