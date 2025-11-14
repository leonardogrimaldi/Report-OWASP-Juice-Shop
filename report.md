# Report

# Scoreboard

è stata trovata analizzando il codice main.js client-side, dove si vede che viene costruita nella sidebar. Si cerca nelle variabili il nome del path, che è 'score-board' e lo si compone seguendo il formato degli altri link in sidebar, ovvero 'http://localhost:3000/#/score-board'.

# XSS

## DOM XSS

Passato ```<iframe src="javascript:alert(`xss`)">``` nella barra di ricerca il che lancia codice JavaScript sul client.
Un utente malizioso potrebbe inviare un link contenente la query di ricerca con iframe per lanciare codice sul computer della vittima.

## Reflected XSS

Ho cercato sul sito delle pagine che nell'URL prendono un parametro. La pagina di tracking ha un parametro ?id= sul quale si può eseguire un attacco XSS passando il payload ```<iframe src="javascript:alert(`xss`)">```.

# Sensitive Data Exposure

## NFT Takeover

Nel codice main.js ho trovato un routerLink alla pagina juicy-nft. Visitandola ho tentato alcune chiavi private arbitrarie per capire il funzionamento. Analizzando meglio il codice ho trovato alcune chiavi, di cui solo una quella pubblica dava un riscontro diverso. Spostando l'attenzione sulla pagina about-us, ho trovato un commento placeholder rimasto dal developer nel quale ha lasciato la sua chiave privata in forma di mnemonico. Convertendo il mnemonico in chiave privata sono riuscito a risolvere la challenge.

## Exposed Metrics

Prometheus è un servizio di monitoraggio dei servizi molto famoso. Esso viene lanciato sull'endpoint /metrics. Accedendo a questa pagina su Juice Shop si possono vedere le metriche che dovrebbero essere nascoste all'utente finale e accessibili solo agli admin.

## Confidential Document

Analizzando la pagina `/about` si è trovato il link a `/ftp/legal.md` che contiene informazioni legali. In questo vediamo il nome della sottocartella `ftp` che corrisponde al famoso *File Transfer Protocol*. Tentiamo quindi di accedere a `/ftp/` e visualizziamo infine la lista dei file che vengono involontariamente presentati all'utente. 

# Injection

## Login Admin

Riferimento usato: https://portswigger.net/support/using-sql-injection-to-bypass-authentication


Tra le recensioni dei prodotti è possibile trovare la e-mail dell'amministratore. Utilizzandola nel portale login e inserendo il payload `admin@juice-sh.op' --'` e una qualsiasi password è possibile accedere al suo account. Un altro payload che può essere utilizzato è `' or true --'`.
- [ ] Usare il password hash dalla risposta login per accedere

## Login Bender

Similmente all'admin, è possibile accedere all'account di un altro (o qualsiasi) utente del sito, sapendo la sua e-mail. Tra le recensioni dei prodotti possiamo trovare l'email di Bender `bender@juice-sh.op` sulla quale faremo un attacco nella pagina login. Provando alcuni payload di e-mail e le risposte di errore, vediamo che il codice SQL fa una query `SELECT * FROM Users WHERE email = ''bender@juice-sh.op' AND password = '..' AND ...`. Si può quindi passare il payload `bender@juice-sh.op' --` per far ignorare le condizioni posteriori e selezionare l'utente. Passando un qualsiasi valore di password, si ha accesso all'account.

# Broken Access Control

## View Basket

Andando sul proprio account si può visualizzare il carrello. Su DevTools, si vede che la pagina fa una richiesta a `/rest/basket/idUtente`, dove idUtente è un numero. Facendo banalmente una richiesta su un idUtente diverso dal nostro e passando l'Authorization header proprio è possibile visualizzare il carrello di un'altra persona.

## Admin section

Accedendo come amministratore facendo una SQL injection e analizzando il codice per trovare pagine nascoste, è stato possibile accedere a `/administration`

# Broken Anti Automation

## CAPTCHA Bypass

Sulla pagina Customer Feedback vediamo un CAPTCHA abbastanza semplice. Facendo una richiesta ordinaria possiamo vedere i parametri passati: ```{UserId: 1, captchaId: 7, captcha: "67", comment: "test1234 (***in@juice-sh.op)", rating: 2}```. E quindi possibile formulare nuove richieste utilizzando gli stessi parametri per ovviare il CAPTCHA. La pagina fa anche una richiesta all'endpoint `/rest/captcha` per cui è anche possibile ottenere immediatamente la risposta al CAPTCHA senza intervento umano. 

# Broken Authentication

## Reset Bender's Password

Raccogliendo informazioni su Bender dalle recensioni oppure dal data export ho scoperto che il suo nome è un riferimento a Bender di Futurma. Procedendo su 'Forgot Password' è possibile rispondere alla domanda di sicurezza con `Stop'n'Drop`, che consente il cambio della password.

