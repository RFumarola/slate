# Descrizione generale della piattaforma

Per modificare ed estendere le funzionalità del chatbot, aprire il progetto con Visual Studio Code.
La struttura del progetto si articola nelle seguenti folder principali:

- .vscode
- Chatbot
  - config
  - core
- lib
  - Database
  - NLU

![Folder progetto](/images/conversational_folder.PNG)

La piattaforma conversational è in grado di inviare/ricevere messaggi attraverso 3 canali: **Echo**, **Facebook** e **Mobile**. Il canale implementato per il chatbot IMI è solo quello mobile.

Di seguito saranno analizzate le cartelle e tutti i relativi script che dovranno essere modificati per integrare il chatbot con le query da effettuare sul db e con eventuali nuovi stati.

Sarà anche descritto il formato del messaggio da inviare/ricevere attraverso questa piattaforma.

## Formato del messaggio da inviare/ricevere

>Formato del messaggio

```javascript--node

{
	"from": {
		"id": "hashedUserID"
	},
	"type": "message",
	"message": "Write the message here",
	"channel": "MOBILEAPP"
}
```

### Campi del messaggio

Campi | Descrizione
--------- | -----------
id | Hashed ID dell'utente della banca o del bot
type | Tipo del messaggio inviato/ricevuto
message | Contenuto del messaggio inviato/ricevuto
channel | Tipo di canale su cui il bot è in ascolto

Gli unici campi variabili sono id e message. Gli altri campi per il momento non devono essere modificati.

## .vscode

>launch.json

```javascript--node

"configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Avvia programma",
            "program": "${workspaceRoot}\\Chatbot\\main.js",
            "env": {
                "useExpress": "true",
                "ENVIRONMENT": "CERT",
                "WEB_SERVICE_BASE_URL": "https://3994b48e.ngrok.io",
                "FB_VERIFY_TOKEN": "one_two_three",
                "FB_PAGE_TOKEN": "EAAFVn9ydGLwBAMifi5rU1xRZCo9sciIgBnqMJkKOXby7ZC3iPOQKfrZC5wOmbRPu7fQMum71ZBidvZB2JZC7aAsY9jujD4C6TKxAd6a5jyiu3IzkmudokqgE0bzCS28GEolrGl63xlmz8PFZABZBXHZALyOtCZBcZCZAsGUTWmYzQqfg7MzeL4upRt8H",
                "APIAI_ENDPOINT": "https://api.dialogflow.com/v1/",
                "APIAI_APIVERSION": "20150601",
                "APIAI_AUTHTOKEN": "414629b7b56345edbf9067516fe74b86",
                "APIAI_LANG": "en",
                "APIAI_TIMEZONE": "Europe/Madrid"
            }
        }   
	]   
```

In questa cartella lo script principale è **launch.json**.

Questo script contiene tutte le configurazioni per l'avvio del bot. 

### Parametri di configurazione

Parametri | Descrizione
--------- | -----------
program | Il main del progetto da eseguire all'avvio.
env     | Tutte le variabili d'ambiente (access token) per l'integrazione dei servizi esterni, come Dialogflow e Facebook Messenger.

L'API Dialogflow prevede 2 tipi di access token: client e developer. Per questa piattaforma è richiesto il developer access token.

## Chatbot

Questo è il cuore della piattaforma. All'interno di questa cartella, il progetto è strutturato nel seguento modo:

- core
  - AppBusinessLogic
  - Connectors
  - Dialogs
- main.js

### AppBusinessLogic

> AppLogic.js

```javascript--node

var OracleDBClient = require("../../../lib/Database/OracleDBClient.js");

class AppLogic {
    constructor() {
    }

    // Utility functions for IMI Chatbot
    nluResponse(context){
        return context.userContext.nluInfo.speech
    }

    isActionIncomplete(context){
        return context.userContext.nluInfo.actionIncomplete
    }

    getIntentName(context){
        return context.userContext.nluInfo.intent
    }

    /**
     * Example of method for performing a query in OracleDb
     * @param {*} context 
     */
    getCurvesAndLevels(context){  
        return new Promise((resolve, reject) => {
            /*
            TODO: Create a real function with a real query for IMI, in OracleDBClient.js module.
            Example: call a function that take the Counterpart entity as input.
                OracleDBClient.getInfo(context.userContext.nluInfo.entities.counterpart)   
            Note: The response of Dialogflow is saved in context.userContext.nluInfo.        
            */
            OracleDBClient.getCompanies("ReplyML").then(
                function(result){
                    if(result!=null){
                        //TODO: some manipulation of the result -> we have to send a message with the results to the user. (String)
                        resolve(result["DESCRIPTION"])
                    } else{
                        resolve(oracleNoResultFound(context))
                    }  
                }
            )
            .catch((err) => {
                reject(err)
            })
        })
    }

    oracleNoResultFound(context){
        return "Sorry, but I din't found anything in Resource."
    }


    generalPromise(value) {
        return new Promise((resolve, reject) => {
            resolve(value);
        })
    }

    substitutePlaceholders(context, string) {
        Object.keys(context.bag).forEach((key) => {
            string = string.replace(new RegExp(key, 'gi'), context.bag[key])
        })
        return string;
    }
	
    ...
}

module.exports = AppLogic;

```

Questa classe è utilizzata dal bot per tutti quei metodi che non riguardano la gestione del dialogo e degli stati del bot. 

In questo script devono essere aggiunte tutte le funzioni utilities per:

- interrogare il db Oracle
- inviare un messaggio custom in determinate circostanze
- manipolare input/output durante la conversazione

<aside class="success">
Ricorda — Non scrivere mai questo tipo di funzioni all'interno degli script nella folder Connectors. Tieni il codice pulito e separato!
</aside>

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

### Connectors

In questa cartella sono implementati i 3 connettori usati dalla piattaforma nella sua versione originale. Per questo bot noi usiamo soltanto il connettore Mobile. 
Nel caso in cui si volesse modificare il formato del messaggio, è necessario modificare lo script **Mobile.js** (all'interno della sottocartella Channel) per interpretare i nuovi campi.
Le modifiche devono essere apportate anche nel ***main.js**.

### Dialogs

> Mobile.js

```javascript--node

var dialogs = {}

/**
 * Initial state of the bot
 */
dialogs['/'] = [
    function (engine, context, appLogic) {
        engine.replaceDialog('/start');
    }
]

/**
 * Second state of the bot. When a user sends a message to the platform, 
 * the bot fowards it to DialogFlow in order to get the response.
 * - if the request contains all the entities required for the specific intent, then the bot moves to the next state;
 * - otherwise, send to the user the response from Dialogflow
 */
dialogs['/start'] = [
    function (engine, context, appLogic) {
        if (appLogic.getIntentName(context) != "Default Fallback Intent"){
            if(appLogic.isActionIncomplete(context)){
                engine.send(appLogic.nluResponse(context));
            } else {
                engine.replaceDialog("/connectToOracleDb",0)
            }    
        } else{
            engine.send(appLogic.nluResponse(context));
        }           
    }
]

/**
 * Third state of the bot.
 * Perform a query to Oracle DB with all the entities retrieved by Dialogflow for a specific intent.
 */
dialogs['/connectToOracleDb'] = [
    function (engine, context, appLogic){
        // Switch on Dialogflow intent names
        switch(appLogic.getIntentName(context)){
            //Response of Dialogflow is saved in context.userContext.nluInfo.entities
            case "getCurvesAndLevels":
                appLogic.getCurvesAndLevels(context)
                .then((res) => {
                    //Send a message to the user with all the informations retrieved in db.
                    engine.send(res);
                })           
                break;                
            case "getMasterAgreement":
            case "getNettingApplicable":
            case "getPricing":
        }
    },
    function (engine, context, appLogic) {
        engine.replaceDialog("/", 0);
    }
]

module.exports = dialogs;

```

In questa cartella sono implementati i flussi di dialogo, nonchè gli stati del bot, per i 3 canali di comunicazione. 

Gli stati realmente implementati per il chatbot IMI sono relativi al canale Mobile. Negli altri canali il flusso implementato è a titolo di esempio.

**State ['\']**

All'avvio del programma, il bot si trova nello stato root, dove resta in ascolto sul canale mobile.

Alla ricezione di un messaggio, il bot inoltra quest'ultimo a Dialogflow per l'identificazione degli intents e delle entities e passa nello stato ['\start']

**State ['\start']**

In questo stato, il bot verifica se le entities richieste per lo specifico intent identificato sono tutte presenti o meno. Se mancano alcune entities, il bot risponde all'utente chiedendo i parametri mancanti, altrimenti passa nello stato ['\connectToOracleDb'] per interrogare il database.

**State ['\connectToOracleDb']**

In questo stato il bot esegue una specifica query sul db Oracle della banca, a seconda dell'intent matchato con l'API Dialogflow. Se la query ha successo e ho trovato le informazioni corrette, il bot le restituisce all'utente inviando un messaggio. Successivamente il bot ritorna nello stato iniziale ['\] in attesa di una nuova richiesta dell'utente.

Questo stato è quello che dovrà essere modificato andando a switchare su tutti gli intents configurati sull'agent in Dialogflow. A titolo di esempio è stato implementato un metodo per interrogare il db con l'intent "getCurvesAndLevels". 

Nella classe **AppLogic.js** è stato scritto il metodo di utility per la connessione al db Oracle e l'esecuzione della query.

I cases dello switch devono avere lo stesso nome degli intent configurati su Dialogflow.

## lib

Nella sezione lib sono integrati tutti i connettori generici ai database comunemente utilizzati e il connettore NLU per tutte le API di Natural Language Processing inserite nel progetto. Al momento l'API NLU è **Google Dialogflow** e il connettore usato per IMI è **OracleDBClient**.

### Connettore NLU Dialogflow

> APIAIClient.js metodo principale per l'estrazione del response

```javascript--node

	/**
     * submit a phrase to API.AI
     * this method extracts intents and entities
     * @param {String} message 
     */
    query(message, sessionId) {
        return new Promise((resolve, reject) => {
            var uri = this.endpoint + "query?v=" + this.apiversion
            request({
                uri: uri,
                method: "POST",
                json: true,
                headers: this.header,
                body: {
                    query: message,
                    timezone: this.timezone,
                    lang: this.lang,
                    sessionId: sessionId
                }
            }, function (err, response, body) {
                if (err) {
                    reject(err)
                } else {
                    if (response.statusCode == 429) {
                        reject("API.AI: request quota exceeded")
                    } else if (body.status.code >= 400 && body.status.code < 500) {
                        reject("API.AI" + body.status.errorType + ": " + body.status.errorDetails);
                    } else {
                        var result = {};
                        result.intent = body.result.metadata.intentName;
                        result.entities = body.result.parameters;
                        result.actionIncomplete = body.result.actionIncomplete;
                        // TODO map smalltalks, by now I just take the default phrase in speech field
                        result.speech = body.result.fulfillment.speech;
                        resolve(result);
                    }
                }
            })
        })
    }
	
```

Tutte le informazioni relative al response ricevuto da Dialogflow sono memorizzate nella variabile context di node.js.

Dal response di Dialogflow sono estratti i parametri intent, entities e actionIncomplete. 

Per salvare gli altri parametri contenuti nel response, aggiungere le opportune righe di codice in questo metodo.

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

### Connettore Database OracleDBClient

<aside class="notice">
Il connettore presenta la creazione di un pool di connessioni, supportata nella versione 12c di Oracle. Nel caso in cui il db sia 11g, eliminare la creazione del pool di connessioni ed effettuare una singola connessione alla volta ad ogni query richiesta dal bot.
</aside>

> OracleDBClient.js constructor

```javascript--node

var oracledb = require('oracledb');

class OracleDBClient {

    constructor() {        

      oracledb.createPool({
        user: 'username',
        password: 'password',
        connectString: 'localhost/XE'
      }, 
      function(err, pool) {
          if (err) {
            logger.error("Failed to create oracle db pool..." + pool.poolAlias);
            callback(err);
          } else {
            console.log(pool.poolAlias); // 'default'
          }
        }
      );
    }

	//...

};

module.exports = new OracleDBClient();

```

> OracleDBClient.js esempio query

```javascript--node

    /**
     * This is an example of query in Oracle Class. 
     * I used a connection to a local copy of EOS in order to test this module.
     * Example: get all the companies having a specific description.
     * @param {*String} descr: the description for AN_COMPANY table.
     */
    getCompanies(descr) {
      return new Promise(function(resolve, reject) {
        let conn; // Declared here for scoping purposes.
     
        var pool = oracledb.getPool()
        pool.getConnection()
          .then(function(c) {
            console.log('Connected to database');
     
            conn = c;          

            return conn.execute(
              "SELECT * FROM AN_COMPANY WHERE description = :descr",
              [descr],
              {
                outFormat: oracledb.OBJECT
              }
            );
          })
          .then(
            function(result) {
              console.log('Query executed');
              resolve(result.rows[0]);
            },
            function(err) {
              console.log('Error occurred', err);
              reject(err);
            }
          )
          .then(function() {
            if (conn) {
              // If conn assignment worked, need to close.
              return conn.close();
            }
          })
          .then(function() {
            console.log('Connection closed');
          })
          .catch(function(err) {
            // If error during close, just log.
            console.log('Error closing connection', e);
          });
      });
    }

```

Questa è la classe per la connessione al database Oracle.

La stringa di connessione al db della banca IMI deve essere inserita nel costruttore. I campi richiesti sono:

- username
- password
- host


&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

La query di esempio è una query SELECT, dal momento che tutte le interrogazioni al db saranno di sola lettura e non di scrittura.

Questa funzione va presa come esempio per poter aggiungere i metodi con le vere query e i veri parametri da ricercare nel db IMI. 

Il metodo rispettivo che chiamerà questa funzione dovrà essere implementato nella classe **AppLogic.js**




















