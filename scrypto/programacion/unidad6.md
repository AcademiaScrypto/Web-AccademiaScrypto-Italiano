# Creación de DAO - Nuevos miembros

::: warning
- Esta unidad **NO** está actualizada para Scrypto version 0.4 o mayor -> [Instrucciones actualización](/scrypto/instalacion/actualizacion.md)
:::

::: tip 📺
- [Video Demostración](https://youtu.be/wFywZeDOcw8)
:::

Durante las siguientes unidades vamos a crear una DAO (Organización autónoma descentralizada), que sin duda durante 2022 están llamadas a ser tendencia y seguro que han llegad1o para quedarse entre nosotros. Aprender a crear DAOs puede tener un gran potencial. ¡Comencemos!

### Análisis
**Problema:** Deseamos crear una DAO donde los afiliado van a poder darse de alta sin ningún tipo de autorización. 

**Análisis:** Al pedirnos una DAO ya estamos dando por supuesto que vamos a crear una dApp. Para ello vamos a crear un *componente*, dentro de la red de Radix, que nos va a permitir dar un NFT a cada nuevo afiliado para que pueda demostrar su pertenencia y en un futuro identificarse dentro de la DAO.

### Diseño:
![Diagramas](/diagramas_unidad6.png)

Dos diagramas: uno para la función constructora y el segundo para la primera funcionalidad que nos solicitan y es la de poder registrarse cualquiera al DAO retornando un NFT como acreditación de su pertenencia al DAO. *(Nota: en cuanto más lenguaje más natural expresemos en estos diagramas mejor, yo a veces peco de incluir términos técnicos propios del lenguaje o plataforma que utilice)*

### Programación:

### NFT Estructura Datos:

Como ya vimos en la [unidad anterior](/scrypto/programacion/unidad5.md) *Scrypto* pone a disposición una macro para poder crear una estructura de datos, mutables e inmutables, que luego podremos incorporar a nuestro NFT:

```rust
#[derive(NonFungibleData)]
struct DatosAfiliado {
    nombre: String
}
```

En este caso hemos llamado a la estructura: *DatosAfiliado* (recuerda, es algo arbitrario) y de momento solo le hemos implementado un dato de tipo *String* (Cadena de caracteres) para guardar el nombre del afiliado (sera un dato inmutable).

### La estructura:

Vamos a necesitar crear un contador para guardar el número de afiliado, lo normal es que esta vez sea un número legible y no uno aleatorio, y que a su vez nos hará de identificador del NFT. Para poder utilizar el 'envoltorio' de identificadores [*NonFungibleKey*](https://radixdlt.github.io/radixdlt-scrypto/scrypto/types/struct.NonFungibleKey.html#) tenemos que crear un dato de tipo u128.

::: tip
|DATA TYPE   |MIN    |MAX |
| ---------- |-------|----|
|u8	         |0      |255 |
|u16         |0      |65535 |
|u32         |0      |4294967295 |
|u64	     |0      |18446744073709551615 |
|u128        |0      |340282366920938463463374607431768211455 |
:::

Luego necesitaremos guardar la definición, con las características,  del tipo de NFT que queremos dar a cada afiliado, sera de tipo ResourceDef.

Y de momento solo nos quedaría añadir un contenedor permanente (Vault) para guardar una *insignia* (badge) que autorice al componente para mintear nuevos NFT. En este caso la insignia con autorización para acuñar se guardara en el propio componente para que la utilice cuando sea necesario. 

```rust
struct SistemaAfiliacion {
        minteador: Vault,
        def_afiliado_nft: ResourceDef,
        num_afiliado: u128
}
```

### Función constructora *new*

1. Primero vamos a crear la autorización para crear NFT:
```rust
let minteador = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
                        .metadata("name", "Autorización mintear nuevos NFT")
                        .initial_supply_fungible(1);
```
Creamos un recurso con, *ResourceBuilder*, de tipo fungible con el parámetro *DIVISIBILITY_NONE* que impide fraccionar el recurso. Un nombre que lo identifique, es especialmente interesante para poder identificarlo cuando realizamos *resim show*. Y finalmente un suministro de 1.

2. Creamos la definición del NFT que vamos a entregar a cada persona que se registre en el DAO:

```rust
let def_afiliado = ResourceBuilder::new_non_fungible()
                            .metadata("name", "Afiliado DAO")
                            .flags(MINTABLE)
                            .badge(minteador.resource_def(), MAY_MINT)
                            .no_initial_supply();
```
En este caso el recurso será de tipo no fungible, *new_non_fungible*, y tiene varios *flags* que restringen sus características:
- MINTABLE: Se puede acuñar más del recurso

Seguidamente indicamos al recurso que va a tener una insignia, *.badge*, la definición de la insignia y a que cosas tiene autorización:
- MAY_MINT: Puede crear nuevos recursos

Y finalmente, *.no_initial_supply()*, no creamos inicialmente ningún suministro.

3. Pasamos a la estructura del componente:
    - La insignia (badge) que hemos creado para autorizar acuñar nuevos NFT, como tenemos el recurso (supply 1) en una variable, *minteador*, podemos utilizar el comando *with_bucket* para crea una bóveda vacía y la llenarla con un depósito inicial de recursos.
    - Pasamos la definición del recurso de NFT al componente.
    - Inicializamos la variable *num_afiliado* a cero. 

```rust
Self {
        minteador: Vault::with_bucket(minteador),
        def_afiliado_nft: def_afiliado,
        num_afiliado: 0
    }
    .instantiate()
```

### Método *afiliarse_dao*

Este método permite poder registrarse en la DAO como afiliado sin ningún tipo de autorización para ello, de forma totalmente libre. Solo ha de introducir un nombre de tipo string. 

El método al ejecutarse toma el nombre que se le ha indicado, añade 1 al num_afiliado para generar el nuevo número y finalmente acuña el NFT con esos dos datos: el numero afiliado como identificador y el nombre como dato inmutable. Finalmente el método devuelve un contenedor temporal con la acreditación de afiliado al DAO en forma de NFT a la cuenta que ha desencadenado este método. 


```rust
pub fn afiliarse_dao(&mut self, nombre: String) -> Bucket {
    self.num_afiliado += 1;

    self.minteador.authorize(|insignia| {
        self.def_afiliado_nft.mint_non_fungible(&NonFungibleKey::from(self.num_afiliado), DatosAfiliado{
            nombre: nombre
        }, insignia)
    })
}
```

Pasamos la autorización para acuñar mediante el contenedor permanente *minteador* de la siguiente manera:
```rust
self.minteador.authorize(|insignia| {
}
```

Creamos el NFT utilizando la definición *def_afiliado_nft* y el comando *mint_non_fungible*, pasamos los datos: identificador + datos y finalmente la autorización *insignia*.

### Código completo:

```rust
use scrypto::prelude::*;

#[derive(NonFungibleData)]
struct DatosAfiliado {
    nombre: String
}

blueprint! {
    struct SistemaAfiliacion {
        minteador: Vault,
        def_afiliado_nft: ResourceDef,
        num_afiliado: u128
    }

    impl SistemaAfiliacion {
        pub fn new() -> Component {
    
            let minteador = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
                            .metadata("name", "Autorización mintear nuevos NFT")
                            .initial_supply_fungible(1);

            let def_afiliado = ResourceBuilder::new_non_fungible()
                                .metadata("name", "Afiliado DAO")
                                .flags(MINTABLE | INDIVIDUAL_METADATA_MUTABLE)
                                .badge(minteador.resource_def(), MAY_MINT | MAY_CHANGE_INDIVIDUAL_METADATA)
                                .no_initial_supply();

            Self {
                minteador: Vault::with_bucket(minteador),
                def_afiliado_nft: def_afiliado,
                num_afiliado: 0
            }
            .instantiate()
        }


        pub fn afiliarse_dao(&mut self, nombre: String) -> Bucket {
            self.num_afiliado += 1;

            self.minteador.authorize(|insignia| {
                self.def_afiliado_nft.mint_non_fungible(&NonFungibleKey::from(self.num_afiliado), DatosAfiliado{
                    nombre: nombre
                }, insignia)
            })
        }
    }
}
```
### Compilación y ejecución

A estas alturas seguro que ya sabes publicar el *package*, instanciar el *component* y llamar a las funciones pasando un parámetro.

::: warning Recuerda
- Antes de empezar siempre es recomendable limpiar el simulador con el comando:
```
resim reset
```
:::

::: details Solo para aquellos (tipo Emilio 🤪) que no quieren pensar!!!
1. Limpiar el simulador
```rust
resim reset
```
2. Crear un Package
```
scrypto new-package dao
```
3. Crear una cuenta (recuerda copiar la dirección de los XRD de tu cuenta)
```rust
resim new-account
set acct [Address de la cuenta que acabamos de crear]
```
4. Copiar o escribir el código (recuerda guardar ctrl + s)
- Recuerda guardar el código de este ejercicio dentro del archivo lib.rs que has creado en la carpeta *\radixdlt-scrypto\dao\src\lib.rs*
5. Publicar y guardamos la dirección del Package
```rust
resim publish .
set pack [New Package Reference]
```
6. Instanciar componente (recuerda que en este caso hay que añadir el argumento del precio) y guardar la dirección del componente.
```rust
resim call-function $pack SistemaAfiliacion new
set comp [dirección del componente]
```
7. Probar método *afiliarse_dao*
```rust
resim call-method $comp afiliarse_dao "Emilio Bitcoin"
// Ojo que cuando pasamos un dato de tipo string y tiene mas de una palabra debemos entrecomillarlo.
```
8. Comprobar el nft de afiliado
```rust
resim show $acct
```
:::

::: danger Importante:
- Soy muy consciente de que hay muchas que no has entendido, ¡TRANQUILO!, no te rindas, las entenderás... 😉
:::