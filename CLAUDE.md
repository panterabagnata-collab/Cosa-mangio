# Cosa mangio oggi

App web in un solo file (`index.html`) che suggerisce cosa mangiare giorno per giorno in deficit calorico, in base alla dispensa e a quello che è già stato mangiato. Pubblicata con GitHub Pages dal branch `main`: ogni push aggiorna il sito in circa un minuto.

Sito: https://panterabagnata-collab.github.io/Cosa-mangio/

## Come lavorare
- Stile "ponytail": la soluzione più semplice che funziona. Niente framework, niente build, niente dipendenze nuove se bastano poche righe. Un solo file.
- Interfaccia in italiano, solo tema scuro (token CSS in `:root`).
- Dopo ogni modifica: esegui il self-test, poi commit e push su `main` con messaggio in italiano.
- Il repository è pubblico: non committare dati personali (peso, misure, backup JSON esportati dall'app, token di sincronizzazione).

## Struttura di index.html
- CSS e HTML con 3 schede (Oggi, Dispensa, Andamento) fatte con radio + `:has()`, senza JS.
- Primo `<script>`: logica pura, senza DOM. `F` (alimenti, valori per 100 g, porzioni min/max), `R` (ricette con alternative `a|b`), `cook()`, `suggest()`, `trend()`, `advice()`, `guessBase()`, `parseOFF()`, `selfTest()`.
- Secondo `<script>`: interfaccia. Stato in `localStorage`, chiave `cosa-mangio-oggi`: `stock`, `skip`, `kcal`, `prot`, `goal`, `days{data:{mt,log}}`, `weights`, `products`, `t` (timestamp ultimo salvataggio, per la sincronizzazione). Backup esporta/importa JSON nella scheda Andamento (non include il token di sync, che sta in una chiave `localStorage` separata `cosa-mangio-oggi-sync`).
- Sincronizzazione (scheda Andamento): un gist segreto sul GitHub dell'utente fa da archivio condiviso. Serve un Personal Access Token *classic* con scope `gist` (i token fine-grained non danno accesso ai gist), incollato una volta per dispositivo. `save()` scrive su `localStorage` e poi propone un push (debounce 1,5s); `syncPull()` gira all'avvio e quando la scheda torna visibile, e sovrascrive lo stato locale solo se il gist ha un `t` più recente (ultimo salvataggio vince, nessun merge).
- Porzioni: `cook(r,pf,cf,av,t,over)` adatta proteina e carboidrato al budget del pasto (kcal e proteine rimaste × quota del pasto) senza superare la dispensa; `over` (mappa nome→grammi, opzionale) forza un ingrediente fisso a un valore o lo esclude (0), usato dalle carte piatto interattive. Verdure a volontà (grammi 0 in `x`) restano sempre fisse.
- Carte piatto (in `suggHTML()`): se la ricetta ha alternative (`p`/`c` con `|`) mostra select per cambiare proteina/carboidrato; gli ingredienti fissi con grammi > 0 (olio, formaggio, sugo…) sono spuntabili e modificabili; ogni modifica ricalcola `cook()` al volo (stato in `cardOver`, per indice nella lista dei suggerimenti del pasto aperto) prima di "L'ho mangiato".
- Assistente AI (scheda Oggi, sotto ai piatti fissi; chiave in Andamento): chiama Gemini (`AI_MODEL`, endpoint `generateContent` con `responseSchema`) per un'idea di piatto oltre alle ricette fisse, con macro **stimate dall'AI** (badge "~ stima AI", mai spacciate per esatte) mostrate per ingrediente e in totale. Stato in `aiDish{meal: piatto|'loading'|{error}}`, non persistito. Ogni ingrediente ha una "×" che lo aggiunge a `S.dislikes.foods` e rigenera il piatto (sostituzione o piatto nuovo, decide l'AI); "Non mi piace questo piatto" fa lo stesso su `S.dislikes.dishes`. La chiave Gemini sta in `localStorage` separato (`cosa-mangio-oggi-ai`), mai nel backup né nel repository. `move()` ignora gli ingredienti AI non presenti in `F` (non hanno una voce in dispensa da scalare).
- Quote dei pasti: senza colazione 42/16/42 (pranzo/spuntino/cena), con colazione 25/32/12/31.
- Barcode: `@zxing/browser@0.2.1` (UMD da jsdelivr, caricato al primo uso) + API v2 di Open Food Facts. La fotocamera funziona solo su https: aperta da file (`file://`, `content://`) viene negata senza chiedere il permesso.

## Test
```
node -e "const s=require('fs').readFileSync('index.html','utf8');const a=s.indexOf('<script>')+8;require('fs').writeFileSync('/tmp/l.js',s.slice(a,s.indexOf('</script>',a)));console.log(require('/tmp/l.js').selfTest())"
```
Deve stampare `selftest ok`. Nel browser: aggiungi `#test` all'URL e guarda la console.

## Vincoli alimentari (non cambiarli senza chiedere)
- Ricette fisse (`R`) e dispensa: solo gli alimenti presenti in `F`. L'assistente AI fa eccezione ed è libero di proporre ingredienti fuori da `F` (confermato dall'utente): in quel caso le macro sono stimate dall'AI, non esatte.
- Esclusi sempre, anche per l'AI (`AI_EXCLUDED`): legumi, frutta secca, avena, cereali da colazione, tonno, salmone, fiocchi di latte, frutta e verdura non in lista (verdure: carote, rucola, cuppettone; frutta: mela).
- Obiettivi predefiniti: 1.800 kcal e 150 g di proteine al giorno, +100 kcal nei giorni di Muay Thai. La colazione di solito si salta.
- Regole dell'andamento: verso i 75 kg proporre −100 kcal; media ferma (> −0,2 kg/settimana per due confronti settimanali) → −100; calo oltre 1 kg/settimana dopo 3 settimane → +100; obiettivo 70,3 kg → mantenimento.

## Da fare
- Roadmap in corso (ordine confermato dall'utente): 1) motore AI + blacklist (fatto), 2) "Piano" settimanale che sostituisce la scheda Oggi (lista verticale: oggi espanso con anello/log, giorni futuri compatti; toggle dispensa-vincolata vs tutto-quello-che-mi-piace; generazione di un giorno alla volta per non sprecare la quota Gemini), 3) homepage di idee AI scorrevole non legate a un giorno preciso, 4) restyling visivo finale (emoji per categoria, badge, più contrasto, select proteina/carboidrato come chip invece di `<select>` nativi).
- Idee rimandate: costruttore libero per categoria (proteine/carboidrati/grassi) per comporre un piatto da zero quando manca un macro specifico, merge dei conflitti di sincronizzazione (oggi è "ultimo salvataggio vince"), ricerca web reale per la homepage (per ora idee generate dall'AI, non ricette vere trovate online).
