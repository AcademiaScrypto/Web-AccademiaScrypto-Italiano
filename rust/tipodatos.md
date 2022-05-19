# Tipi primitivi
I tipi di dato primitivi, nella programmazione, spesso vanno di pari passo con le operazioni aritmetiche e Rust non è diverso.
## 🔢 Numeri
Abbiamo tipi numerici interi come **i32** o **i64** e tipi numerici in virgola mobile come **f64**. 

<div class="alert alert-dark" role="alert">
  <b>Cos’è un numero intero?</b>
  <p>Gli interi sono quelli che non contengono decimali, possono essere positivi o negativi oltre allo zero.</p>
  <hr>
  <b>Cos’è un numero a virgola mobile?</b> 
  <p>Diventa un numero reale, 😅, è fondamentalmente un numero con decimali.</p>
  <i>Nota: la spiegazione sarebbe un po' più complessa, ma per ora atteniamoci a questa.</i>
</div>

Tieni presente che quando si dichiarano variabili è possibile specificare il tipo di di dati indicandolo esplicitamente nella dichiarazione.   

Ad esempio: se vogliamo memorizzare una variabile che contiene un intero senza segno, di dimensione 8 bit, dobbiamo fare qualcosa di simile al seguente:  
```
fn main() {
  let age: u8 = 18;
  println!("You are {} years old.", age)
}
```
*Nota: questo è applicabile anche ad altri tipi di dati*
::: tip
- •	Se avessi appena scritto *let age = 18;*, Rust avrebbe dedotto che il tipo di età era di tipo i32. Sebbene l'inferenza semplifichi le cose, non è sempre il modo migliore per programmare. 
:::

## ✔️❌ Booleani
Abbiamo anche tipi di dati bool che possono contenere i valori **true** o **false**.

## 🔠 Stringhe, & str e caratteri
I dati di tipo **string**, **&str** e **char** vengono utilizzati per memorizzare i testi.

::: tip 
- l tipo di carattere (char) è scritto tra virgolette singole, esempio 'C'
- Le stringhe di dati (string) sono implementate come una raccolta di byte. 
- **&str** è noto come segmento stringa o stringa letterale, **immutabile**
:::
Ejemplo:
```
let s = "Hacia la ciencia de datos";
```
qui Rust deduce che *s* è  **&str** invece con il seguente comando lo prenderà come una stringa di tipo **string**:
```
let s = String :: from ("Hacia la ciencia de datos");
```
## 👨‍👩‍👧‍👦 Raccolte di dati
**Tupple**, **Array**, **Vector** e  **hash map** come raccolte di dati che diventano essi stessi dati.

- **Array** di dimensione fissa (accesso più veloce, ne abbiamo già parlato!)
```
let mi_array = [1, 2, 3];
```
- **Tupple** viene utilizzato per memorizzare diversi tipi di dati in una raccolta, ad esempio:
```
let mezcla: (i32, f64, char) = (7, 3.141592, 'K');
let pi = mezcla.1;
println!("El numero pi es aproximadamente igual a {}... ", pi);
```
- **Vector** è una raccolta di dati che può aumentare le dimensioni, può contenere solo un tipo di dati. 
```
sea ​​v = vec! [1, 2, 3];
```
**Hash map**: memorizza una mappatura di chiavi di tipo *k* su valori di tipo *v*.
```
fn main() {
  use std::collections::HashMap;

  let mut webs = HashMap::new();

  webs.insert(String::from("Estado"), "https://status.radixdlt.com/");
  webs.insert(String::from("Explorador"), "https://explorer.radixdlt.com/");
  webs.insert(String::from("Wallet"), "https://wallet.radixdlt.com/");
  webs.insert(String::from("Radix Scrypto Github"), "https://github.com/radixdlt/radixdlt-scrypto");

  }
```
## 🔣 Enum
 **Enum** un tipo di dati che gioca un ruolo molto importante in Rust, possiamo effettivamente creare un elenco di possibili valori come:  
```
enum Genero {
    Hombre, Mujer
}
```

## ♾️ Altri tipi
Certo…! Abbiamo anche  **structs**, **methods** e  **associated functions** cioè un tipo di classe di  programmazione orientata agli oggetti. **Trait** è come un’interfaccia di GoLang.


::: tip Structs
- Nella sintassi dei **blueprints** utilizziamo **structs** 😉
:::

::: warning Riassunto
- Suggerimento: l'uso di alcuni di questi tipi di dati viene compreso meglio attraverso esempi e facendo pratica! Lo vedremo presto, per ora, facciamoci un'idea generale dei tipi di dati che esistono in Rust.
:::

