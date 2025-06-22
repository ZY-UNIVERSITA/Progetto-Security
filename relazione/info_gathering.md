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
