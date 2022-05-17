# Branch

![Git básico](/git_ramas.svg)

Per la creazione di progetti semplici, un buon metodo quello di avere un ramo principale (branch) chiamato **master** e un ramo **development** dove verificheremo che tutte le modifiche funzionino correttamente prima di inviarle al **master** e quindi avremo multipli "features branches", in modo che ogni sviluppatore possa assumere un compito definito, lavorarci sopra e testarlo a proprio piacimento prima di applicare tali modifiche al ramo **development**. 

### Passaggi per iniziare a lavorare a una feature
Supponiamo di voler lavorare a una nuova **funzione** (feature). Per prima cosa dobbiamo creare un branch esclusivamente per la feature. Questo branch deve essere creato da **development**, e non da **master**. Quando apriamo *git* nella cartella in cui cloniamo questa repository dovremmo vedere in quale ramo ci troviamo, per questo useremo il seguente comando:

```rust
$ git branch
* master (può essere main o un altro nome simile)
```
Ora vogliamo dividere il nostro lavoro da **development**, per cui dobbiamo muoverci in questo branch:

::: tip
- Useremo **-b** solo se il branch dove vogliamo muoverci non esiste, verrà creato e ci sposteremo lì. 
:::

```rust
$ git checkout -b development 
Switched to a new branch 'development'
```

Ci dice è che la nostra cartella locale ha apportato modifiche per assomigliare al ramo **development**. Se ci sono differenze tra il ramo **master** e il ramo development potremo vederlo. Esempio, se manca un file che è in master ma non in development. **L'importante ora è che siamo nel ramo da cui vogliamo iniziare**.

Il passo seguente serve per creare *"feature branch"* nella quale lavoreremo, per cui dobbiamo fare:

```rust
$ git checkout -b feature_branch development
Switched to a new branch 'feature_branch'
```

Ci dice che abbiamo creato localmente il branch "feature_branch", che è un sub-branch di **development**. Ma poiché vogliamo lavorare con il server remoto, in questo caso Github, dobbiamo notificarlo per "connettere" il ramo locale "feature_branch" con il ramo remoto "feature_branch":

```rust
git push --set-upstream origin feature_branch
```

Ora siamo pronti. Quando apportiamo modifiche al codice che sono rilevanti per la funzionalità su cui stiamo lavorando, dobbiamo eseguire il push delle modifiche (*git push*), così invieremo le modifiche al ramo "feature_branch". 

### Passaggi per completare il lavoro su una "feature" 

Una volta fatte tutte le modifiche necessarie per aggiungere una "feature" al progetto, i test sono pronti e gli altri sviluppatori approvano le modifiche, dobbiamo "spingere" le modifiche al ramo di sviluppo. Per passare le modifiche da un ramo all'altro faremo *code review* attraverso le **Pull Requests** che Github implementa (tutto questo viene fatto su Github). Dopo aver terminato il lavoro, solo quando la funzionalità è già applicata e testata in fase di sviluppo, dobbiamo eliminare il ramo corrispondente. 


::: tip
- Per elminare un branch dobbiamo inviare il comando:
```
git branch -d nomebranch
// Nota che per eliminare un branch non è necessario esserci dentro.
```
:::

Da Github selezioniamo prima dal menu orizzontale **Pull Request** e dopo **Compare & pull request**:

![rama1](/git_rama1.png)

Per prima cosa possiamo **selezionare i rami da confrontare**, e dove verranno incorporate le modifice. In secondo luogo possiamo **confrontare** il codice del branch con la nuova funzionalità del ramo di destinazione per verificare che sia tutto corretto. Terzo e ultimo passaggio, una volta che le verifiche sono state fatte, creiamo una **pull request**:

![rama1](/git_rama2.png)

::: tip
- Hai notato che possiamo lasciare commenti per spiegare le modifiche che abbiamo fatto?
:::

Uniamo i rami, incorporando le nuove funzionalità nel codice di origine.

![rama1](/git_rama3.png)

E infine eliminiamo il branch con le nuove funzionalità:

![rama1](/git_rama4.png)

::: tip
-  Ricordati di eliminare localmente il branch in modo da non vederlo più tra i tuoi.
```
 git branch -d feature_branch
```
:::

::: warning Riassunto
- Se non sai in che branch ti trovi: ***git branch***
- Per cambiare branch: ***git checkout branch_in_cui_vuoi_spostarti***
:::