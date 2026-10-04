# Codici precisi verificati — componenti Step 1 (fonte TME)

*Riscritto per riflettere i componenti effettivamente scelti e verificati su TME durante il lavoro (sostituisce la versione precedente basata su MD1213/TC6320/AD8331/HE3621, componenti di un percorso di progettazione diverso da quello poi effettivamente seguito).*

**Metodo di ricerca**: cercare direttamente su tme.eu (o tme-italia.it) il codice esatto o il termine indicato. Verificare sempre "in stock presso TME" prima di ordinare — alcuni codici (es. AD9280ARSZ) risultano periodicamente a 0 stock.

---

## Blocco 1 — Pulser + Boost HV

| Componente | Codice TME | Produttore | Stato verificato |
|---|---|---|---|
| Driver MOSFET pulser | **MC33151PG** | ONSEMI | ✅ In stock (verificato, 42pz) |
| MOSFET HV (pulser + boost) | **STP8NK100Z** | STMicroelectronics | ✅ In stock (58pz), 1000V/4.3A/160W, TO220-3 — usarne 2 (1 pulser, 1 boost) |
| Controller PWM boost | **UC3843BN** | STMicroelectronics | ✅ In stock (192pz), MiniDIP8 |
| Capacitore storage pulser | **FKP1T014705D00JSSD** (FKP1-4.7N/1600) | WIMA | ⚠️ "Prodotto difficilmente disponibile" ma in stock (190pz) — 4.7nF, 1.6kVDC |
| Diodo rettificatore/protezione | **UF4007-DIO** | Diotec Semiconductor | ✅ In stock (20.640pz), 1kV, 75ns — servono 3 (D1 boost, D2/D3 pulser) |
| Nucleo trasformatore | **E20/10/6-3C94** | Ferroxcube | ✅ In stock (400pz) — **⚠️ [V] verificare quantità**: servono tipicamente 2 metà (set E+E o E+I) per un trasformatore completo, non 1 pezzo. Verificare anche se esiste variante già gappata su TME (più semplice del traferro calcolato a mano) |
| Coil former | **EF20-K-H-8P** | Feryster | ✅ In stock (71pz) — compatibilità con E20/10/6 confermata dalla pagina prodotto TME stessa |
| Capacitore filtro uscita boost | **PF2G470MNN1625** (47µF, 400VDC) | — | ⚠️ Da verificare disponibilità specifica su TME |

---

## Blocco 2 — T/R switch + Ricezione

| Componente | Codice TME | Produttore | Stato verificato |
|---|---|---|---|
| Diodi T/R switch | **1N4148-DIO** | Diotec Semiconductor | ✅ In stock (291.221pz) — servono 2 (D4/D5) |
| Op-amp RX | **LM6172IN/NOPB** | Texas Instruments | ✅ In stock (1pz — scorta minima, verificare prima di ordinare), package PDIP-8 THT, nessun adattatore necessario |
| Connettore BNC femmina | **B6251C1-NT3G-50** | Amphenol RF | ✅ Confermato, THT per PCB |
| Cavo coassiale BNC-BNC (extra) | **CABLE-505-50-1** (cod. produttore 50272) | Goobay | ✅ In stock (257pz), 1m |

---

## Blocco 3 — Digitale

| Componente | Codice TME | Produttore | Stato verificato |
|---|---|---|---|
| Microcontrollore | **SC0917** (Raspberry Pi Pico H) | Raspberry Pi | ✅ In stock (308pz) — versione con header preassemblati |
| Zoccolo DIP8 | **DS1001-01-08BT1NSF6X-JKB** (GOLD-8P) | Connfly | ✅ In stock (87.651pz) — servono 3 (LM6172IN, MC33151PG, UC3843BN) |
| ADC | **AD9280ARSZ** | Analog Devices | ⚠️ "Prodotto difficilmente disponibile", **0 in stock** al momento — monitorare. 8-bit, 32Msps. **[?] Nota da revisione 29/09/2026**: la documentazione del progetto affermava "stesso chip usato da un0rick/pic0rick", ma altrove si cita pic0rick come "60Msps 10-bit" — le due cifre sono incompatibili con l'AD9280. Una delle due affermazioni è sbagliata; verificare sulla documentazione ufficiale pic0rick quale ADC usa realmente prima di riaffermare la corrispondenza |

---

## Componenti generici ancora da cercare (nessun codice unico — categoria standard)

| Componente | Valore | Note |
|---|---|---|
| Resistenza current sense (R1) | 1Ω, 1W | THT |
| Resistenze gate/carica (R11, R12) | 1kΩ | THT |
| Resistori partitore feedback (R2, R3) | ~~10kΩ, 1kΩ~~ → **R2≈560kΩ (2×270kΩ), R3=10kΩ** | THT — ricalcolati 29/09/2026 per ottenere realmente 142V (vedi `simulazione_ltspice_handoff.md`), non più 10kΩ/1kΩ |
| Condensatore in serie a R6 | 100nF-1µF | THT — **[?] da verificare se serve** (dipende dall'esito del test `.op` sul bug R6 segnalato in revisione) |
| RT/CT frequenza UC3843 | Da calcolare (~100kHz) | THT — **mancante dalla lista originale**, sostituito in simulazione da generatori ideali |
| Rete compensazione pin COMP UC3843 | Da calcolare | THT — **mancante dalla lista originale** |
| Filtro RC current sense UC3843 | Da calcolare | THT — **mancante dalla lista originale** |
| Resistenza limitatrice T/R (R4) | 100Ω, **1-2W film metallico** (non 1/4W — tenuta dielettrica contro picco ~130V, non calore) | THT |
| Resistori guadagno RX (R6, R7) | 1kΩ, 10kΩ | THT |
| Resistori bias RX (R9, R10) | 10kΩ ×2 | THT |
| Capacitore compensazione RX (C5) | 10pF | THT |
| Capacitore accoppiamento AC (C6) | 10µF | THT |
| Resistenza bleeder | 1MΩ, 1-2W | Sicurezza scarica C1 |
| Breakout board ATX | — | Probabilmente non su TME — AliExpress/eBay |
| LM317 | — | Cercare "LM317T" su TME |
| Adattatore SSOP28→DIP28 | — | Probabilmente non su TME — AliExpress/eBay |

---

## Riepilogo ordine consolidato TME

⚠️ **Ordine congelato dalla revisione del 29/09/2026** (vedi `revisione-progetto-29-09-2026.md`) — il totale sotto non è più attendibile: cambiano R2/R3 (partitore feedback), possibile rimozione LM317/dissipatore, da verificare quantità nucleo trasformatore, mancano 4 componenti di supporto UC3843. **Ordinare per ora solo il trasduttore 5P20N**; il resto resta in attesa dei ricalcoli (dettagli in `step1_componenti_priorita_tme.md`).

~~Totale confermato in carrello (voci con stock verificato sopra): ~70,75€~~ — valore storico, non più valido.

## Nota generale

A differenza di un progetto basato su chip di nicchia per ultrasuoni professionali (dove capita spesso di trovare package discontinuati), i componenti scelti in questo progetto sono in gran parte parti standard/diffuse (MOSFET, controller PWM, op-amp, diodi comuni) — il principale punto di attenzione resta la disponibilità di **AD9280ARSZ** (chip di nicchia, ADC specifico) e **LM6172IN** (scorta minima rilevata, 1 solo pezzo in stock al momento della verifica).

**Nota aggiunta in revisione — range AVDD AD9280**: verificato leggendo il datasheet reale, AVDD accetta 2,7-5,5V — questo rende possibile alimentare l'ADC direttamente dal rail 3,3V dell'ATX, senza il LM317 inizialmente previsto (vedi nota in `step1_componenti_priorita_tme.md`, sezione Bilancio di potenza).