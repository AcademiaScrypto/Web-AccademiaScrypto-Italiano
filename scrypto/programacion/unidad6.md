# Creaci贸n de DAO - Nuevos miembros

::: warning
- Esta unidad **NO** est谩 actualizada para Scrypto version 0.4 o mayor -> [Instrucciones actualizaci贸n](/scrypto/instalacion/actualizacion.md)
:::

::: tip 馃摵
- [Video Demostraci贸n](https://youtu.be/wFywZeDOcw8)
:::

Durante las siguientes unidades vamos a crear una DAO (Organizaci贸n aut贸noma descentralizada), que sin duda durante 2022 est谩n llamadas a ser tendencia y seguro que han llegad1o para quedarse entre nosotros. Aprender a crear DAOs puede tener un gran potencial. 隆Comencemos!

### An谩lisis
**Problema:** Deseamos crear una DAO donde los afiliado van a poder darse de alta sin ning煤n tipo de autorizaci贸n. 

**An谩lisis:** Al pedirnos una DAO ya estamos dando por supuesto que vamos a crear una dApp. Para ello vamos a crear un *componente*, dentro de la red de Radix, que nos va a permitir dar un NFT a cada nuevo afiliado para que pueda demostrar su pertenencia y en un futuro identificarse dentro de la DAO.

### Dise帽o:
![Diagramas](/diagramas_unidad6.png)

Dos diagramas: uno para la funci贸n constructora y el segundo para la primera funcionalidad que nos solicitan y es la de poder registrarse cualquiera al DAO retornando un NFT como acreditaci贸n de su pertenencia al DAO. *(Nota: en cuanto m谩s lenguaje m谩s natural expresemos en estos diagramas mejor, yo a veces peco de incluir t茅rminos t茅cnicos propios del lenguaje o plataforma que utilice)*

### Programaci贸n:

### NFT Estructura Datos:

Como ya vimos en la [unidad anterior](/scrypto/programacion/unidad5.md) *Scrypto* pone a disposici贸n una macro para poder crear una estructura de datos, mutables e inmutables, que luego podremos incorporar a nuestro NFT:

```rust
#[derive(NonFungibleData)]
struct DatosAfiliado {
    nombre: String
}
```

En este caso hemos llamado a la estructura: *DatosAfiliado* (recuerda, es algo arbitrario) y de momento solo le hemos implementado un dato de tipo *String* (Cadena de caracteres) para guardar el nombre del afiliado (sera un dato inmutable).

### La estructura:

Vamos a necesitar crear un contador para guardar el n煤mero de afiliado, lo normal es que esta vez sea un n煤mero legible y no uno aleatorio, y que a su vez nos har谩 de identificador del NFT. Para poder utilizar el 'envoltorio' de identificadores [*NonFungibleKey*](https://radixdlt.github.io/radixdlt-scrypto/scrypto/types/struct.NonFungibleKey.html#) tenemos que crear un dato de tipo u128.

::: tip
|DATA TYPE   |MIN    |MAX |
| ---------- |-------|----|
|u8	         |0      |255 |
|u16         |0      |65535 |
|u32         |0      |4294967295 |
|u64	     |0      |18446744073709551615 |
|u128        |0      |340282366920938463463374607431768211455 |
:::

Luego necesitaremos guardar la definici贸n, con las caracter铆sticas,  del tipo de NFT que queremos dar a cada afiliado, sera de tipo ResourceDef.

Y de momento solo nos quedar铆a a帽adir un contenedor permanente (Vault) para guardar una *insignia* (badge) que autorice al componente para mintear nuevos NFT. En este caso la insignia con autorizaci贸n para acu帽ar se guardara en el propio componente para que la utilice cuando sea necesario. 

```rust
struct SistemaAfiliacion {
        minteador: Vault,
        def_afiliado_nft: ResourceDef,
        num_afiliado: u128
}
```

### Funci贸n constructora *new*

1. Primero vamos a crear la autorizaci贸n para crear NFT:
```rust
let minteador = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
                        .metadata("name", "Autorizaci贸n mintear nuevos NFT")
                        .initial_supply_fungible(1);
```
Creamos un recurso con, *ResourceBuilder*, de tipo fungible con el par谩metro *DIVISIBILITY_NONE* que impide fraccionar el recurso. Un nombre que lo identifique, es especialmente interesante para poder identificarlo cuando realizamos *resim show*. Y finalmente un suministro de 1.

2. Creamos la definici贸n del NFT que vamos a entregar a cada persona que se registre en el DAO:

```rust
let def_afiliado = ResourceBuilder::new_non_fungible()
                            .metadata("name", "Afiliado DAO")
                            .flags(MINTABLE)
                            .badge(minteador.resource_def(), MAY_MINT)
                            .no_initial_supply();
```
En este caso el recurso ser谩 de tipo no fungible, *new_non_fungible*, y tiene varios *flags* que restringen sus caracter铆sticas:
- MINTABLE: Se puede acu帽ar m谩s del recurso

Seguidamente indicamos al recurso que va a tener una insignia, *.badge*, la definici贸n de la insignia y a que cosas tiene autorizaci贸n:
- MAY_MINT: Puede crear nuevos recursos

Y finalmente, *.no_initial_supply()*, no creamos inicialmente ning煤n suministro.

3. Pasamos a la estructura del componente:
    - La insignia (badge) que hemos creado para autorizar acu帽ar nuevos NFT, como tenemos el recurso (supply 1) en una variable, *minteador*, podemos utilizar el comando *with_bucket* para crea una b贸veda vac铆a y la llenarla con un dep贸sito inicial de recursos.
    - Pasamos la definici贸n del recurso de NFT al componente.
    - Inicializamos la variable *num_afiliado* a cero. 

```rust
Self {
        minteador: Vault::with_bucket(minteador),
        def_afiliado_nft: def_afiliado,
        num_afiliado: 0
    }
    .instantiate()
```

### M茅todo *afiliarse_dao*

Este m茅todo permite poder registrarse en la DAO como afiliado sin ning煤n tipo de autorizaci贸n para ello, de forma totalmente libre. Solo ha de introducir un nombre de tipo string. 

El m茅todo al ejecutarse toma el nombre que se le ha indicado, a帽ade 1 al num_afiliado para generar el nuevo n煤mero y finalmente acu帽a el NFT con esos dos datos: el numero afiliado como identificador y el nombre como dato inmutable. Finalmente el m茅todo devuelve un contenedor temporal con la acreditaci贸n de afiliado al DAO en forma de NFT a la cuenta que ha desencadenado este m茅todo. 


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

Pasamos la autorizaci贸n para acu帽ar mediante el contenedor permanente *minteador* de la siguiente manera:
```rust
self.minteador.authorize(|insignia| {
}
```

Creamos el NFT utilizando la definici贸n *def_afiliado_nft* y el comando *mint_non_fungible*, pasamos los datos: identificador + datos y finalmente la autorizaci贸n *insignia*.

### C贸digo completo:

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
                            .metadata("name", "Autorizaci贸n mintear nuevos NFT")
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
### Compilaci贸n y ejecuci贸n

A estas alturas seguro que ya sabes publicar el *package*, instanciar el *component* y llamar a las funciones pasando un par谩metro.

::: warning Recuerda
- Antes de empezar siempre es recomendable limpiar el simulador con el comando:
```
resim reset
```
:::

::: details Solo para aquellos (tipo Emilio 馃お) que no quieren pensar!!!
1. Limpiar el simulador
```rust
resim reset
```
2. Crear un Package
```
scrypto new-package dao
```
3. Crear una cuenta (recuerda copiar la direcci贸n de los XRD de tu cuenta)
```rust
resim new-account
set acct [Address de la cuenta que acabamos de crear]
```
4. Copiar o escribir el c贸digo (recuerda guardar ctrl + s)
- Recuerda guardar el c贸digo de este ejercicio dentro del archivo lib.rs que has creado en la carpeta *\radixdlt-scrypto\dao\src\lib.rs*
5. Publicar y guardamos la direcci贸n del Package
```rust
resim publish .
set pack [New Package Reference]
```
6. Instanciar componente (recuerda que en este caso hay que a帽adir el argumento del precio) y guardar la direcci贸n del componente.
```rust
resim call-function $pack SistemaAfiliacion new
set comp [direcci贸n del componente]
```
7. Probar m茅todo *afiliarse_dao*
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
- Soy muy consciente de que hay muchas que no has entendido, 隆TRANQUILO!, no te rindas, las entender谩s... 馃槈
:::