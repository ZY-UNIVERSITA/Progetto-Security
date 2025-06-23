# **Vulnerability assessment**

## **1. Accesso alla cartella FTP**
### **Descrizione della vulnerabilità**
Il server espone una cartella accessibile via protocollo FTP senza richiedere autenticazione. Questo espone potenzialmente file sensibili o informazioni critiche a chiunque abbia accesso alla rete, violando i principi di confidenzialità. Per esempio, al suo interno di trovano file come:

* Package.json: che permette di avere una visione completa sull'architettura usata dal server.
* Acquisition.md: file confidenziale interno all'azienda.

### **Riproducibilità**
1. Aprire il browser
2. Connettersi all’indirizzo `http://127.0.0.1:3000/ftp`
3. Accedere liberamente ai file interni

### **Prova della rilevazione**
![Cartella FTP accessibile](../immagini/va/ftp.png)

### **Classificazione OWASP TOP 10**
* **A01 - Broken access control**: anche se la cartella è pubblicamente accessibile, il server non esegue nessun controllo sull'identità della persona e quindi dei suoi permessi che prova ad accedere ai file confidenziali.
* **A02 - Cryptographic failuer**: i file confidenziali non sono stati cifrati.
* **A04 - Insecure Design**: il server potrebbe essere stato progettato male, senza tenere in considerazione l'accesso alla cartella e ai suoi file.
* **A05 - Security misconfiguration**: il server potrebbe essere stato progettato bene ma l'implementazione e la sua configurazione sono state definite in modo superficiale ed errato.

### **Requisiti dell'attaccante**
Per l'accesso alla cartella e ad alcuni dei file è necessario solamente un terminale in grado di effettuare una richiesta al server per il path definito.

### **Gravità e Impatti**
La gravità è alta: dentro la cartella sono presenti chiaramente dei file di tipologia confidenziale e file riguardante la sicurezza dell'applicazione che, se risultassero accessibili a chiunque, comprometterebbero la sicurezza degli utenti.

* Il file `package` contiene dati riguardante l'architettura del sistema.
* Il file `incident-support` potrebbe contenere dati riguardante incidenti di sicurezza.
* Il file `suspicious_erros` potrebbe contenere dati riguardante attività sospette.
* Il file `encrypt e announcement` potrebbe contenere dati confidenziali protetti.   

### **Fix del Codice**
* Disabilitare l'accesso alla cartella FTP
* Imporre un controllo dei permessi d'accesso alla cartella e ai file
* Criptare tutti i file per i quali è richiesta la triade CIA

---

## **2. Accesso ai file diversi da .md e .pdf**