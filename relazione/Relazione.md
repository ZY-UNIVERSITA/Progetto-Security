# Relazione finale
## **Introduzione**
OWASP Juice Shop è un'applicazione web deliberatamente vulnerabile, progettata a fini didattici e formativi, che incorpora numerose debolezze comunemente riscontrate in contesti reali. Essa rappresenta un valido strumento per lo studio delle principali vulnerabilità elencate nella OWASP Top 10, fornendo un ambiente realistico e controllato per l’analisi della sicurezza applicativa.

Il progetto si avvale dell’immagine Docker ufficiale di OWASP Juice Shop, la quale consente di creare un ambiente isolato, facilmente replicabile e sicuro. Questa configurazione risulta particolarmente adatta per lo svolgimento di un’attività completa di Vulnerability Assessment and Penetration Testing (VAPT). 

Nel presente elaborato, verranno applicate in maniera pratica le quattro fasi fondamentali del penetration testing sull’ambiente Docker dell’applicazione, illustrando le metodologie adottate, gli strumenti impiegati e i risultati ottenuti. L’obiettivo è quello di comprendere in che modo potrebbero verificarsi attacchi in scenari reali e quali strategie di prevenzione e mitigazione potrebbero essere adottate, il tutto in un contesto controllato e a scopo formativo.

Per la realizzazione del progetto è stata utilizzata la distribuzione Kali Linux, in quanto rappresenta uno standard di riferimento nel campo della sicurezza informatica. Essa include una vasta gamma di strumenti preinstallati specificamente progettati per il penetration testing, quali Burp Suite, OWASP ZAP, NMap, tra gli altri. Questo permette a Kali Linux di essere una piattaforma ideale sia per l’apprendimento che per l’applicazione pratica in ambito accademico e professionale.

## Fasi del PT
Il Penetration Testing (PT) è una metodologia strutturata che consente di simulare attacchi reali su un sistema reale, con l’obiettivo di scoprire falle di sicurezza prima che possano essere sfruttate da attori malevoli. Questo processo si articola in 6 fasi principali: Pre-engagement, Information Gathering, Vulnerability Assessment, Exploitation e Post-Exploitation e Post-engagement, ciascuna con un ruolo specifico nel ciclo di valutazione della sicurezza. All'interno di questo progetto è stato eseguito un Penetration Testing che riguarda solo 4 fasi su 6 fasi totali in quanto non viene eseguito .

## Information Gathering
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

### 4. Ricerca manuale e analisi del sito web
Una parte dell'information gathering è stato effettuato in modo manuale, per idetificare punti d'attacco sfuggiti durante le scansioni automatici, sfruttando l'eventuale esperienza acquisita. Durante la navigazione manuale sono stati individuati:

- Link nascosti, come la cartella `ftp` accessibile tramite la pagina `about`.
- Enumerazione utenti tramite analisi di prodotti e pagine, identificando possibili account amministrativi.
- Stack tecnologico utilizzato (Angular, Node.js, MongoDB, ecc.) tramite l'analisi dei file JS e HTML.
- Nuove rotte scoperte analizzando i file JavaScript.
- Punti di input utente (form di login, ricerca, feedback, chatbot, ecc.).
- Credenziali hard-coded trovate nel codice sorgente.

### 5. Raccolta di informazioni dopo autenticazione
Successivamente è stato effettuato l'autenticazione che permette, comportandosi come un utente normale, l'accesso a nuovi punti di collegamento client-server che erano preclusi durante l'information gathering anonimo.

Durante questa fase sono state fatte, in collaborazione con Burp Suite:

1) Analisi delle pagine accessibili solo dopo login, come review, feedback, chatbot, complaint e carrello. 
2) Osservazione delle risposte del server contenenti dati sensibili, come password hashate.

### 6. Analisi dei path e file scoperti
- Classificazione delle pagine trovate in base al codice di risposta HTTP (200, 301, 500).
- Esplorazione di cartelle sensibili come `ftp`, `metrics`, `api-docs`, `.well-known` e `encryptionkey`.
- Individuazione di file di configurazione e chiavi potenzialmente critiche come ad esempio la JWT key dentro encryptionkey.
- Analisi del file `robots.txt` per individuare directory nascoste.

### 7. Ricognizione DNS e dominio
Tramite il comando Whois è stato fatto la raccolta di informazioni sul dominio, registrar, data di creazione e scadenza e contatti amministrativi

Tramite DNSRecon è stato possibile identificare informazioni DNS rilevanti come i name server (NS), mail server (MX), record A e AAAA (indirizzi IP).

### 8. Identificazione delle tecnologie con WhatWeb
Ha permesso il rilevamento di framework, librerie e configurazioni HTTP e le loro versioni associati dal quale sarà possibile effettuare scansioni per rilevare vulnerabilità non patchat e l'individuazione di potenziali configurazioni insicure, come CORS permissivi.
