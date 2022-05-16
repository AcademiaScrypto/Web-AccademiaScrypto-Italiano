# Processo di Sviluppo - Analisi

![Análisis](/analisis.png)

### Modello a Cascata
Nonostante esistano diversi modelli per definire il ciclo vitale di un software, in questo corso parleremo del Waterfall Model, che è il più semplice con cui iniziare a programmare. Secondo il Modello a Cascata, il processo di realizzazione del software è strutturato in una sequenza lineare di fasi. Solo al termine di una fase si può proseguire con la successiva, e così via.

::: tip
- **"Pensare prima di programmare" 😉**
:::

### Analisi

La fase di analisi definisce i requisiti del software da sviluppare. Inizieremo con un colloquio individuale o una riunione di gruppo con i clienti. Dovremo indagare che cosa vuole di preciso il cliente o che cosa pensa di aver bisogno.

La comunicazione bidirezionale è importante ed è necessario il consenso di entrambe le parti per arrivare alla definizione dei requisiti del software. A tale scopo, viene creato un report ERS (System Requirements Specification). 

In questa fase i requisiti da definire sono:

*Requisiti funzionali*: con questi requisiti descriveremo in dettaglio cosa fa il sistema e come reagisce a diversi input e situazioni.

*Requisiti non funzionali*: con questi requisiti ci concentreremo, ad esempio, sulla capacità di archiviazione o l'affidabilità del sistema. Questi requisiti non includono il funzionamento del sistema.


Per rappresentare i dati ottenuti abbiamo diverse soluzioni. Diagrammi di flusso, diagrammi di transizione di stato, diagrammi di classe, diagrammi Entità/Relazione o dizionario di dati.

![analisis_1](/analisis_problema.png)

### Esempio di analisi

Leggere il raggio di una circonferenza e con quel dato calcolare e restituire il risultato di superficie e perimetro.

**Analisi**

**Definizione del problema:** Vogliamo sapere qual è il raggio di una circonferenza e calcolare area e perimetro. Per fare questo abbiamo bisogno di conoscere il valore del raggio e utilizzare le formule note per calcolare l'area e il perimetro.

|Specifiche||
|----------------|-----------------------------------------|
|Entrate|	Raggio della circonferenza (Variabile RAGGIO).|
|Uscite|	Superficie (Variabile SUPERFICIE). Perimetro (Variabile PERIMETRO)|
|Variabili|	RAGGIO, SUPERFICIE, PERIMETRO di tipo REALI.|

I dati di input e le informazioni di output verranno salvati in variabili. Le variabili possono essere diversi tipi di dati: interi, reali, stringhe, booleane,...


### Contenuti Extra:
- [Técnicas para Identificar Requisitos Funcionales y No Funcionales](https://sites.google.com/site/metodologiareq/capitulo-ii/tecnicas-para-identificar-requisitos-funcionales-y-no-funcionales) 
- [Especificación de requerimientos - Universidad de Granada](https://elvex.ugr.es/idbis/db/docs/design/2-requirements.pdf)
- [Video: Requerimientos Funcionales y No Funcionales en desarrollo de software](https://www.youtube.com/watch?v=SIr2qP59dA0)
- [Ejemplo muy completo - Documentos requerimientos](https://www.enabel.be/sites/default/files/tenders/anexo_a_requerimientos_funcionales_y_no_funcionales.pdf)
- [Ejemplo simple - Documento requerimientos](http://www.lsi.us.es/~javierj/cursos_ficheros/02.%20Un%20ejemplo%20de%20requisitos.pdf)
- [Video: GUÍA 3 ESPECIFICAR LOS REQUISITOS FUNCIONALES Y NO FUNCIONALES DEL SISTEMA](https://youtu.be/6oh4QXoBpUY)