# Crear nuestra primera colecci贸n de NFT

::: warning
- Esta unidad est谩 actualizada para Scrypto version 0.4 o mayor -> [Instrucciones actualizaci贸n](/scrypto/instalacion/actualizacion.md)
:::


::: tip 馃摵
- [Muy pronto Video Demostraci贸n]()
:::

En la unidad de hoy vamos a crear nuestros primeros *NFT*, partiendo de la definici贸n b谩sica de que un NFT es un recurso 煤nico e indivisible en la red. Para no ser triviales vamos a crear una m谩quina expendedora de tickets para una sala de cine. La idea es que al instanciar el blueprint *TicketNft* pre-crearemos una serie de NFT que daran acceso a una butaca numerada en una sala de cine. 隆Y luego los venderemos!

Este seria el c贸digo completo, 隆no te asustes! vamos a analizarlo poco a poco:


*(Nota: este ejemplo lo encuentras en el [repositorio oficial de Scrypto en Github](https://github.com/radixdlt/scrypto-examples/blob/main/core/hello-nft/src/lib.rs))*

```rust
use scrypto::prelude::*;

#[derive(NonFungibleData)]
pub struct Ticket {
    pub fila: u32,
    pub columna: u32,
}

blueprint! {
    struct TicketNft {
        tickets_disponibles: Vault,
        ticket_precio: Decimal,
        bolsa_xrd: Vault
    }

    impl TicketNft {
        pub fn instanciate_ticket_nft(price: Decimal) -> ComponentAddress {
            let mut tickets = Vec::new();
            for fila in 1..5 {
                for columna in 1..5 {
                    tickets.push((
                        NonFungibleId::random(), 
                        Ticket { fila, columna }
                    ));
                }
            }

            let ticket_bucket = ResourceBuilder::new_non_fungible()
                .metadata("name", "Ticket")
                .initial_supply(tickets);

            Self {
                tickets_disponibles: Vault::with_bucket(ticket_bucket),
                ticket_precio: price,
                bolsa_xrd: Vault::new(RADIX_TOKEN),
            }
            .instantiate()
            .globalize()
        }

        pub fn comprar_ticket(&mut self, mut payment: Bucket) -> (Bucket, Bucket) {
            assert!(self.tickets_disponibles.amount() >= Decimal::one(), "No hay tickets disponibles");
            assert!(payment.amount() >= self.ticket_precio, "Saldo insuficiente para comprar Ticket");

            self.bolsa_xrd.put(payment.take(self.ticket_precio));
            let ticket = self.tickets_disponibles.take(1);

            (ticket, payment)
        }

        pub fn disponible_ticket(&self) -> BTreeSet<NonFungibleId> {
            self.tickets_disponibles.non_fungible_ids()
        }
    }
}
```

### Despu茅s de la primera l铆nea, el prelude de Rust que carga las caracter铆sticas de Scrypto , nos encontramos un macro de Rust:
```rust
#[derive(NonFungibleData)]
pub struct Ticket {
    pub fila: u32,
    pub columna: u32,
}
```
Scrypto utiliza los *Attributes* (#[Attr]) de Rust para implementar, en este caso, una estructura de datos la cual nos permitir谩 agregar a un recurso 'no fungible' (NFT) una serie de metadatos/atributos ya sean inmutables, como en este caso, como mutables. Dichos *'metadatos/atributos'* se guardaran como parte del NFT en el libro mayor de Radix.

::: tip Derive:
- El derive atributo permite que se generen autom谩ticamente nuevos elementos para las estructuras de datos. [+info](https://doc.rust-lang.org/reference/attributes/derive.html)
:::

![cine](/cine.jpg)

En este caso, en concreto, los metadatos/atributos tiene que ver con la numeraci贸n de butacas de una sala de cine, por eso hemos creado las variables fila y columna de [tipo i32](/rust/tipodatos.md). 

### Sigamos analizando la **estructura**:

```rust
struct TicketNft {
        tickets_disponibles: Vault,
        ticket_precio: Decimal,
        bolsa_xrd: Vault
    }
```
En ella definimos tres dato: una contenedor permanente (Vault) que guardara tantos TicketNft como filas y columnas tengamos, lo llamaremos *tickets_disponibles*. Tambi茅n asignaremos un precio a cada TicketNft, ya sab茅is normalmente cuando hablamos de precios se declara como Decimal. Y finalmente crearemos otro contenedor permanente (Vault) para guardar los XRD que ganemos con la venta de los TicketNft, a este dato lo llamaremos bolsa_xrd. 
 
### Analicemos ahora la funci贸n constructora *instanciate_ticket_nft*:
```rust
pub fn instanciate_ticket_nft(price: Decimal) -> ComponentAddress {
    let mut tickets = Vec::new();
        for fila in 1..5 {
            for columna in 1..5 {
                tickets.push((
                    NonFungibleId::random(), 
                    Ticket { fila, columna }
                ));
            }
        }

        let ticket_bucket: Bucket = ResourceBuilder::new_non_fungible()
            .metadata("name", "Ticket")
            .initial_supply(tickets);

        Self {
            tickets_disponibles: Vault::with_bucket(ticket_bucket),
            ticket_precio: price,
            bolsa_xrd: Vault::new(RADIX_TOKEN),
        }
        .instantiate()
        .globalize()
}
```
Lo primero que vemos es que la funci贸n *instanciate_ticket_nft* tiene un par谩metro de entrada de tipo *Decimal* para establecer el precio de venta de los TicketsNFT, acto seguido vemos que solo tiene como par谩metro de salida un dato de tipo *ComponentAddress* como es normal en las funciones constructoras ya que finalmente esa es su funci贸n principal instanciar un componente y devolver gracias a *globalize()* una direcci贸n para este recurso. 

```rust
let mut tickets = Vec::new();
        for fila in 1..5 {
            for columna in 1..5 {
                tickets.push((
                    NonFungibleId::random(), 
                    Ticket { fila, columna }
                ));
            }
}
```
En este bloque de c贸digo lo que hacemos es crear un [vector](/rust/vectores.md) que contendr谩 dos datos (en realidad son 3 馃槄): el primero un identificador 煤nico y el segundo el tipo de dato *Ticket* (recuerdas que lo definimos al inicio del blueprint gracias a los Attributes de Rust) que a su vez contiene dos datos, a saber, una fila y una columna. Estos tres datos/atributos har谩n 煤nico a nuestro *no fungible*. 

Para conseguir esto primero declaramos una variable mutable, si no especificamos su mutabilidad luego no podr铆amos alterarla, llamada *tickets* y la inicializamos como un vector vac铆o de la siguiente manera:

```rust
let mut tickets = Vec::new();
```

::: tip
- Recuerda que, en Rust, un **vector** es una colecci贸n de datos de un mismo tipo.
:::

Este vector, tickets, lo vamos a rellenar recorriendo dos [bucles](/rust/bucles.md) anidados. El primer bucle recorrer谩 el n煤mero de filas indicado, en este caso cuatro (1..5). El segundo bucle recorrer谩 el n煤mero de columnas de cada fila, en este caso cuatro (1..5). La mec谩nica es la siguiente: el bucle de las filas realiza la primer iteraci贸n para la fila 1 e inmediatamente el segundo bucle, el de las columnas, itera las 4 columnas. Este proceso se repite tantas veces como filas y columnas haya, dando como resultado, en este caso, un vector con una lista de 16 tickets. 

::: tip
- 驴16?, 驴no son 25? recuerda que la expresi贸n dentro del for toma una posici贸n de inicio y una de fin, por tanto solo itera los elementos entre valores.
:::

Ahora que ya solo nos queda guardar cada uno de esos tickets, junto a todos sus atributos:

```rust
tickets.push((
    NonFungibleId::random(), 
    Ticket { fila, columna }
));
```

Para ello utilizamos la propiedad de los vectores: *push* que no permite introducir nuevos elementos de forma ordenada, uno detr谩s del otro. En este caso vamos a guardar una [tupla](/rust/tipodatos.md) que contendr谩 un tipo de dato *NonFungibleId* el cual generaremos autom谩ticamente con la funci贸n *[random()](https://radixdlt.github.io/radixdlt-scrypto/scrypto/engine/types/struct.NonFungibleId.html#method.random)* y un tipo de dato *ticket* que a su vez guarda una fila y una columna. 

Una vez tenemos toda la lista de *tickets* con sus atributos vamos a pasar a crear los NFT para luego finalmente guardarlos en el contenedor permanente *tickets_disponibles*. Pero vayamos por partes:

```rust
let ticket_bucket: Bucket = ResourceBuilder::new_non_fungible()
            .metadata("name", "Ticket")
            .initial_supply(tickets);
```

Primero creamos un nuevo recurso, en este caso de tipo non_fungible (*new_non_fungible()*). Al cual asignamos un nombre y un suministro inicial (*initial_supply*) con la lista de **tickets** que antes creamos. Finalmente guardamos este recurso en una variable llamada *ticket_bucket* que en este caso act煤a como un contenedor temporal (bucket).

Una vez tenemos los 16 'no fungibles' creados, que corresponden a cada una de las butacas del cine y tenemos el precio por ticket ya solo nos queda instanciar el componente y crear la direcci贸n, primero pasando a la estructura los datos que vamos a guardar la colecci贸n de tickets/nft y luego el precio:

```rust
tickets_disponibles: Vault::with_bucket(ticket_bucket),
ticket_precio: price,
```
::: tip Bucket -> Vault
- Para pasar un contenedor temporal a uno permanente utilizaremos el siguiente m茅todo:
```rust	
Vault::with_bucket(Bucket)
```
:::

*bolsa_xrd* se crea como un contenedor vac铆o que acepta XRD gracias al par谩metro *RADIX_TOKEN*. Ten en cuenta que en un Vault, normalmente solo guardaremos recursos de un mismo tipo.

```rust
Self {
            tickets_disponibles: Vault::with_bucket(ticket_bucket),
            ticket_precio: price,
            bolsa_xrd: Vault::new(RADIX_TOKEN),
        }
        .instantiate()
        .globalize()
```

Finalmente *.instanciamos()* el componente y *.globalize()* el componente, asignando una direcci贸n, para que se pueda acceder desde cualquier lugar.

### M茅todo *comprar_ticket*:

```rust
pub fn comprar_ticket(&mut self, mut payment: Bucket) -> (Bucket, Bucket) {
    assert!(self.tickets_disponibles.amount() >= Decimal::one(), "No hay tickets disponibles");
    assert!(payment.amount() >= self.ticket_precio, "Saldo insuficiente para comprar Ticket");

    self.bolsa_xrd.put(payment.take(self.ticket_precio));
    let ticket = self.tickets_disponibles.take(1);

    (ticket, payment)
}
```

Gracias a la unidad anterior ya estamos familiarizados con este c贸digo. Pasamos a este m茅todo la estructura a traves de la part铆cula *self*, adem谩s le damos la propiedad de mutable con la part铆cula *mut* y as铆 podemos realizar cambios en los datos y contenedores de la estructura. Adem谩s entramos como par谩metro un contendedor temporal llamado *payment* que contendr谩 los XRD para el pago del NFT.

Y de vuelta recibiremos dos contenedores temporales o *bucket*, uno para devolvernos el NFT y el otro para devolvernos el cambio de XRD si fuera necesario. 

```rust
assert!(self.tickets_disponibles.amount() >= Decimal::one(), "No hay tickets disponibles");
assert!(payment.amount() >= self.ticket_precio, "Saldo insuficiente para comprar Ticket");

```
Primero verificamos, gracias a los macro *assert!*, que la bolsa de tickets tenga todav铆a entradas que vender y segundo verificamos que nos han pasado saldo suficiente como para pagar el precio requerido. 

Recuerda que los macros *assert!* se componen de una pregunta que espera de vuelta un boleano, si o no, si dicha sentencia no se cumple, false, romper谩 el m茅todo y devolver谩 por consola un mensaje de error formateado por nosotros. 

```rust
self.bolsa_xrd.put(payment.take(self.ticket_precio));
```
Esta composici贸n de c贸digo ya la vimos en la unidad anterior, tomamos el precio declarado para el NFT (*payment.**take**(self.ticket_precio)*) y lo ponemos en el contenedor permanente destinado a guardar los XRD llamado *bolsa_xrd* (*self.bolsa_xrd.**put***).

```rust
let ticket = self.tickets_disponibles.take(1);
```
Guardamos en la variable *ticket* el NFT que hemos tomado, gracias al comando *take*, del contenedor permanente que tiene el componente para guardar los NFT, llamado *tickets_disponibles*.

```rust
(ticket, payment)
```
Y finalmente devolvemos a la cuenta que este ejecutando este m茅todo el ticket junto con las vueltas del pago.

### M茅todo *disponible_ticket*:

```rust
pub fn disponible_ticket(&self) -> BTreeSet<NonFungibleId> {
            self.tickets_disponibles.non_fungible_ids()
}
```
Este m茅todo nos devolver谩 un listado de los NFT disponibles en el componente. Para ello devuelve un *BTreeSet* que incluye un tipo de dato *NonFungibleId* que es el necesario para representar los identificadores 煤nicos del non-fungible. 

::: tip BTreeSet
- Este tipo mapas tipo B-tree son mucho m谩s eficientes en el uso de memoria que otro tipo de colecciones, ya que entre otras cosas minimizan el cantidad real de trabajo para realizar una b煤squeda. 
- En realidad es como un *Vector* pero no permite valores duplicados.
:::

Los recursos de tipo *no fungible* tienen algunos implementaciones gracias a scrypto que nos ayudan en la programaci贸n como en este caso: *non_fungible_ids()*, que nos devuelve una colecci贸n con todos los identificadores en el contenedor. 


```rust
 self.tickets_disponibles.non_fungible_ids()
```
::: tip
- Una *expresi贸n* no lleva el punto y coma al final y as铆 retorna el valor. 
::: 

### Compilaci贸n y ejecuci贸n

Ahora que hemos le铆do y analizado el c贸digo que nos propone el ejemplo, nos toca compilarlo y ejecutarlo. Te voy a indicar cuales serian los pasos, los comando seguro que ya sabes cuales son.

1. Limpiar el simulador
2. Crear un Package
3. Crear una cuenta (recuerda copiar la direcci贸n de los XRD de tu cuenta)
4. Copiar o escribir el c贸digo (recuerda guardar)
5. Publicar
6. Instanciar componente (recuerda que en este caso hay que a帽adir el argumento del precio)
7. Probar m茅todo *comprar_ticket*
8. Probar m茅todo *disponible_ticket*

::: details Solo para aquellos (*tipo Emilio* 馃お) que no quieren pensar!!!
1. Limpiar el simulador
```rust
resim reset
```
2. Crear un Package
```
scrypto new-package ticketcine
```
3. Crear una cuenta (recuerda copiar la direcci贸n de los XRD de tu cuenta)
```rust
resim new-account
set acct [Address de la cuenta que acabamos de crear]
```
4. Copiar o escribir el c贸digo (recuerda guardar ctrl + s)
- Recuerda guardar el c贸digo de este ejercicio dentro del archivo lib.rs que has creado en la carpeta *\radixdlt-scrypto\ticketcine\src\lib.rs*
5. Publicar y guardamos la direcci贸n del Package
```rust
resim publish .
set pack [New Package Reference]
```
6. Instanciar componente (recuerda que en este caso hay que a帽adir el argumento del precio) y guardar la direcci贸n del componente.
```rust
resim call-function $pack TicketNft instanciate_ticket_nft 5.5
set comp [direcci贸n del componente]
```
7. Probar m茅todo *comprar_ticket*
```rust
resim show $acct
set xrd [resource_def del XRD]
resim call-method $comp comprar_ticket 10,$xrd
```
8. Probar m茅todo *disponible_ticket*
```rust
resim call-method $comp disponible_ticket
```
:::

::: tip
- El comando **resim show <direcci贸n>** es sumamente 煤til para ir comprobando tanto en las cuentas como componentes y recursos como todo funciona. 
:::

Seguro que con estas indicaciones lo ten茅is muy f谩cil 馃槄  

En esta unidad no solo has creado NFT's sino que has visto como pueden ser m谩s que simplemente *arte digital*, en este caso han sido entradas para una sala de cine. Este ejemplo se puede mejorar mucho, 驴te atreves a incluir, por ejemplo, un verificador de que nos pagan todo el precio del ticket tal y como hicimos en la unidad anterior?. Suerte!

::: warning Importante:
- Soy muy consciente de que hay muchas que no has entendido, 隆TRANQUILO!, no te rindas, las entender谩s... 馃槈
:::