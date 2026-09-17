---
name: frontend-statico
description: ATTIVARE ESCLUSIVAMENTE SU RICHIESTA ESPLICITA DELL'UTENTE. NON ATTIVARE MAI AUTOMATICAMENTE. La skill deve essere utilizzata solo quando l'utente chiede esplicitamente di usare questa skill, oppure la identifica chiaramente con un riferimento come "frontend-statico", "usa la mia skill per il frontend", "usa la skill che ho creato per i siti", "usa la mia skill delle pagine", "usa la skill del frontend", "usa quella skill per il front end" o formulazioni equivalenti. Il riferimento può essere abbreviato, storpiato o espresso in inglese purché sia inequivocabilmente riferito a questa specifica skill. Non attivarla mai in base al contenuto della richiesta: parlare di frontend, HTML, CSS, Tailwind, pagine web, landing page, dashboard, portfolio, design o ServiceNow NON costituisce una richiesta di utilizzo della skill. Se l'utente chiede una pagina o un lavoro frontend senza riferirsi a questa skill, rispondere normalmente senza utilizzare questa skill. Contenuto: pipeline per generare pagine statiche autonome in HTML + Tailwind compilato e inlinato con font da npm e verifica visiva Playwright, metodo per esplorare direzioni estetiche tramite cataloghi di varianti votate, e archivio degli stili gia valutati.
---

# Frontend statico

## 1. Regole tecniche non negoziabili

1. Un solo file HTML autonomo. Nessun framework JS.
2. Tailwind compilato in locale e inlinato, mai da CDN. Il CDN `cdn.tailwindcss.com` non è raggiungibile dal container: una pagina che ci dipende si vede nuda negli screenshot di verifica.
   ```
   npm i -D tailwindcss@3.4.1
   # tema custom in tailwind.config.js, content puntato al file HTML
   npx tailwindcss -i in.css -o out.css --minify
   # sostituire un segnaposto <!--TWSTYLE--> nell'head con <style>…</style>
   ```
3. I font vanno presi da npm, non da Google Fonts. `fonts.googleapis.com` è bloccato nel container; il registry npm no.
   ```
   npm i @fontsource/<nome-font>
   # i .woff2 stanno in node_modules/@fontsource/<font>/files/
   # prendere solo i latin-<peso>-normal.woff2 dei pesi effettivamente usati
   # convertirli in base64 e generare @font-face inline
   ```
   Così il file funziona offline, è identico ovunque, e lo screenshot di verifica mostra i font veri invece dei fallback di sistema.
4. Verifica visiva obbligatoria prima di consegnare. Consegnare senza aver guardato è la causa principale dei risultati scadenti.
   ```
   python3 -m playwright install chromium   # già presente in /opt/pw-browsers
   # screenshot con device_scale_factor=2; per un catalogo, screenshottare sezione per sezione
   ```
   Guardare ogni schermata, non solo generarla. Controllare in particolare: testo che esce dal contenitore, titoli troppo grandi, pagine con grandi vuoti in basso.
5. Nessun commento nel codice se non richiesto.
6. Accessibilità di base: focus visibile, `prefers-reduced-motion` rispettato, contrasto sufficiente, responsive fino a mobile. Titoli grandi con `clamp()` o classi responsive, mai dimensioni fisse che spaccano su schermi stretti.
7. Non usare `web-artifacts-builder`: produce React compilato e minificato, output illeggibile. Inadatta alle pagine statiche.

## 2. Metodo: come si trova la direzione giusta

**Comportamento predefinito: fai la pagina, non il catalogo.** Scegli lo stile in base al contenuto, usando la sezione 3 come archivio di riferimento. I voti dicono cosa è piaciuto finora, non quale stile usare: sono un input alla scelta, non una classifica da scorrere dall'alto. Uno stile da 7 adatto al contenuto batte il 9 usato fuori contesto. Dichiara in una riga quale stile hai scelto e perché.

Il ventaglio di varianti si produce solo in due casi: se viene chiesto esplicitamente, oppure se il contenuto non rientra in nessuno degli stili in archivio. Non è il modo normale di fare una pagina: è la fase di esplorazione del gusto.

Orientamento contenuto → stile, ricavato dai voti:

| Contenuto | Stili da preferire |
|---|---|
| Strumenti tecnici, indici di progetti, dashboard, documentazione | terminale ciano, blueprint, vetro scuro, HUD sci-fi |
| Portfolio, landing di presentazione, pagine personali | vetro scuro, vetro scuro / lime, cyber neon, art déco, notturno giapponese |
| Raccolte eterogenee, pagine con molte voci da distinguere | bento pastello, vetro scuro bento, aurora bento |
| Pagine leggere, informative, da leggere di giorno | aurora chiara, aurora bento, gradiente caldo, acquerello/organico |

Non sono vincoli: se il contenuto suggerisce altro, si sceglie altro e si dice perché.

## 2b. Il ventaglio, quando serve

Quando si esplora una direzione nuova:

1. **Genera un ventaglio, non una proposta.** Stessa pagina, 10-14 estetiche nettamente diverse, in un unico file catalogo.
2. **Varia davvero.** Non solo la palette: font (sans / serif / mono / display), bordi (spessi / sottili / assenti), raggi (vivi / morbidi / pillola), ombre (dure / diffuse / nessuna), layout (lista / griglia / riquadri / colonne), fondo (scuro / chiaro / colorato / texture).
3. **Mantieni il contenuto comparabile.** Nel catalogo, contenuti, gerarchia informativa e quantità di elementi devono rimanere il più possibile identici tra le varianti. Devono cambiare principalmente linguaggio visivo, tipografia, palette, superfici, bordi, ombre e trattamento del layout.
4. **Screenshot di ogni variante e revisione**, prima di consegnare il catalogo.
5. **Chiedi voti da 1 a 10** su ogni variante.
6. **Al giro dopo, muoviti su direzioni nuove.** Due strade, entrambe valide: estetiche mai toccate, oppure ricombinazioni di singole caratteristiche che hanno funzionato (una palette presa da uno stile, un trattamento tipografico da un altro, un layout da un terzo) — non fusioni di stili interi, che spesso non sono compatibili. Errore da non ripetere: dopo un giro di voti, produrre automaticamente varianti minime dello stile vincente. Il ventaglio serve a mappare il gusto, non a perfezionare una pagina. Le rifiniture si fanno solo quando lo stile è stato scelto per una pagina vera.
7. **Registra i voti** nella sezione 3, come dati. Non trasformarli in una regola fissa.
8. **Verifica anche il mobile.** La verifica visiva deve comprendere almeno una viewport desktop e una mobile. Una variante non è considerata verificata se funziona solo a desktop.

## 3. Archivio gusti

Voti raccolti, dal più alto. Fotografia di quello che si sa oggi, da aggiornare a ogni giro. Le osservazioni specifiche sulle singole varianti vanno mantenute separate dalle deduzioni generali sul gusto.

| Stile | Voto | Note |
|---|---|---|
| Terminale ciano (mono, scanline, bordi netti, ciano su blu-nero) | 9 | il più alto finora |
| Art déco (cornice dorata, Cinzel, righe sottili) | 9 | semplice ed elegante |
| Notturno giapponese (blu notte, Shippori Mincho, luna) | 8.5-9 | |
| Vetro scuro / lime (bagliori verde-teal, pannelli sfocati) | 8.5 | |
| HUD sci-fi (verde fosforescente, Orbitron, griglia tecnica) | 8.5 | molto bella, la resa dipende dal tipo di sito |
| Vetro scuro (bagliori blu-viola, pannelli sfocati, angoli larghi) | 8 | |
| Cyber neon (verde acido e rosa, glow, angoli tagliati) | 8 | il taglio sfalsato però non convince |
| Bento pastello (riquadri colorati, angoli morbidi, niente bordi) | 8 | |
| Aurora bento (bagliori pastello + riquadri in vetro chiaro) | 8 | |
| Vetro scuro bento (la stessa palette riorganizzata a riquadri) | 8 | |
| Blueprint (griglia tecnica, ciano, squadrato) | 7.5 | |
| Aurora chiara (bianco con bagliori pastello, vetro chiaro) | 7.5 | il dark glass ribaltato funziona |
| Acquerello / organico (Caveat, DM Serif, macchie pastello sfumate) | 7.5 | elegante, classico |
| Isometrico (Space Grotesk, griglia a rombi, ombre offset colorate) | 7.5 | |
| Gradiente caldo (sfumatura satura, pannello bianco arrotondato) | 7 | |
| Soft UI / neumorfismo (grigio, ombre morbide, niente bordi) | 7 | |
| Y2K chrome (viola e ciano, gradienti metallici, bolle) | 7 | |
| Clay (viola pieno, forme gonfie 3D) | 7 | |
| Bento saturo (colori pieni su carta chiara) | 7 | |
| Materiali fisici / legno (venature, rivetti, DM Serif) | 7 | |
| Vetro scuro / magenta | 7 | il magenta piace meno del blu e del verde |
| Fumetto (Permanent Marker, sfondo giallo puntinato, card sagomate) | 6.5 | |
| Retro-futurismo anni 70 (Righteous, arance/marroni, archi a pillola) | 6.5 | font e colori piacciono, la resa (forma ad arco) meno |
| Bento scuro (riquadri colorati su fondo notte) | 6.5 | |
| Terminale ambra (stessa struttura, fosforo ambra) | 6 | struttura sì, colori no |
| Terminale viola | 6 | |
| Stampa risografica (Archivo Black, cerchi arancio/blu sovrapposti) | 6 | i colori penalizzano |
| Notturno oro (nero, serif alto, filetti oro) | 6 | |
| Collage / fanzine (Archivo Black, evidenziatore giallo, Space Mono) | 5.5 | |
| Memphis anni 80 (Righteous, pastello, forme geometriche sparse) | 5.5 | |
| Brutalist (bordi spessi, ombre dure, giallo acido) | 5 | |
| Carta calda (panna, serif, terracotta) | 4 | |
| Rivista (serif display enorme, filetti, rosso su bianco) | 4 | |
| Museo e galleria (DM Serif, minimale, filetti grigi) | non votato | percepito troppo simile ad altri stili già in archivio |
| Mono caldo (beige, tutto mono, nessun bordo né ombra) | scartato | troppo spoglio |
| Editoriale svizzero (griglia, filetti, bianco) | 2 | il più basso |

Cosa se ne ricava, per ora:

- Finora ottengono i voti più alti profondità, colore e materia. Bagliori, vetro, glow, ombre generose.
- L'eleganza classica (art déco, notturno giapponese, acquerello) regge molto bene quando è pulita e non sovraccarica: entra nella fascia alta insieme agli stili "materici".
- Il monospace funziona benissimo, sia come stile intero (terminale) sia come dettaglio dentro altri stili (numeri, percorsi tipo `~/progetti`, etichette).
- Fondo scuro forte, ma non è una regola: le versioni chiare con bagliori e riquadri colorati prendono voti alti quanto le scure. Quello che perde non è il chiaro, è il **piatto**.
- Font e palette possono piacere anche quando la resa complessiva convince meno: il retro-futurismo anni 70 ne è un esempio, penalizzato più dalla forma (archi a pillola) che dai colori.
- Le combinazioni "giocose" con più colori vivaci contemporaneamente (Memphis, collage/fanzine) restano nella fascia medio-bassa nonostante la personalità.
- Perde il tipografico puro: editoriale, rivista, serif su carta. Nessun voto sopra 4.
- Perde il minimalismo senza profondità: viene percepito come scadente o intercambiabile con altri stili (vedi museo e galleria, non votato per questo motivo).
- Il brutalist non convince nonostante la forte personalità.

## 4. Direzioni ancora da esplorare

Da usare per i prossimi ventagli, invece di rifinire quelle già viste: HUD sci-fi in versione portfolio/prodotto (non solo dashboard tecnica), notturno giapponese applicato a contenuti diversi dal testo puro, art déco in chiave più scura/contrastata, retro-futurismo anni 70 con layout diverso dagli archi (es. onde, strisce), acquerello con palette più satura, isometrico con più profondità 3D, museo/galleria ripensato per differenziarlo davvero dagli altri stili chiari minimali, e le direzioni mai ancora provate: HUD sci-fi già coperto, quindi restano da questa lista originale: fumetto in variante più matura, materiali fisici con metallo o tessuto invece del solo legno.
