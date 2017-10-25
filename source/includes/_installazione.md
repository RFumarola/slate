# Installazione

## Prerequisiti
Prima di poter procedere all'installazione della piattaforma node.js è necessario preparare l'ambiente.

### Visual Studio Code

Aprire il seguente link, scaricare e installare l'ultima versione di Visual Studio Code.

`GET https://code.visualstudio.com/`

### Python 2.7

Aprire il seguente link, scaricare e installare Python 2.7.

`GET https://www.python.org/downloads/`

### Node.js

Per installare node.js, aprire il link seguente e scaricare l'ultima versione adatta al SO (Windows 64 bit). 

`GET https://nodejs.org/it/`

Una volta scaricato il pacchetto eseguirlo; durante l’installazione verrà chiesto se installare anche npm, accettare, completare l’installazione e riavviare Windows.
Una volta installato node, è necessario configurare le variabili di ambiente:

**User Variables**

Aggiungere nella viariabile PATH il percorso:

`C:\Users\r.fumarola\AppData\Roaming\npm`

**System Variables**

`Name: NODE_PATH`

`Value: %APPDATA%\npm\node_modules`

### Configurazione Proxy

<aside class="notice">
La configurazione del Proxy è necessaria solo se la piattaforma deve essere installata su una macchina della banca IMI.
</aside>

Aprire il prompt di Windows e digitare i seguenti comandi:

`npm config set proxy http://proxy.company.com:8080`

`npm config set https-proxy http://proxy.company.com:8080`

### Configurazione dei moduli node.js

Aprire il prompt di Windows e digitare i seguenti comandi:

`npm install -g express`

`npm install -g body-parser`

### Configurazione del modulo oracledb

Prima di installare il modulo oracle in node.js, bisogna installare Oracle Instant Client. 
Quindi, aprire l'URL seguente, scegliere la versione adatta di Oracle (12c o 11g) insieme alla versione del SO (Windows 64bit), accettare la licenza e scaricare i due pacchetti:

- Instant Client Package Basic
- Instant Client Package SDK

`GET http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html`

Estrarre gli zip. Andare in C: e creare la cartella Oracle. All'interno di essa creare la cartella instantclient. Quindi alla fine si avrà il path C:\Oracle\instantclient.
Copiare il contenuto degli zip Client Package Basic e Client Package SDK nella cartella instantclient.

A questo punto si devono configurare le variabili d'ambiente.

**User variables**

Nella variabile PATH, aggiungere all'inizio (non in coda a tutte le altre!) il path 

`C:\Oracle\instantclient`

**System variables**

Creare le due variabili d'ambiente seguenti:

- OCI_LIB_DIR

`C:\Oracle\instantclient\sdk\lib\msvc`

- OCI_INC_DIR

`C:\Oracle\instantclient\sdk\include`

Si può installare finalmente il module oracledb su node.js. Aprire il prompt di Windows e digitare il comando:

`npm install -g oracledb`

### Installazione della piattaforma di Christian e Andrea






