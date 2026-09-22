# Step 1 — Checklist componenti finale (post-simulazione LTspice)

Legenda: 🟢 Confermato/posseduto/in carrello — 🟡 Trovato/scelto, ordine da finalizzare — 🔴 Da cercare/ordinare

## GIÀ IN MANO (posseduti, confermati dall'utente)
Oscilloscopio FNIRSI DPOX180H + generatore integrato, sonde oscilloscopio ×2, cavo coassiale BNC-BNC posseduto, PC per elaborazione Python, cavo USB Raspberry Pi Pico H, saldatore+stagno+flussante, filo rigido per collegamenti, breadboard e PCB proto, kit cavi dupont, multimetro.

## ORDINE 1 — Digitale + Alimentazione (ATX)
| Elemento | Stato | Codice/Dettaglio |
|---|---|---|
| Raspberry Pi Pico H | 🟢 | **SC0917** — in carrello TME, 5.48€ |
| Zoccolo DIP8 ×5 (ne servono 3) | 🟢 | **DS1001-01-08BT1NSF6X-JKB** (GOLD-8P, Connfly) — in carrello TME, 1.44€ |
| Alimentatore ATX da PC usato | 🔴 | Recupero |
| Breakout board ATX 24pin | 🟢 | AliExpress — https://it.aliexpress.com/item/1005005137163760.html |
| LM317T | 🟢 | In carrello TME, 0.63€ |
| Dissipatore TO-220 per LM317 ×10 | 🟢 | AliExpress (T8WC, 15×10×20mm) — https://it.aliexpress.com/item/1005006082544471.html, 3.89€ |
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
| Nucleo E20/10/6-3C94 ×1 | 🟢 | In carrello TME, 0.81€ |
| Coil former EF20-K-H-8P ×2 | 🟢 | In carrello TME, 1.13€ |
| Filo di rame smaltato | 🔴 | Da recuperare (costo zero) |
| Condensatore elettrolitico 47µF 400V THT (C1) | 🟢 | **PF2G470MNN1625** — in carrello TME, 0.85€ |
| Resistenza current sense 1Ω 1W (R1) | 🟢 | **KNP01WS-1R** — in carrello TME, 0.73€ |
| Resistenza gate M1 1kΩ (R11) | 🟢 | **1W-1K-1%** (MF01SFF1001A10) — in carrello TME |
| Resistenza carica C2 1kΩ (R12) | 🟢 | **1W-1K-1%** (MF01SFF1001A10) — in carrello TME |
| Resistori partitore feedback 10kΩ (R2), 1kΩ (R3) | 🟡 | R3: 1W-1K-1% ✅ in carrello. R2 10kΩ: **MF01SFF1002A10 — ⚠️ 0 in stock presso TME al momento**, verificare disponibilità o trovare alternativa prima di ordinare |

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
Il codice **MF01SFF1002A10 (10kΩ, 1W metal film)** risulta a 0 stock presso TME nonostante sia nel carrello. Serve per **4 resistori: R2, R7, R9, R10**. Verificare disponibilità reale al checkout o cercare un'alternativa 10kΩ/1W metal film THT (es. altra serie Royalohm o Yageo su TME) prima di confermare l'ordine.

## Nota di assemblaggio
Il loop di feedback reale (R2/R3 → pin feedback UC3843BN) va collegato fisicamente nell'hardware — in simulazione si usava un riferimento fisso, ma sul circuito vero serve per la regolazione automatica della tensione boost.

## Totale confermato in carrello TME (ordini 1-4, componenti attivi + passivi aggiunti)
70.75€ (batch originale) + 13.93€ (batch passivi: C1, nucleo dup., LM317T, coil former dup., GOLD-8P dup., UF4007 dup., 1N4148 dup., R1, bleeder, R3/R6/R11/R12, R4, C5) + C6 (pochi centesimi) ≈ **~84.70€**

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