# Lavoro in remoto - GitHub

![Git básico](/git_basico.png)

::: warning Requisiti
- [Dovresti già avere un account su GitHub](/fundamentos/git/unidad1.html#crea-una-cuenta-en-github), tutto ciò di cui hai bisogno è un'e-mail, vai su [GitHub](https://github.com/) e procedi con la registrazione.
:::

### Repository

Le Repository su GitHub sono i 'luoghi' in cui vengono archiviati i singoli progetti. Per creare una nuovo repository all'interno di Github:

1. Nell'angolo in alto a destra di qualsiasi pagina, utilizza il menù a discesa e seleziona "New Repository".

![git1](/github_1.png)

2. Digita il nome della tua Repository, e una descrizione opzionale.

![git2](/github_2png.png)

3. Scegli la visibilità della tua Repository: Pubblica o Privata (normalmente si seleziona Pubblica)

![git1](/github_3.png)

4. Clicca su "Create Repository".

![git1](/github_4.png)

5. GitHub ci mostra le istruzioni da seguire per caricare il nostro progetto nella *repository* appena creata:
 
![git1](/github_5.png)

### Inviare il nostro progetto alla repository di Github

CCome abbiamo visto prima, quando creiamo un nuovo repository GitHub conclude dandoci le istruzioni da seguire per caricare il nostro progetto. Nell'esempio precedente erano le seguenti:
 
![git1](/github_6.png)

1. Entrare nella directory del nostro *Package* di Scrypto (con il terminale)

*Nota: per continuare con i seguenti passaggi dobbiamo aver già avviato git e aver eseguito il commit delle modifiche localmente, come mostrato nella [unità precedente](/fundamentos/git/unidad2.md), questa directory del package.*

![git1](/github_7.png)

2. E utilizza il comando *git remote add* seguito da: nome remoto *holamundo* (shortname) e l'Url remoto, in questo caso sarà quello della nostra repository git.

```
git remote add holamundo https://github.com/noelserdna/scrypto-hola-mundo.git
```

::: tip
- Il comando *git remote* ci mostra l'elenco delle repository remote che sono state aggiunte.
```
git remote
```
:::

3. Creamo un *branch* principale dove verrà salvato il nostro progetto, nella prossima unità vedremo cosa sono i branch e come utilizzarli. 

```
git branch -M main
```

4. Inviamo il nostro progetto a GitHub con il comando *git push* seguito da *-u* che stabilisce quale sia la repository principale, cioè la repository remota che abbiamo aggiunto per sviluppare il progetto: *holamundo* e il branch a cui abbiamo dato il nome *main* nel passo precedente.

```
git push -u holamundo main
```

![git1](/github_8.png)

::: tip
- A partire da ora utilizzeremo solo il comando *git push* per inviare i file nella nostra repository Github.
```
git push
```
:::











