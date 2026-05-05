# Saturation Studio

Carica un CSV con spesa pubblicitaria e ricavi giornalieri. Ottieni una curva di saturazione Hill, raccomandazioni di spesa ottimale su orizzonti LTV e un grafico di scenario modificabile in tempo reale.

**Funziona interamente nel browser.** Il tuo CSV non lascia mai il tuo dispositivo — nessun server, nessun upload, nessuna telemetria.

## Utilizzo

**La via più semplice:** [apri la versione live](https://YOUR-USERNAME.github.io/saturation-tool/) (sostituisci `YOUR-USERNAME` con il tuo handle GitHub dopo aver abilitato GitHub Pages — vedi "Hosting" in seguito).

**Offline:** scarica `index.html` e aprilo in qualsiasi browser. File singolo, nessuna installazione.

## Formato CSV

Scarica [`template.csv`](./template.csv) come file di partenza.

| Colonna | Obbligatoria | Note |
|---|---|---|
| `date` | sì | `YYYY-MM-DD` o `M/D/YYYY` |
| `spend` | sì | Spesa pubblicitaria giornaliera in dollari (numeri grezzi — `12450`, non `$12,450`) |
| `revenue` | sì | Ricavi pubblicitari giornalieri attribuiti |
| `purchases` | opzionale | Numero ordini giornalieri — usato per proiettare ordini / nuovi clienti |

Si consigliano 90+ giorni. 365 giorni fornisce il fit più stabile. I giorni con $0 di spesa o $0 di ricavi vengono esclusi automaticamente.

## Cosa calcola

- **Fit di saturazione Hill** sui dati giornalieri: `revenue = V_max · s^h / (K^h + s^h)`
- **Spesa giornaliera ottimale** per sette obiettivi diversi:
  - Massimo margine di contribuzione oggi
  - Massimo CM LTV a 3, 6 e 12 mesi
  - Massimo ricavo, massimo ricavo NC
  - ROAS target (con bisezione peak-ROAS — segnala obiettivi irraggiungibili)
- **Curve mensili** quando un mese ha 8+ osservazioni; altrimenti il fit globale scalato per stagionalità del mese dell'anno
- **Fit pesato per recency** (emivita predefinita di 60 giorni) in modo che la curva rifletta il regime di efficienza attuale, non quello dell'anno scorso

## Parametri configurabili

| | Predefinito | Cosa fa |
|---|---|---|
| Costo variabile % | 35 | COGS + spedizione + elaborazione come % dei ricavi |
| Quota NC % | 60 | Quota di ricavi da nuovi clienti |
| AOV mediano | dai dati | Usato per proiettare ordini e nuovi clienti |
| LTV 3m / 6m / 12m | 1,4× / 1,8× / 2,5× | Ricavi cumulativi del cliente all'orizzonte ÷ ricavi del primo ordine. **Gli input più importanti — recuperali dal tuo strumento di coorte.** |
| Emivita recency | 60 giorni | Un punto N giorni fa riceve peso `0,5^(N/half_life)` nel fit |
| Decadimento adstock (λ) | 0 | Guadagno della spesa stazionaria = `1/(1-λ)`. Prova 0,3 per il tipico carryover Meta. |
| Efficienza adj. | 0% | Scala `V_max` se ti aspetti che la piattaforma performi meglio/peggio del fit storico |
| ROAS target | 1,2× | Risolve al contrario: spesa massima che mantiene ancora questo ROAS |
| Spesa giornaliera attuale | dalla media dati | Dove si trova la riga "oggi" sul grafico |

## Hosting su GitHub Pages

```bash
# 1. Pusha questo repo su GitHub
git remote add origin https://github.com/YOUR-USERNAME/saturation-tool.git
git push -u origin main

# 2. Nella pagina del repo GitHub:
#    Settings → Pages → Source → Deploy from branch
#    Branch: main, cartella: / (root) → Save

# 3. Attendi ~30 secondi, il tuo strumento è live su:
#    https://YOUR-USERNAME.github.io/saturation-tool/
```

Quell'URL è quello che condividi con chiunque — nessuna installazione, basta caricare il CSV.

## Avvertenze

Il modello è **indicativo**, non preciso:

- **I moltiplicatori LTV fanno la maggior parte del lavoro.** I valori predefiniti DTC generici probabilmente non sono giusti per la tua coorte. Un oscillazione del LTV a 12 mesi da 2,5× a 1,5× ribalta la maggior parte delle raccomandazioni.
- **Le raccomandazioni oltre il tuo intervallo di spesa osservato sono estrapolazioni.** Se i tuoi dati arrivano a $30K/giorno, una raccomandazione di $60K/giorno è una stima del modello.
- **Il fit è osservazionale, non causale.** I ricavi riportati potrebbero essere sovra-attribuiti dalla piattaforma. Uno studio di geo lift (GeoLift, ecc.) ti dice il ROAS incrementale genuino — inseriscilo nei tuoi input VC% / LTV.
- **Il timing del flusso di cassa è importante.** La matematica dice redditizio al mese 12 — ma hai speso il giorno 1 e non vedrai i ricavi ricorrenti per mesi. Usa un flusso di cassa scontato se il capitale circolante è limitato.

## Licenza

MIT — fanne quello che vuoi.
