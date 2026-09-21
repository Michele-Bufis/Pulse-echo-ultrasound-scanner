# Step 3 — Switching matrix e 4 elementi (ACCANTONATO — stretch goal futuro)

> ⚠️ **Stato aggiornato: questo step è stato esplicitamente valutato e ACCANTONATO per motivi di budget** durante la pianificazione (lo switching matrix multistatico a 4 elementi risultava troppo costoso — modulo boost HV professionale da solo costava 150-200€, più tutto il resto). Il progetto attuale procede con **Approccio A: singolo elemento + rotazione meccanica** (vedi Step 2).
>
> Questo documento resta come riferimento tecnico se in futuro si deciderà di riprendere l'estensione a 4 elementi come **stretch goal** — concettualmente è la stessa famiglia di tecniche del progetto **Midjourney Medical** (scanner full-body a ultrasuoni, migliaia di elementi, 2 petaflop di calcolo) applicata a scala hobbistica: buona narrativa di portfolio se ripreso in futuro, non necessario per completare il progetto base.

---

## 1. Relè meccanici vs switch analogico HV a stato solido

| Aspetto | Relè meccanici (reed relay HV) | Switch analogico HV a stato solido |
|---|---|---|
| Velocità | Millisecondi | Nanosecondi-microsecondi |
| Adeguato per pulse-echo per elemento (non simultaneo) | Sì | Over-engineering per questo caso |
| Debug | Facile (multimetro, "click") | Difficile (scatola nera) |
| Costo | Pochi euro/canale | 10-20€+ per chip |

**Raccomandazione se ripreso**: relè reed HV, stesso ragionamento costo/apprendimento del piano originale.

---

## 2. Schema concettuale

```
Pulser (Step 1) -----|  Matrice 4x2  |---- Elemento 1/2/3/4
RX front-end --------|  (relè HV)    |     comandata da 4 pin GPIO RP2040
```

Multiplexer 4:1 verso lo stesso ramo pulser/RX condiviso — non serve una vera matrice 4x4 per pulse-echo per elemento.

---

## 3. Componenti (se ripreso in futuro)

| Blocco | Componente | Note |
|---|---|---|
| Relè | Reed relay HV ~200-400V (es. HE3621A1210) | 4 + 2-4 scorta |
| Driver relè | Transistor NPN (2N2222A/MMBT2222A) + diodo flyback 1N4148 | RP2040 non pilota direttamente la bobina |
| Trasduttori aggiuntivi | 3× stesso modello dello Step 1 (5MHz) | Coerenza dati tra elementi |
| Anello aggiornato | Stampa 3D con 4 sedi | Ristampa se già previsto nel design Step 2 |
| Cavi coassiali aggiuntivi | 3× stessa lunghezza tra loro | Evita asimmetrie tra canali |

---

## 4. Assemblaggio — sequenza (se ripreso)

1. Stadio driver relè, un canale alla volta, test con multimetro
2. Ripeti per 4 canali, testati isolatamente
3. Integrazione con ramo pulser/RX condiviso (uscita T/R switch Step 1)
4. Test oscilloscopio: nessun impulso HV deve trapelare sui canali non selezionati
5. Montaggio 3 trasduttori aggiuntivi, cablaggio
6. Test funzionale completo, coerenza tra i 4 elementi su stesso target

---

## 5. Lista acquisti indicativa (se ripreso)

| Cosa | Termine ricerca | Prezzo |
|---|---|---|
| Reed relay HV ×6-8 | `HE3621 reed relay` | 2-5€/pz |
| Transistor driver | `MMBT2222A` | 1-2€/lotto |
| 3× trasduttori 5MHz aggiuntivi | Stesso termine Step 1 | ~50€ × 3 (stesso modello AliExpress scelto) |
| 3× cavi coassiali | `SMA cable coaxial 50 ohm` | 3-6€ × 3 |

**Totale indicativo se ripreso: ~180-220€** (aggiornato: i trasduttori reali scelti nel progetto costano ~50€/pz, non i pochi euro dei dischi grezzi ipotizzati originariamente).

---

## 6. Checklist (se ripreso)

- [ ] Ciascun relè commutato e verificato isolatamente
- [ ] Nessun impulso HV trapela sui canali non selezionati
- [ ] I 4 elementi danno risultati coerenti sullo stesso target
- [ ] Firmware gestisce correttamente angolo+elemento nel salvataggio dati
