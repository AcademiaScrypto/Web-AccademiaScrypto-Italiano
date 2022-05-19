![Banner Api](./img/banner.png)

# 🔑 Radix API
Dall'inizio del 2022, dopo una completa ristrutturazione, Radix ci offre tre modi diversi per accedere ai dati di rete. Ciascuno risponde a diversi casi d'uso e permette agli sviluppatori di limitare l'accesso al proprio progetto secondo necessità.  
- Core API: : è progettata per l'accesso di basso livello allo stato del registro, fornisce tutti strumenti per costruire le transazioni e consente di firmare messaggi con la chiave privata del nodo, laddove abilitata. 
- System API: è progettata l’uso e il monitoraggio dei nodi da parte dei loro validator.
- Gateway API: questa porta è pensata per Explorers e la creazione di Wallet. In linea di principio è l'API che ci interessa e di cui di occuperemo in [Academia](/src/academia/README.md) poiché ci consente di accedere in modo semplice e veloce ai dati dei conti delle transazioni effettuate sulla Rete

### Gateway API
Il Gateway fornisce l'API pubblica per Wallet ed Explorer. Gestisce le query di lettura utilizzando il database e invia le richieste all'API principale di uno o più nodi completi. Radix offre un nodo Gateway per piccoli test, ma raccomanda di utilizzare il nostro nodo Gateway per i progetti più importanti.

Il consiglio che diamo è quello di concentrarsi sulla logica della nostra DApp e lasciare ad altri lo sviluppo e l'implementazione dell'infrastruttura. Per fare questo possiamo appoggiarci a progetti straordinari come  [Clana.io](https://clana.io/) che mette a nostra disposizione un Gateway scalabile e altamente fruibile per Radix.

### Cos'è API?
API è l'abbreviazione di *Application Programming Interfaces*, in italiano interfaccia di programmazione delle applicazioni. È un insieme di definizioni e protocolli utilizzati per sviluppare e integrare software applicativi, che consentono la comunicazione tra due applicazioni software attraverso un insieme di regole.

Quindi possiamo dire che le API sono una sorta di insieme di specifiche che permettono ai tuoi prodotti o servizi di comunicare con altri prodotti o servizi senza che sia necessario sapere come vengono implementati, il tutto per svolgere una o più funzioni.

Nel nostro caso ci permette di interagire con la rete pubblica Radix. Ci permette di inviare richieste o transazioni.

*In questo corso faremo solo esempi con JavaScript, NodeJs e Firebase Functions*

Vi lascio un esempio di accesso all'API di test Radix, utilizziamo il modulo *node-fetch* per NodeJsOs:

```javascript
import fetch from 'node-fetch';

var raw = JSON.stringify({"network_identifier": {"network": "mainnet"},
           "account_identifier": {"address": "rdx1qspqp0nyg3a3dj7e8vkd2ecxrgnrgvsl4d72efx2a4jwavfn995652c7f6uqy"},
           "cursor":"3",
           "limit": 1});


var requestOptions = {
  method: 'POST',
  headers: {
    "Content-Type": "application/json",
    "x-radixdlt-target-gw-api": "1.0.3",
  },
  body: raw,
  redirect: 'follow'
};

const response = await fetch('https://mainnet.radixdlt.com/account/transactions', requestOptions);
const data = await response.json();

console.log(data.transactions);
```


### Link API
- [Documentazione API RADIX](https://docs.radixdlt.com/main/apis/introduction.html)
- [Postman specification Gateway API](https://documenter.getpostman.com/view/14449947/UVXnHaJf)
