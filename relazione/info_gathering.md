# Information Gathering

## 1. nmap
### **Comando completo**

```sh
nmap -sV 127.0.0.1 -p 3000
```

### **Spiegazione**
*   **Motivo dell'utilizzo:** Identificare il servizio e la sua versione in ascolto sulla porta specificata.

*   **Obiettivo della scansione:** Determinare quale applicazione (es. server web) e la sua versione è in esecuzione sulla porta 3000 del server locale (127.0.0.1). Sarebbe ovviamente possibile, eseguire una mappatura sull'IP e su tutte le porte possibili collegate all'IP stesso.

*   **Spiegazione del funzionamento:** Il comando `nmap` con il flag `-sV` (Service Version detection) invia una richiesta alla porta 3000 per analizzare le risposte del servizio e confrontarle con il suo database di firme, riuscendo così a identificare il tipo di servizio e la sua versione. In questo caso, la scansione è mirata specificamente all'indirizzo localhost e alla porta 3000.

### **Risultato della scansione**
![Risultati della scansione Nmap](../risultati/nmap.png)

### **Analisi della scansione**
nmap non è stato in grado di identificare correttamente il nome del servizio e la sua versione, probabilmente perchè il container nasconde l'infrastruttura che ci sta dietro.

---

## 2. ffuf
### **Comando completo (prima scansione)**

```sh
ffuf -w /usr/share/wordlists/dirb/small.txt -u http://127.0.0.1:3000/FUZZ -t 5
```

### **Spiegazione**
*   **Motivo dell'utilizzo:** Eseguire un'enumerazione di directory e file (directory brute-forcing) per scoprire risorse nascoste sul server web.

*   **Obiettivo della scansione:** Trovare percorsi web validi (come `/admin`, `/login`, `/css`, ecc.) sul server in esecuzione all'indirizzo `http://127.0.0.1:3000` utilizzando una wordlist fornita dall'utente.

*   **Spiegazione del funzionamento:** `ffuf` prende ogni parola dalla wordlist specificata (in questo caso `small.txt`) e la sostituisce al placeholder `FUZZ` nell'URL. Invia una richiesta HTTP per ogni URL generato e ne analizza la risposta. Il flag `-t 5` imposta il numero di thread a 5 per ridurre il numero di richieste contemporanee per non sovracaricare il server e cercare di ridurre eventuali blocchi anti-bot.

### **Risultato della scansione**
![Risultati della prima scansione Ffuf](../risultati/ffuf1.png)

### **Analisi della scansione**
ffuf è stato in grado di mappare i path del sito web. A quanto pare il sito web gestisce qualsiasi path non presente con status `200` e payload di lunghezza `80117`.
