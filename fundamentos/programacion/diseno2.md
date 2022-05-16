# Processo di Sviluppo - Progettazione 2

![diseño](/diseño.png)

### Problemi complessi

Le soluzioni a problemi complessi possono richiedere molti passaggi. Le strategie più comuni sono:

- **Scomposizione o divide et impera:** consiste nel dividere un grande problema in unità più piccole che possono essere risolte una alla volta.
    - Esempio: possiamo minimizzare il problema della pulizia di un’intera casa in compiti più semplici, cioè pulire una stanza per volta.
- **Risolvere per analogia:** Dato un problema, si tratta di ricordare un compito simile che è già stato risolto. I due problemi analoghi possono anche appartenere ad aree di conoscenza completamente diverse.
    - Esempio: Il calcolo medio delle temperature della province della Toscana e la media dei voti di una classe di studenti si realizza allo stesso modo.

Scomporre il problema originale in sottoproblemi più facili e quindi dividere questi sottoproblemi in sottoproblemi ancora più semplici è chiamato design top-down. Dopo una prima descrizione del problema (molto vaga), segue un approccio più dettagliato con passaggi più specifici. Questo processo è chiamato raffinamento dell'algoritmo.

**Esempio di progettazione:**

Leggi il raggio di una circonferenza e con quel dato calcola e restituisci il risultato della sua superficie e il perimetro.

- Può essere suddiviso in tre sottoproblemi più semplici::
    - Leggere Raggio
    - Calcolare Superficie
    - Calcolare Perimetro
    - Pubblicare i risultati  

- Raffinamento dell’algoritmo:
    - Leggere Raggio
    - Superficie <- PI * RAGGIO ^ 2
    - Perimetro <- 2 * PI * RAGGIO
    - Pubblicare RAGGIO, SUPERFICIE, PERIMETRO
Possiamo vederlo in un diagramma strutturato:

![topdown](/algoritmo_estructurado.png)

### Contenuti Extra:
- [Corso di intruzione alla programmazione con pseudo-codice - SPAGNOLO](https://plataforma.josedomingo.org/pledin/cursos/programacion/)
- [PROGETTAZIONE STRUTTURA DI ALGORITMI - SPAGNOLO](http://www.geocities.ws/apuntesitpn/dise/temario.htm)
