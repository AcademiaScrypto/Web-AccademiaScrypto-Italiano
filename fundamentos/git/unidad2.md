# Lavoro Locale - Basi

![Git básico](/git_basico.png)

### Workspace
📁 È la cartella locale, cioè sul nostro computer, dove abbiamo salvato il nostro progetto. Per poter inizializzare un progetto con Git useremo il comando *init*:

```
<cartella progetto> git init
```
Il comando *git init* creerà una cartella nascosta *.git* e un archivo *.gitignore* nella cartella/directory locale dove lo abbiamo eseguito. *(Nota: Questo passaggio verrà eseguito solo una volta)*

### Add
Il comando *add* aggiunge file all' **Index** di Git. Con il *punto* dopo *add* indichiamo che vogliamo aggiungere tutti i file che pendono dalla directory corrente, comprese le sottocartelle. 

```
git add .
```

### Index 
Questo indice è anche noto come area di montaggio o area di preparazione. In quest'area i file sono in attesa di essere salvati nella repository. Per aggiungere file a quest'area useremo il comando *add* e per rimuovere i file il comando *rm*. 

```
git status
```
Il comando *status* ci mostra lo stato dei nostri file nell' *index*.

### Commit
Il comando *commit* salva tutte le modifiche apportate ai file trovati nell'indice *git*. Insieme a una breve descrizione fatta dall'utente permetterà di sapere rapidamente cosa è stato cambiato. L'opzione *-m* ci consente di includere un messaggio di non più di 50 caratteri. Questi messaggi sono molto importanti e devono essere precisi per aiutare a capire l'entità delle modifiche apportate.

```
git commit -m "messaggio di non più di 50 caratteri"
```

### Repository

È il luogo in cui viene conservata la raccolta di file con tutte le loro diverse versioni.

```
git log
```
Il comando log ci mostra l'elenco dei *commit* salvati nella Repository.

::: tip
- Qui finisce l'utilizzo di Git in forma locale. Il ciclo va da:  
add -> commit
:::



