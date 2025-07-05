# **Relazione finale**
## **Introduzione**
OWASP Juice Shop è un'applicazione web deliberatamente vulnerabile, progettata a fini didattici e formativi, che incorpora numerose debolezze comunemente riscontrate in contesti reali. Essa rappresenta un valido strumento per lo studio delle principali vulnerabilità elencate nella OWASP Top 10, fornendo un ambiente realistico e controllato per l’analisi della sicurezza applicativa.

Il progetto si avvale dell’immagine Docker ufficiale di OWASP Juice Shop, la quale consente di creare un ambiente isolato, facilmente replicabile e sicuro. Questa configurazione risulta particolarmente adatta per lo svolgimento di un’attività completa di Vulnerability Assessment and Penetration Testing (VAPT). 

Nel presente elaborato, verranno applicate in maniera pratica le quattro fasi fondamentali del penetration testing sull’ambiente Docker dell’applicazione, illustrando le metodologie adottate, gli strumenti impiegati e i risultati ottenuti. L’obiettivo è quello di comprendere in che modo potrebbero verificarsi attacchi in scenari reali e quali strategie di prevenzione e mitigazione potrebbero essere adottate, il tutto in un contesto controllato e a scopo formativo.

Per la realizzazione del progetto è stata utilizzata la distribuzione Kali Linux, in quanto rappresenta uno standard di riferimento nel campo della sicurezza informatica. Essa include una vasta gamma di strumenti preinstallati specificamente progettati per il penetration testing, quali Burp Suite, OWASP ZAP, NMap, tra gli altri. Questo permette a Kali Linux di essere una piattaforma ideale sia per l’apprendimento che per l’applicazione pratica in ambito accademico e professionale.

## **Fasi del PT**
Il Penetration Testing (PT) è una metodologia strutturata che consente di simulare attacchi reali su un sistema reale, con l’obiettivo di scoprire falle di sicurezza prima che possano essere sfruttate da attori malevoli. Questo processo si articola in 6 fasi principali: Pre-engagement, Information Gathering, Vulnerability Assessment, Exploitation e Post-Exploitation e Post-engagement, ciascuna con un ruolo specifico nel ciclo di valutazione della sicurezza. All'interno di questo progetto è stato eseguito un Penetration Testing che riguarda solo 4 fasi su 6 fasi totali in quanto non viene eseguito .

## **Information Gathering**
La fase di **Information Gathering** rappresenta il primo passo fondamentale nel Penetration Testing, in cui si raccolgono tutte le informazioni utili sul target, che possono includere dati di configurazione, servizi attivi, host attivi, porte aperte. Queste informazioni sull'azienda e sull'architettura di dominio pubblico e i suoi dipendenti e altre caratteristiche rilevanti per orientare le successive attività di analisi e attacco. Nel caso dell’immagine Docker di OWASP Juice Shop, questa fase è stata svolta attraverso una serie di strumenti e tecniche, descritte di seguito in modo sequenziale.

### 1. Scansione dei servizi attivi con Nmap 
L'obiettivo è dentificare il servizio in ascolto sul container del server locale (la porta 3000 in particolare, dove è eseguito il container Docker) e determinarne la versione confrontando le risposte del server con un database di firme. La scansione ha confermato la presenza di un server web sulla porta 3000, anche se non è stato possibile identificare con precisione la versione del servizio, probabilmente a causa dell’astrazione fornita dal container.

### 2. Enumerazione delle directory con ffuf
L'obiettivo è scoprire directory e file nascosti o non documentati sul server web tramite brute forcing. Usando liste di parole ad uso comune dove ogni parola della wordlist in fasi successive, usando wordlist sempre più ampie. Ogni parola viene inviata come richiesta HTTP al server per verificare l’esistenza delle risorse. 

Il server rispondeva con codice 200 anche per path inesistenti, rendendo difficile distinguere le risorse valide. Per questo motivo la maggior parte delle risposte sono state filtrate, in particolare in base alla lunghezza di contenuto pari a 80117 byte, escludendo così i falsi positivi.

Questa scansione ha permesso di identificare directory effettivamente esistenti che verranno poi analizzate nella fase successiva per identificare potenziali punti d'attacco.

### 3. Analisi del traffico con Burp Suite
In supporto a ffuf, è stato usato Burp Suite che è stato utilizzato come proxy per intercettare e analizzare il traffico HTTP tra client e server.
Ha permesso di mappare l’architettura del sito, identificare endpoint, parametri e potenziali vulnerabilità.

Questo ha permesso di mappare tutti le directory direttamente visitabili e quelle per le quali le pagine web hanno eseguito richieste specifiche. Inoltre Burp Suite permette anche di intercettare le richieste in entrata e in uscita, con la possibilità, in modo semplificato anche tramite plugin, di modificarle per ottenere gli effetti voluti, per esempio cercando di bypassare controlli lato-client.

### 4. Ricognizione DNS e dominio
Tramite il comando Whois è stato fatto la raccolta di informazioni sul dominio, registrar, data di creazione e scadenza e contatti amministrativi

Tramite DNSRecon è stato possibile identificare informazioni DNS rilevanti come i name server (NS), mail server (MX), record A e AAAA (indirizzi IP).

### 5. Identificazione delle tecnologie con WhatWeb
Ha permesso il rilevamento di framework, librerie e configurazioni HTTP e le loro versioni associati dal quale sarà possibile effettuare scansioni per rilevare vulnerabilità non patchat e l'individuazione di potenziali configurazioni insicure, come CORS permissivi.

### 6. Ricerca manuale e analisi del sito web
Una parte dell'information gathering è stato effettuato in modo manuale, per idetificare punti d'attacco sfuggiti durante le scansioni automatici, sfruttando l'eventuale esperienza acquisita. Durante la navigazione manuale sono stati individuati:

- Link nascosti, come la cartella `ftp` accessibile tramite la pagina `about`.
- Enumerazione utenti tramite analisi di prodotti e pagine, identificando possibili account amministrativi.
- Stack tecnologico utilizzato (Angular, Node.js, MongoDB, ecc.) tramite l'analisi dei file JS e HTML.
- Nuove rotte scoperte analizzando i file JavaScript.
- Punti di input utente (form di login, ricerca, feedback, chatbot, ecc.).
- Credenziali hard-coded trovate nel codice sorgente.

### 7. Raccolta di informazioni dopo autenticazione
Successivamente è stato effettuato l'autenticazione che permette, comportandosi come un utente normale, l'accesso a nuovi punti di collegamento client-server che erano preclusi durante l'information gathering anonimo.

Durante questa fase sono state fatte, utilizzando in parallello Burp Suite:

1) Analisi delle pagine accessibili solo dopo login, come review, feedback, chatbot, complaint e carrello. 
2) Osservazione delle risposte del server contenenti dati sensibili, come password hashate.

### 8. Analisi dei path e file scoperti
- Classificazione delle pagine trovate in base al codice di risposta HTTP (200, 301, 500).
- Esplorazione di cartelle sensibili come `ftp`, `metrics`, `api-docs`, `.well-known` e `encryptionkey`.
- Individuazione di file di configurazione e chiavi potenzialmente critiche come ad esempio la JWT key dentro encryptionkey.
- Analisi del file `robots.txt` per individuare directory nascoste.

### Information gathering in dettaglio
Il capitolo rappresenta solo una riassunto dell'information gathering che è stato eseguito. Per informazioni dettagliate, riferirsi al file "**info_gathering.pdf**".

## **Vulnerability Assessment**
Segue la fase di **Vulnerability Assessment**, in cui si identificano e classificano le vulnerabilità presenti nel sistema a partire dalle informazioni raccolte dallo step precedente. Questa fase permette di costruire una mappa delle debolezze presenti all'interno del sistema. Si possono, poi in particolare, creare delle PoC (Proof of concept) che permettono di definire, dal punto di vista teorico, eventuali strategie d'attacco che potranno poi essere effettivamente sfruttate nella fase di exploitation.

### 1. Accesso alla cartella FTP
Dopo aver rilevato i framework tecnologici usati dal sitor web, si è cercato di analizzare tali versioni ma senza troppo successo. Successivamente si è passato all'analisi dei path accessibili dal sito web, ed è stata scoperta la cartella `FTP` che normalmente è nascosta e protetta dagli accessi indesiderati. 

È stato osservato che il server espone una cartella accessibile FTP senza richiedere autenticazione. Questo espone potenzialmente file sensibili o informazioni critiche a chiunque abbia accesso alla rete, violando i principi di confidenzialità. Al suo interno sono stati trovati file come `package.json` e `acquisition.md`, che forniscono per esempio una visione completa sull'architettura usata dal server (stack usato e le loro versioni) e informazioni confidenziali interne all'azienda.

### 2. Accesso ai file diversi da .md e .pdf tramite "Null Byte Injection"
Dopo aver effettuato l'accesso all'interno della path `FTP`, è stata fatta una ricognizione del path stesso che ha portato al rilevamento di file confidenziali. Si è provato ad accedere ai vari file presenti, ma il server presentava un controllo degli accessi ai file tramite estensione. 

Si è cercato di trovare un modo per bypassare questo blocco di sicurezza inducendo il server a pensare che si stia accedendo ad un file valido. È stato scoperto che inviando un input particolare costituito da una stringa che contiene `%2500`, il server legge il nome del file fino a incontrare il valore `%2500`, che viene decodificato nel valore `NULL` byte, terminando così la lettura del nome del file. Questo può permette dal punto di vista teorico l'accesso a file con estensioni diverse da `.md` e `.pdf`.

### 3. SQL injection tramite il login
Dopo aver constatato che il sito web permette di inviare al server degli input utente, in particolare nella pagina di `Login`, si è testata la possibilità che il server gestisca in modo sbagliato o non gestisca affatto gli input che gli arrivino. Inserendo un carattere speciale (es. ') nel campo username della pagina di login, l’applicazione ritorna un messaggio generico [object Object] e, nell’HTTP response body intercettata, compare parte della query SQL. 

Questo indica che i messaggi d’errore del database non sono gestiti correttamente, rivelando dettagli interni dell’implementazione SQL e potenzialmente facilitando un attacco di SQL Injection. Inoltre, si è scoperto in questo modo che il server non esegue nessun controllo sull'input sia lato frontend sia lato backend.

### 4. Cross-site scripting (XSS) nella barra di ricerca
Dopo aver constatato che il sito web permette di eseguire delle ricerche sui prodotti, la barra di ricerca è stata sottoposta ad indagine per capire se esegue una qualche operazione di sanificazione e validazione degli input, per evitare l'esecuzione di script malevoli indesiderati. 

È stata scoperta una vulnerabilità XSS presente nella funzione di ricerca dell'applicazione. Un utente malintenzionato può inserire codice JavaScript dannoso nel campo di ricerca, che viene poi eseguito nel browser della vittima senza adeguata sanitizzazione. 

Inoltre, l'applicazione web salva il valore del campo di ricerca direttamente nell'url del sito web, permettendo ad un attaccante di inviare un link malevolo all'utente che, se accedesse al sito tramite il link, eseguirebbe involontariamente lo script malevolo.

### 5. Accesso ai dati del carrello degli altri utenti
Analizzando il funzionamento dell’API utilizzata per recuperare i dati del carrello dell’utente, si è osservato che essa richiede esclusivamente il basket ID come parametro. Questo significa che, modificando manualmente il basket ID, è possibile tentare di accedere ai dati del carrello di altri utenti. 

È stato scoperto che l'applicazione consente di visualizzare i carrelli di altri utenti semplicemente modificando l'ID del carrello nella richiesta. Questa esposizione diretta di un identificatore senza un adeguato controllo degli accessi permette l’accesso non autorizzato a dati sensibili.

### 6. Utilizzo di un algoritmo di hashing vulnerabile per le password
Dopo aver osservato che, a seguito del logout, il server restituisce informazioni relative all’account utente, si è analizzata la struttura dei dati ricevuti per determinare l’algoritmo utilizzato per l’hashing delle password. 

È stato identificato l'uso dell'algoritmo MD5 per hashare le password utente senza l’aggiunta di un salt o di misure di protezione:

1) MD5 è un algoritmo obsoleto e vulnerabile ad attacchi basati su dizionario, rainbow table e brute-force. 
2) Inoltre, l’assenza di un salt rende possibile precomputare gli hash e riutilizzarli per attaccare facilmente più account che condividano la stessa password.

### 7. Account Takeover tramite Reset Password con Domanda di Sicurezza
L’indirizzo email è stato scoperto analizzando i commenti e le recensioni pubblicate sul sito web, dove è visibile in chiaro. Questa semplice esposizione rende possibile un primo contatto con l’account, potenzialmente sfruttabile in attacchi mirati. 

È stato scoperto che la funzionalità di reset password si basa su una security question legata all'utente. Tuttavia, le domande sono prevedibili, le risposte sono deboli, comuni o recuperabili da informazioni pubbliche, e non esiste un meccanismo di rate-limiting o CAPTCHA che impedisca un attacco brute-force. Un attaccante può facilmente ottenere il controllo completo dell’account bersaglio rispondendo correttamente a una domanda di sicurezza prevedibile.

### VA in dettaglio
Il capitolo rappresenta solo una riassunto del VA che è stato eseguito. Per informazioni dettagliate, riferirsi al file "**va.pdf**".

## **Exploitation**
La terza fase, **Exploitation**, a partire dalle vulnerabilità catalogate nello step precedente, è possibile esegure dei tentativi di sfruttamento delle vulnerabilità individuate, andando ad eseguire le PoC definite, per ottenere un accesso non autorizzato al sistema o ai dati oppure compromettere il sistema e l'architettura stessa. 

Questa fase è cruciale per verificare la reale pericolosità delle vulnerabilità rilevate e solitamente viene fatta in stretto accordo con l'azienda richiedente per evitare danni reali all'interi infrastruttura.

### Exploitation in dettaglio
Il capitolo rappresenta solo una riassunto dell'exploitation che è stato eseguito. Per informazioni dettagliate, riferirsi al file "**exploitation.pdf**".

## **Post-Exploitation**
Infine, la fase di **Post-Exploitation** si concentra sulle attività successive all’accesso ottenuto  al fine di valutare l’impatto potenziale di un attacco riuscito e la possibilità dalle nuove informazioni raccolte eseguire attachi più profondi e mirati. Le operazioni che solitamente vengono eseguite sono:

1) Il privilege escalation
2) Il lateral movement
3) Il persistence
4) Il detection evasion
5) La raccolta di ulteriori informazioni sensibili dall'interno del sistema (pillaging),

### Post-Exploitation in dettaglio
Il capitolo rappresenta solo una riassunto del post-exploitation che è stato eseguito. Per informazioni dettagliate, riferirsi al file "**post-exploitation.pdf**".
