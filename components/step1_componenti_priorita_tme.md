# Step 1 — Checklist componenti finale (post-simulazione LTspice)

Legenda: 🟢 Confermato/posseduto/in carrello — 🟡 Trovato/scelto, ordine da finalizzare — 🔴 Da cercare/ordinare

> ⚠️ **Revisione del 29/09/2026** (vedi `revisione-progetto-29-09-2026.md`): diversi valori in questa checklist sono cambiati dopo un ricalcolo del partitore di feedback boost e una verifica di alcuni consumi. **Ordine di lavoro concordato: congelare il resto del carrello TME, ordinare per ora solo il trasduttore 5P20N** — il resto va confermato solo dopo aver chiuso i punti aperti su `Eco.asc` (vedi file di revisione, Sez. 8).

## GIÀ IN MANO (posseduti, confermati dall'utente)
Oscilloscopio FNIRSI DPOX180H + generatore integrato, sonde oscilloscopio ×2, cavo coassiale BNC-BNC posseduto, PC per elaborazione Python, cavo USB Raspberry Pi Pico H, saldatore+stagno+flussante, filo rigido per collegamenti, breadboard e PCB proto, kit cavi dupont, multimetro.

## ORDINE 1 — Digitale + Alimentazione (ATX)
| Elemento | Stato | Codice/Dettaglio |
|---|---|---|
| Raspberry Pi Pico H | 🟢 | **SC0917** — in carrello TME, 5.48€ |
| Zoccolo DIP8 ×5 (ne servono 3) | 🟢 | **DS1001-01-08BT1NSF6X-JKB** (GOLD-8P, Connfly) — in carrello TME, 1.44€ |
| Alimentatore ATX da PC usato | 🔴 | Recupero |
| Breakout board ATX 24pin | 🟢 | AliExpress — https://it.aliexpress.com/item/1005005137163760.html |
| LM317T | 🟡 | In carrello TME, 0.63€ — **⚠️ [V] probabilmente non necessario**: l'ATX ha già un rail 3,3V diretto, e l'AD9280 accetta AVDD 2,7-5,5V (verificato sul datasheet reale, non solo a memoria) — usare il rail 3,3V ATX con un filtro LC/ferrite invece del regolatore dedicato risparmia il componente, il dissipatore, e i ~0,9W di dissipazione. Decisione da confermare prima di saldare |
| Dissipatore TO-220 per LM317 ×10 | 🟡 | AliExpress (T8WC, 15×10×20mm) — https://it.aliexpress.com/item/1005006082544471.html, 3.89€ — **necessario solo se si conferma l'uso del LM317** (vedi riga sopra) |
| Resistenza di Bleeder 1MΩ, 1-2W | 🟢 | **MF01SFF1004A10** (metal film, 1W) — in carrello TME, 1.45€ |
| Occhiali di protezione | 🔴 | Ferramenta/Amazon |

## ORDINE 2 — Blocco Pulser + Boost HV
| Elemento | Stato | Codice/Dettaglio |
|---|---|---|
| MC33151PG | 🟢 | In carrello TME, 1.45€ |
| STP8NK100Z ×2 | 🟢 | In carrello TME, 8.16€ (1 pulser, 1 boost) |
| WIMA FKP1 4.7nF ×2 | 🟢 | **FKP1T014705D00JSSD** — in carrello TME, 2×1.38€ |
| UF4007-DIO ×3 | 🟢 | In carrello TME (2 righe: 1+2), 0.72€ tot |
| UC3843BN | 🟢 | In carrello TME, 0.94€ |
| Nucleo E20/10/6-3C94 ×1 | 🟡 | In carrello TME, 0.81€ — **⚠️ [V] verificare quantità**: un trasformatore E20/10/6 richiede tipicamente **due metà** (set E+E o E+I), non 1 pezzo — controllare se il codice in carrello è già per il set completo o per una sola metà. Verificare anche disponibilità di una variante già gappata (AL≈200-250nH), più semplice da usare del calcolo del traferro a mano |
| Resistenza gate/frequenza UC3843 (RT, CT) | 🔴 | **Mancante dalla lista originale** — fissa la frequenza di oscillazione del controller PWM, sostituita in simulazione da generatori ideali (V2/V3). Valori da calcolare per ≈100kHz secondo datasheet UC3843 |
| Rete di compensazione pin COMP | 🔴 | **Mancante dalla lista originale** — stabilizza il loop di controllo, sostituita in simulazione da U1/S1 ideali |
| Filtro RC current sense | 🔴 | **Mancante dalla lista originale** — filtra il rumore di commutazione sul pin current sense (collegato a R1) |
| Resistenza gate + pulldown (driver M1) | 🔴 | **Mancante dalla lista originale** — R11 (1kΩ, già in lista come resistenza gate) copre solo parte del requisito; verificare se serve anche un pulldown dedicato |
| Coil former EF20-K-H-8P ×2 | 🟢 | In carrello TME, 1.13€ |
| Filo di rame smaltato | 🔴 | Da recuperare (costo zero) |
| Condensatore elettrolitico 47µF 400V THT (C1) | 🟢 | **PF2G470MNN1625** — in carrello TME, 0.85€ |
| Resistenza current sense 1Ω 1W (R1) | 🟢 | **KNP01WS-1R** — in carrello TME, 0.73€ |
| Resistenza gate M1 1kΩ (R11) | 🟢 | **1W-1K-1%** (MF01SFF1001A10) — in carrello TME |
| Resistenza carica C2 1kΩ (R12) | 🟢 | **1W-1K-1%** (MF01SFF1001A10) — in carrello TME |
| Resistori partitore feedback ~~10kΩ (R2), 1kΩ (R3)~~ → **560kΩ o 2×270kΩ (R2), 10kΩ (R3)** | 🔴 | **⚠️ [C] valori ricalcolati (29/09/2026)**: i valori precedenti (R2=10kΩ, R3=1kΩ) davano solo 27,5V invece di 142V una volta chiuso il loop reale (Vout=2,5×(1+R2/R3), riferimento UC3843≈2,5V). Nuovi valori: **R3=10kΩ** (stesso codice MF01SFF1002A10 già in carrello, verificare stock), **R2≈560kΩ** (consigliato 2×270kΩ in serie per la tenuta di tensione, non ancora in carrello) — dissipazione risultante ≈35mW, molto più bassa dei ≈0,35W di una combinazione scalata ingenuamente. Aggiungere anche uno zener di clamp sul pin feedback come sicurezza |

## ORDINE 3 — Blocco Ricezione + Trasduttore
| Elemento | Stato | Codice/Dettaglio |
|---|---|---|
| Trasduttore NDT 5MHz, cristallo 20mm | 🟢 | YUSHI/XMSJ 5P20N, BNC — https://it.aliexpress.com/item/1005011632945564.html, 55.69€ — ⚠️ tenuta stagna per immersione da confermare col venditore prima dell'ordine |
| 1N4148-DIO ×2 | 🟢 | In carrello TME, 0.20€ |
| Resistenza limitatrice T/R 100Ω (R4) | 🟢 | **MF01SFF1000A10** (metal film, 1W) — in carrello TME, 0.61€ — corretto: 1W non 1/4W, per tenuta dielettrica contro picco ~130V |
| LM6172IN/NOPB | 🟢 | In carrello TME, 17.08€ |
| Cavo coassiale BNC-BNC aggiuntivo | 🟢 | **CABLE-505-50-1** (cod. produttore 50272) — in carrello TME, 2.90€ |
| Connettore BNC femmina THT | 🟢 | **B6251C1-NT3G-50** — confermato |
| Resistori guadagno RX 1kΩ (R6), 10kΩ (R7) | 🟡 | R6: 1W-1K-1% ✅ in carrello. R7 10kΩ: **MF01SFF1002A10 — ⚠️ 0 in stock presso TME al momento**, stesso codice di R2 |
| Resistori bias 10kΩ (R9), 10kΩ (R10) | 🟡 | Stesso codice **MF01SFF1002A10 — ⚠️ 0 in stock presso TME**, servono altri 2 pezzi (4 totali con R2/R7) |
| Capacitore compensazione 10pF (C5) | 🟢 | **CC-10/100** (ceramico, C0G/NP0) — in carrello TME, 2.45€ (min. ordine 100pz) |
| Capacitore accoppiamento AC 10µF (C6) | 🟢 | **EWH1HM100D11X25T** (elettrolitico 10µF/50V THT) — in carrello TME |

## ORDINE 4 — Digitalizzazione e Test
| Elemento | Stato | Codice/Dettaglio |
|---|---|---|
| AD9280ARSZ | 🟢 | In carrello TME, 27.68€ — **0 in stock, monitorare disponibilità** |
| Adattatore PCB SSOP28→DIP28 | 🔴 | Probabilmente non su TME — AliExpress (termine ricerca: "SSOP28 SOP28 TSSOP28 to DIP28 adapter 0.65mm") |
| Vasca/contenitore acqua | 🔴 | Ferramenta/casalinghi |
| Piastra target di test | 🔴 | Ferramenta |
| Calibro digitale | 🔴 | Amazon |

## ⚠️ Azione richiesta prima di finalizzare l'ordine TME
Il codice **MF01SFF1002A10 (10kΩ, 1W metal film)** risulta a 0 stock presso TME nonostante sia nel carrello. Serve per **4 resistori: R3, R7, R9, R10** (R2 non usa più questo codice — dopo il ricalcolo del partitore feedback, R2 è ora ≈560kΩ, non 10kΩ, vedi sopra). Verificare disponibilità reale al checkout o cercare un'alternativa 10kΩ/1W metal film THT (es. altra serie Royalohm o Yageo su TME) prima di confermare l'ordine. **Aggiungere anche 2× resistori 270kΩ per R2**, non ancora in carrello.

## Bilancio di potenza — verifica ATX + LM317 (o rail 3,3V diretto)

*Analisi fatta su carta per verificare che l'alimentazione scelta regga tutti i blocchi insieme, non solo singolarmente.*

### Consumi per blocco
| Blocco | Componente | Corrente tipica | Rail |
|---|---|---|---|
| Microcontrollore | RP2040 (Pico H) | ~40-60mA (attivo, PIO+USB) | 5V (VSYS) |
| ADC | AD9280ARSZ | 31,7mA @3V (da datasheet) | 3V (regolato, LM317) |
| Amplificatore RX | LM6172IN | 4,6mA/canale | 12V singola |
| Driver pulser | MC33151PG | ~5-10mA quiescente | 12V |
| Controller boost | UC3843BN | ~11-17mA | 12V |
| MOSFET pulser/boost ×2 | STP8NK100Z | Trascurabile in quiescenza | pilotati da driver |
| Motore Step 2 (rail separato) | NEMA17 + A4988 | ~0,4-1,2A/fase | 12V dedicato, separato |

### Verifica capacità ATX
Elettronica di segnale/HV messa insieme: **~100-150mA totali**. Un ATX standard (anche datato, 300-500W) eroga tipicamente 10-20A sul rail 12V e 15-20A sul rail 5V — **ordini di grandezza sopra il necessario**. Il motore Step 2 (fino a ~1-2A) resta su rail separato come già previsto, senza competere con l'elettronica di segnale. **Nessun rischio strutturale sul bilancio di potenza complessivo.**

### Dimensionamento LM317 (regolatore parte analogica, es. 3V per AD9280)
```
Corrente stimata attraverso LM317 ≈ 32-37mA (AD9280) + margine ≈ 50-100mA
Caduta di tensione (ingresso 12V ATX → uscita 3V) = 9V
Potenza dissipata sul LM317 ≈ 9V × 0,1A ≈ 0,9W
```
Con il dissipatore TO-220 già scelto (T8WC), 0,9W è gestibile senza problemi termici.

### ⚠️ Punto RISOLTO in revisione — LM317 probabilmente non necessario
**[V] Confermato leggendo il datasheet reale dell'AD9280** (non solo a memoria): `AVDD, Operating Voltage: Min 2,7V — Max 5,5V`. Il rail **3,3V già presente sull'ATX standard** rientra comodamente in questo range, senza bisogno del LM317 né del suo dissipatore né della dissipazione di ≈0,9W calcolata sopra.

**Raccomandazione aggiornata:** usare direttamente il rail 3,3V dell'ATX, con un filtro LC o una ferrite in serie per ripulire eventuale rumore di commutazione dello switching ATX prima che arrivi all'AD9280 (la parte più sensibile al rumore di alimentazione del circuito). Il LM317 resta un'opzione di ripiego se il rumore sul rail 3,3V ATX risultasse troppo alto in pratica — da valutare solo dopo un test empirico col multimetro/oscilloscopio sul rail reale.

---


## Nota di assemblaggio
Il loop di feedback reale (R2/R3 → pin feedback UC3843BN) va collegato fisicamente nell'hardware — in simulazione si usava un riferimento fisso, ma sul circuito vero serve per la regolazione automatica della tensione boost.

## Totale confermato in carrello TME (ordini 1-4, componenti attivi + passivi aggiunti) — ⚠️ DA RICALCOLARE
~~70.75€ (batch originale) + 13.93€ (batch passivi) ≈ ~84.70€~~ — **questo totale non è più affidabile** dopo le modifiche della revisione del 29/09/2026: R2 cambia da 10kΩ a ~560kΩ (2×270kΩ nuovi, non ancora in carrello), LM317+dissipatore possibilmente da rimuovere (-4,52€ circa), nucleo trasformatore da verificare in quantità, 4 componenti UC3843 mancanti da aggiungere (RT/CT, compensazione, filtro RC, pulldown gate). **Non finalizzare l'ordine TME finché questi punti non sono chiusi** — vedi nota "ordine congelato" in cima al file.

## Fuori TME — confermati
| Elemento | Fonte | Prezzo |
|---|---|---|
| Breakout board ATX 24pin | AliExpress | verificare al checkout |
| Dissipatore TO-220 ×10 (LM317) | AliExpress | 3.89€ |
| Trasduttore NDT 5MHz 20mm | AliExpress | 55.69€ |

## Fuori TME — ancora da trovare
| Elemento | Fonte suggerita |
|---|---|
| Adattatore SSOP28→DIP28 | AliExpress |
| Occhiali di protezione | Ferramenta/Amazon |
| Vasca/contenitore acqua | Ferramenta/casalinghi |
| Piastra target di test | Ferramenta |
| Calibro digitale | Amazon |
| Alimentatore ATX da PC usato | Recupero |
| Filo di rame smaltato | Recupero |