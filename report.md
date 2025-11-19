# Report

# Scoreboard

è stata trovata analizzando il codice main.js client-side, dove si vede che viene costruita nella sidebar. Si cerca nelle variabili il nome del path, che è 'score-board' e lo si compone seguendo il formato degli altri link in sidebar, ovvero 'http://localhost:3000/#/score-board'.

# XSS

## DOM XSS

Passato ```<iframe src="javascript:alert(`xss`)">``` nella barra di ricerca il che lancia codice JavaScript sul client.
Un utente malizioso potrebbe inviare un link contenente la query di ricerca con iframe per lanciare codice sul computer della vittima.

## Reflected XSS

Ho cercato sul sito delle pagine che nell'URL prendono un parametro. La pagina di tracking ha un parametro ?id= sul quale si può eseguire un attacco XSS passando il payload ```<iframe src="javascript:alert(`xss`)">```.

## API-Only XSS

Questa challenge richiedeva una attenta analisi degli endpoint API e l'utilizzo di strumenti esterni come Postman per fare richieste specifiche. Inviando una richiesta PUT su `/api/Products/1` otteniamo una risposta di successo, indicando che è possibile modificare il prodotto senza averne i permessi (*Broken Authorization*)! Ho quindi modificato la descrizione del prodotto, passando il payload ```<iframe src="javascript:alert(`xss`)">```, eseguendo quindi un attacco persistente XSS. 

## CSP Bypass
https://www.giac.org/paper/gwapt/8252/content-security-policy-bypass-exploiting-misconfigurations/171869

Per questa challenge occorre attaccare la pagina `/profile`. Inviare un semplice payload XSS dentro il campo username non basta: viene filtrato e la pagina ha una regola Content Security Policy (CSP). Il metodo per ovviarla è cambiare formare un payload nell'url dell imagine, aggiungendo le regole che vogliamo. Dopodiché si invia un payload XSS nell'username che evita i filtri per far funzionare lo script.

https://www.youtube.com/watch?v=oyDa862JKqI

```<<script><<<sscript>alert(xss)</script>```

# Sensitive Data Exposure

## NFT Takeover

Nel codice main.js ho trovato un routerLink alla pagina juicy-nft. Visitandola ho tentato alcune chiavi private arbitrarie per capire il funzionamento. Analizzando meglio il codice ho trovato alcune chiavi, di cui solo una quella pubblica dava un riscontro diverso. Spostando l'attenzione sulla pagina about-us, ho trovato un commento placeholder rimasto dal developer nel quale ha lasciato la sua chiave privata in forma di mnemonico. Convertendo il mnemonico in chiave privata sono riuscito a risolvere la challenge.

## Exposed Metrics

Prometheus è un servizio di monitoraggio dei servizi molto famoso. Esso viene lanciato sull'endpoint /metrics. Accedendo a questa pagina su Juice Shop si possono vedere le metriche che dovrebbero essere nascoste all'utente finale e accessibili solo agli admin.

## Confidential Document

Analizzando la pagina `/about` si è trovato il link a `/ftp/legal.md` che contiene informazioni legali. In questo vediamo il nome della sottocartella `ftp` che corrisponde al famoso *File Transfer Protocol*. Tentiamo quindi di accedere a `/ftp/` e visualizziamo infine la lista dei file che vengono involontariamente presentati all'utente. 

## Forgotten Sales Backup

Vedi [questo](#easter-egg)

## Forgotten Developer Backup

Vedi [questo](#easter-egg)

## Misplaced Signature File

[SIEM](https://en.wikipedia.org/wiki/Security_information_and_event_management).
Vedi [questo](#easter-egg)

# Injection

## Login Admin

Riferimento usato: https://portswigger.net/support/using-sql-injection-to-bypass-authentication


Tra le recensioni dei prodotti è possibile trovare la e-mail dell'amministratore. Utilizzandola nel portale login e inserendo il payload `admin@juice-sh.op' --'` e una qualsiasi password è possibile accedere al suo account. Un altro payload che può essere utilizzato è `' or true --'`.
- [ ] Usare il password hash dalla risposta login per accedere

## Login Bender

Similmente all'admin, è possibile accedere all'account di un altro (o qualsiasi) utente del sito, sapendo la sua e-mail. Tra le recensioni dei prodotti possiamo trovare l'email di Bender `bender@juice-sh.op` sulla quale faremo un attacco nella pagina login. Provando alcuni payload di e-mail e le risposte di errore, vediamo che il codice SQL fa una query `SELECT * FROM Users WHERE email = ''bender@juice-sh.op' AND password = '..' AND ...`. Si può quindi passare il payload `bender@juice-sh.op' --` per far ignorare le condizioni posteriori e selezionare l'utente. Passando un qualsiasi valore di password, si ha accesso all'account.

## Database Schema

Questa è stata una challenge un po' più difficile. Ho provato a ottenere lo schema dalla pagina login, ma non ho avuto successo. Mi sono spostato sulla barra di ricerca. Con ZAP ho tenuto traccia delle richieste quando si fa una ricerca e ho visto una chiamata a `/rest/products/search?q=`. Formando alcuni payload ho scoperto la tecnologia SQL che il server utilizza. Dopodiché ho cercato su internet come ottenere gli schema e ho inviato la richiesta. Ho dovuto infine formare la query con UNION per ottenere una risposta. Per far funzionare il comando, ho dovuto far combaciare il numero di colonne della query di ricerca con quello della union. Il payload finale è stato: `test')) UNION SELECT sql,2,3,4,5,6,7,8,9 FROM sqlite_master--`

# Broken Access Control

## View Basket

Andando sul proprio account si può visualizzare il carrello. Su DevTools, si vede che la pagina fa una richiesta a `/rest/basket/idUtente`, dove idUtente è un numero. Facendo banalmente una richiesta su un idUtente diverso dal nostro e passando l'Authorization header proprio è possibile visualizzare il carrello di un'altra persona.

## Admin section

Accedendo come amministratore facendo una SQL injection e analizzando il codice per trovare pagine nascoste, è stato possibile accedere a `/administration`

## Forged Feedback

Lasciando una recensione sul portale `/contact` si può visualizzare il funzionamento dell'api `/api/Feedbacks` con ZAP. In POST viene inviato il seguente JSON `{"UserId":23,"captchaId":7,"captcha":"49","comment":"12345 (***ting@gmail.com)","rating":3}`. Creando una richiesta con un UserId arbitrario si può inviare una recensione a nome di un altro utente. Inoltre, non vi è nessuna verifica sul autore della recensione che si vede nel commento, quindi è possibile creare uno scollegamento tra l'autore nel commento e l'userID

## Product Tampering

La challenge dice di modificare il `href` del prodotto 'OWASP SSL Advanced Forensic Tool (O-Saft)'. Sapendo che le operazioni di PUT non sono state bloccate per altre vulnerabilità, ho deciso di provare queste. Per prima cosa ho intercettato il formato JSON della risposta del prodotto e ho modificato l'attributo `href` secondo le indicazioni per la challenge. Poi ho cercato l'API dei prodotti: prima ho provato con `/Products/9` e poi con `/api/Products/9`. Sull'ultimo ho inviato richiesta PUT con Postman e sono riuscito a aggiornare la descrizione del prodotto.

## Easter Egg

Per questa challenge ho dovuto cercare un modo di accedere ai file. Dalla challenge [precedente](#confidential-document) abbiamo scoperto la presenza e visibilità della pagina `/ftp/`. Su questa troviamo una risorsa, chiamata `eastere.gg` che sembra essere quella necessaria per completare la missione. Purtroppo, non è possibile accedervi poiché il sito filtra oppure blocca estensioni diverse da `.md` e `.pdf`. Per ovviare questo, ho usato come riferimento [questa pagina](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion) e in particolare la modalità *Null byte injection*. Questa consiste nel aggiungere il payload `%00` in questo modo: `/ftp/eastere.gg%00.pdf`, il che però non funziona. Ritento allora un altro payload, `/ftp/eastere.gg%2500.pdf`. `%25` Corrisponde al carattere `%`, ma non ho trovato una reference del perché questo payload funziona. Comunque, il server restituisce un "pdf", che non è nient'altro che il nostro file `easter.egg` cambiato di estensione. Possiamo quindi rinominarlo e accedere direttamente al file per leggerlo, dopo averlo scaricato.

# Broken Anti Automation

## CAPTCHA Bypass

Sulla pagina Customer Feedback vediamo un CAPTCHA abbastanza semplice. Facendo una richiesta ordinaria possiamo vedere i parametri passati: ```{UserId: 1, captchaId: 7, captcha: "67", comment: "test1234 (***in@juice-sh.op)", rating: 2}```. E quindi possibile formulare nuove richieste utilizzando gli stessi parametri per ovviare il CAPTCHA. La pagina fa anche una richiesta all'endpoint `/rest/captcha` per cui è anche possibile ottenere immediatamente la risposta al CAPTCHA senza intervento umano. 

# Broken Authentication

## Reset Bender's Password

Raccogliendo informazioni su Bender dalle recensioni oppure dal data export ho scoperto che il suo nome è un riferimento a Bender di Futurma. Procedendo su 'Forgot Password' è possibile rispondere alla domanda di sicurezza con `Stop'n'Drop`, che consente il cambio della password.

# Security Misconfiguration

## Deprecated Interface

Cercando nel codice `main.js` ho trovato che l'interfaccia `/file-upload` accetta diversi tipi di estensioni, mentre nel frontend non è così (solo `.pdf` e `zip`). Questa interfaccia appartiene alla pagina `/complain` alla quale ho aggiunto un payload modificato chiamato `test.xml.zip`. Inviando il reclamo, ho intercettato e modificato la richiesta, eliminando la parte `.zip` ed effettivamente inviando `test.xml`, utilizzando l'interfaccia deprecata.

# XXE

## XXE Data Access

Per questa challenge ho dovuto utilizzare l'interfaccia deprecata di `/complain` e inviare un payload XML
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [<!ENTITY ent SYSTEM "C:\Windows\system.ini"> ]>
<complaint>
 <message>&ent;</message>
 <author>test</author>
</complaint>
```
Inviandolo però, non ho ricevuto la risposta attesa e riscontravo un errore di parsing o unexpected end of form forse dovuto ai permessi di Windows o misure di sicurezza Juice Shop.

# Improper Input Validation

## Upload Type

La pagina di interesse è `/complain`.  

## Deluxe Fraud

Ho testato la pagina `/deluxe` per la sottoscrizione del abbonamento utilizzando diversi metodi (carta, wallet). Ho notato che al server viene inviata la modalità di pagamento e dettagli come id della carta. Ho provato a modificare questi senza successo. Dopodiché ho considerato di utilizzare la modalità dei coupon e ho inviato una richiesta `{"paymentMode":"coupon", "balance": 10000, "paymentId":3}` con la quale ho riscontrato successo. 

# Unvalidated Redirects

## Allowlist Bypass

In precedenza avevo già individuato nella sidebar una pagina di redirect che porta a GitHub, sulla quale ho tentato di trovare vulnerabilità SSRF, senza successo. Trattandosi questa challenge di aggirare la lista di URL consentiti, ho provato alcuni payload `/redirect?to=https://github.com`, `/redirect?to=localhost:3000`... Utilizzando finalmente questo strano redirect `/redirect?to=http://localhost:3000/redirect?to=https://github.com/juice-shop/juice-shop` sono riuscito a completare la challenge, anche se non lo sapevo. La pagina mi reindirizzava su github, ma non capivo come potesse essere una vulnerabilità. Cercando su google ho scoperto che il payload malizioso può essere formato indirizzando a qualsiasi sito e assicurandosi di mettere il link github nei parametri, esempio: `redirect?to=https://example.com/?test=https://github.com/juice-shop/juice-shop`. Arrivo quindi alla conclusione che il codice backend verifica solo la presenza dell'url di GitHub.

# Vulnerable Components

## Legacy Typosquatting

Questa challenge è stata molto interessante, ma anche difficile. Per prima cosa ha requisito aver completato challenge priori come [Easter Egg](#easter-egg) e [Forgotten developer backup](#forgotten-developer-backup) che richiedeva il *Poison Null Byte*. Scaricando il file dei package da `/ftp/package.json.bak`, è possibile vedere le dipendenze. Dopodiché da questo [articolo](https://iamakulov.com/notes/npm-malicious-packages/) ho scoperto come alcuni package potrebbero essere maliziosi. Ho provato anche a scannerizare il `package.json` con [npscan.com](https://npmscan.com), fallendo però a trovare la vulnerabilità. Provando a cercare su NPM i package uno ad uno, si scopre che `epilogue-js` è un package malizioso, con nome simile al vero `epilogue`. Completo la challenge inviando il nome del package sulla pagina contatti. 