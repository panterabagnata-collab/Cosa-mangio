# Cosa mangio oggi

App web in un solo file (`index.html`) che suggerisce cosa mangiare giorno per giorno in deficit calorico, in base alla dispensa e a quello che è già stato mangiato. Pubblicata con GitHub Pages dal branch `main`: ogni push aggiorna il sito in circa un minuto.

Sito: https://panterabagnata-collab.github.io/Cosa-mangio/

## Come lavorare
- Stile "ponytail": la soluzione più semplice che funziona. Niente framework, niente build, niente dipendenze nuove se bastano poche righe. Un solo file.
- Interfaccia in italiano, solo tema scuro (token CSS in `:root`).
- Dopo ogni modifica: esegui il self-test, poi commit e push su `main` con messaggio in italiano.
- Il repository è pubblico: non committare dati personali (peso, misure, backup JSON esportati dall'app).

## Struttura di index.html
- CSS e HTML con 3 schede (Oggi, Dispensa, Andamento) fatte con radio + `:has()`, senza JS.
- Primo `<script>`: logica pura, senza DOM. `F` (alimenti, valori per 100 g, porzioni min/max), `R` (ricette con alternative `a|b`), `cook()`, `suggest()`, `trend()`, `advice()`, `guessBase()`, `parseOFF()`, `selfTest()`.
- Secondo `<script>`: interfaccia. Stato in `localStorage`, chiave `cosa-mangio-oggi`: `stock`, `skip`, `kcal`, `prot`, `goal`, `days{data:{mt,log}}`, `weights`, `products`. Backup esporta/importa JSON nella scheda Andamento.
- Porzioni: `cook(r,pf,cf,av,t,over)` adatta proteina e carboidrato al budget del pasto (kcal e proteine rimaste × quota del pasto) senza superare la dispensa; `over` (mappa nome→grammi, opzionale) forza un ingrediente fisso a un valore o lo esclude (0), usato dalle carte piatto interattive. Verdure a volontà (grammi 0 in `x`) restano sempre fisse.
- Carte piatto (in `suggHTML()`): se la ricetta ha alternative (`p`/`c` con `|`) mostra select per cambiare proteina/carboidrato; gli ingredienti fissi con grammi > 0 (olio, formaggio, sugo…) sono spuntabili e modificabili; ogni modifica ricalcola `cook()` al volo (stato in `cardOver`, per indice nella lista dei suggerimenti del pasto aperto) prima di "L'ho mangiato".
- Quote dei pasti: senza colazione 42/16/42 (pranzo/spuntino/cena), con colazione 25/32/12/31.
- Barcode: `@zxing/browser@0.2.1` (UMD da jsdelivr, caricato al primo uso) + API v2 di Open Food Facts. La fotocamera funziona solo su https: aperta da file (`file://`, `content://`) viene negata senza chiedere il permesso.

## Test
```
node -e "const s=require('fs').readFileSync('index.html','utf8');const a=s.indexOf('<script>')+8;require('fs').writeFileSync('/tmp/l.js',s.slice(a,s.indexOf('</script>',a)));console.log(require('/tmp/l.js').selfTest())"
```
Deve stampare `selftest ok`. Nel browser: aggiungi `#test` all'URL e guarda la console.

## Vincoli alimentari (non cambiarli senza chiedere)
- Solo gli alimenti presenti in `F`. Esclusi: legumi, frutta secca, avena, cereali da colazione, tonno, salmone, fiocchi di latte, frutta e verdura non in lista (verdure: carote, rucola, cuppettone; frutta: mela).
- Obiettivi predefiniti: 1.800 kcal e 150 g di proteine al giorno, +100 kcal nei giorni di Muay Thai. La colazione di solito si salta.
- Regole dell'andamento: verso i 75 kg proporre −100 kcal; media ferma (> −0,2 kg/settimana per due confronti settimanali) → −100; calo oltre 1 kg/settimana dopo 3 settimane → +100; obiettivo 70,3 kg → mantenimento.

## Da fare
- Idee rimandate: sincronizzazione tra dispositivi, ricette generate con IA, costruttore libero per categoria (proteine/carboidrati/grassi) per comporre un piatto da zero quando manca un macro specifico (es. "mi servono 80g di proteine in più").
