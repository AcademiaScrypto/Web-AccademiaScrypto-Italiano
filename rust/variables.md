# Variabili
Le Varibiali in Rust si definiscono attraverso la parola chiave: *let*
::: tip
**Le Variabili:** Sono immutabili e di forma costante
:::
Facciamo un esempio. (OCCHIO: *Il seguente codice non verrà compilato.*)
```
fn main() {
    let x = 1;
    println!("El valor x es: {}", x);
    x = 0; 
    println!("El valor x es: {}", x);
}
```
Leggiamo linea per linea:
1.	Dichiariamo la funzione principale 'main', per il momento fermiamoci qua.
2. **Qui diamo alla variabile di nome 'x' il valore 1**, usiamo la parola chiave let seguita dal nome che vogliamo assegnare alla variabile più il simbolo di uguale (=) e il valore che vogliamo dare.
3. Stampiamo, nel terminale, il valore di x.
4. Si ottiene un errore (*x=0;*), ricorda che **in Rust le variabili ,per impostazione predefinita, sono immutabili,**, ovvero non è possibile modificare il loro valore iniziale.
5. Questa riga non verrebbe eseguita, a causa dell'errore precedente.

Per risolvere questo problema, possiamo fare quanto segue:
```
fn main() {
    let mut x = 1;
    println!("The value of x is: {}", x);
    x = 0;
    println!("The value of x is: {}", x);
}
```
Vedi la differenza?... si trova sulla riga 2.  
Sì, una nuova parola chiave: mut è stata inclusa tra la parola chiave let il nome della variabile. Con 'mut' la variabile acquisisce mutabilità, ora possiamo cambiarne il valore come fatto nella riga 4. 

::: warning Riassunto
- Le variabili sono immutabili per definizione
- *let*: permette di creare variabili
- *mut*: attribuisce mutabilità alla variabile.
:::

