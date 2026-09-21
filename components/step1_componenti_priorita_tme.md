# Step 1 — Checklist componenti finale (post-simulazione LTspice)

Legenda: 🟢 Confermato/posseduto/in carrello — 🔴 Da cercare/ordinare

## GIÀ IN MANO (posseduti, confermati dall'utente)
Oscilloscopio FNIRSI DPOX180H + generatore integrato, sonde oscilloscopio ×2, cavo coassiale BNC-BNC posseduto, PC per elaborazione Python, cavo USB Raspberry Pi Pico H, saldatore+stagno+flussante, filo rigido per collegamenti, breadboard e PCB proto, kit cavi dupont, multimetro.

## ORDINE 1 — Digitale + Alimentazione (ATX)
| Elemento | Stato | Codice/Dettaglio |
|---|---|---|
| Raspberry Pi Pico H | 🟢 | **SC0917** — in carrello TME, 5.48€ |
| Zoccolo DIP8 ×3 | 🟢 | **DS1001-01-08BT1NSF6X-JKB** (GOLD-8P, Connfly) — in carrello TME |
| Alimentatore ATX da PC usato | 🔴 | Recupero |
| Breakout board ATX | 🔴 | Probabilmente non su TME — AliExpress/eBay |
| LM317 | 🔴 | Cercare "LM317T" su TME |
| Dissipatore piccolo per LM317 | 🔴 | Da cercare TME |
| Resistenza di Bleeder 1MΩ, 1-2W | 🔴 | Da cercare TME |
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
| Condensatore elettrolitico 47µF 400V THT (C1) | 🔴 | **PF2G470MNN1625** (verificare su TME) |
| Resistenza current sense 1Ω 1W (R1) | 🔴 | Cercare su TME |
| Resistenza gate M1 1kΩ (R11) | 🔴 | Cercare su TME |
| Resistenza carica C2 1kΩ (R12) | 🔴 | Cercare su TME |
| Resistori partitore feedback 10kΩ (R2), 1kΩ (R3) | 🔴 | Cercare su TME |

## ORDINE 3 — Blocco Ricezione + Trasduttore
| Elemento | Stato | Codice/Dettaglio |
|---|---|---|
| Trasduttore NDT 5MHz | 🔴 | AliExpress |
| 1N4148-DIO ×2 | 🟢 | In carrello TME, 0.20€ |
| Resistenza limitatrice T/R 100Ω (R4) | 🔴 | **1W o 2W THT, film metallico** (non 1/4W) — non per motivi termici (potenza media solo 0.033W, duty cycle 0.02%), ma per tenuta dielettrica: il picco istantaneo di 130V ai capi rischia breakdown interno su un corpo resistivo troppo piccolo come quello di un 1/4W standard. Cercare su TME "resistore film metallico 100R 1W THT" |
| LM6172IN/NOPB | 🟢 | In carrello TME, 17.08€ |
| Cavo coassiale BNC-BNC aggiuntivo | 🟢 | **CABLE-505-50-1** (cod. produttore 50272) — in carrello TME, 2.90€ |
| Connettore BNC femmina THT | 🟢 | **B6251C1-NT3G-50** — confermato |
| Resistori guadagno RX 1kΩ (R6), 10kΩ (R7) | 🔴 | Cercare su TME |
| Resistori bias 10kΩ (R9), 10kΩ (R10) | 🔴 | Cercare su TME |
| Capacitore compensazione 10pF (C5) | 🔴 | Cercare su TME |
| Capacitore accoppiamento AC 10µF (C6) | 🔴 | Cercare su TME |

## ORDINE 4 — Digitalizzazione e Test
| Elemento | Stato | Codice/Dettaglio |
|---|---|---|
| AD9280ARSZ | 🟢 | In carrello TME, 27.68€ — **0 in stock, monitorare disponibilità** |
| Adattatore PCB SSOP28→DIP28 | 🔴 | Probabilmente non su TME — AliExpress/eBay |
| Vasca/contenitore acqua | 🔴 | Ferramenta/casalinghi |
| Piastra target di test | 🔴 | Ferramenta |
| Calibro digitale | 🔴 | Amazon |

## Nota di assemblaggio
Il loop di feedback reale (R2/R3 → pin feedback UC3843BN) va collegato fisicamente nell'hardware — in simulazione si usava un riferimento fisso, ma sul circuito vero serve per la regolazione automatica della tensione boost.

## Totale confermato in carrello TME (ordini 1-4)
5.48+1.44+1.45+8.16+2.76+0.72+0.94+0.81+1.13+0.20+17.08+2.90+27.68 ≈ **70.75€**
