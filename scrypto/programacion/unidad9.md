# Lista de Boda II + NFT 

::: warning
- Esta unidad **NO** est谩 actualizada para Scrypto version 0.4 o mayor -> [Instrucciones actualizaci贸n](/scrypto/instalacion/actualizacion.md)
:::

::: tip 馃摵
- [Video Demostraci贸n](https://youtu.be/PCO0faacShw)
:::

Seguimos con el ejemplo de *Guestbook* enmarcado en una *lista de bodas* donde hasta el momento hemos podido mandar un mensaje a los novios y un regalo econ贸mico (XRD). El ejemplo de hoy no es m谩s que una vuelta de rosca a c贸digo que ya hemos escrito anteriormente 馃槑.

::: warning Opini贸n
- Al final el c贸digo es repetitivo ya que las soluciones suelen tener caracter铆sticas comunes. Lo mismo que una casa siempre tiene puertas, ventanas, paredes, luces... las aplicaciones siempre tienen elementos comunes que debemos conocer e incorporar de la forma mas 煤til, esto lo podemos lograr respetando y utilizando patrones de dise帽o.
:::

### An谩lisis
**Problema/An谩lisis:** 

Muchos novios dan detalles/regalos a sus invitados para que recuerden ese d铆a. Nos piden incorporar a nuestro *componente* la capacidad de generar un recuerdo NFT a todos aquellos que registren un mensaje. Y como en muchas ocasiones, pasado un tiempo, las bodas terminan en separaciones, queremos poder registrar ese dato de fin del enlace y que los NFT que se dieron como regalo pudieran actualizar la informaci贸n. Adem谩s nos piden mejorar algo el sistema de retirada de fondos para que de alguna manera la pareja llegue a un consenso para dicha retirada.

- Poder generar NFT con datos mutables e inmutables.
- Permitir guardar fecha de fin del enlace.

### Dise帽o
*Nota: En muchas ocasiones podemos realizar el dise帽o con psudoc贸digo que es una representaci贸n escrita de un algoritmo, es decir, muestra en forma de texto los pasos a seguir para solucionar un problema.*

- Funci贸n constructora
    - Crear insignia para administradores (novios)
    - Creer definici贸n NFT
    - Autorizaci贸n acu帽ar NFT
    - Crear Tabla de registros
    - Crear deposito XRD (regalos)
    - Fecha boda inicio y fin

- Agregar registros
    - Insertar a la tabla registros: nombre, comentario y regalo 
    - Guardar regalo en deposito
    - Acu帽ar NFT personalizado con el nombre del registro
    - Pasar a la cuenta del invitado

- Leer registros
    - Sacar por pantalla listado de registros

- Sacar XRD de componente
    - Buscar consenso para mandar el monto total de XRD (restringido al admin)

- Actualizar fecha fin boda
    - M茅todo para introducir fecha fin de boda (restringido al admin)

- Actualizar NFT
    - M茅todo para cambiar datos mutables en NFT (fecha boda fin)

### Programaci贸n
Como en ocasiones anteriores solo nos centraremos en los cambios realizados en el c贸digo con respecto a la [unidad anterior](/scrypto/programacion/unidad8.md) y las nuevas mejoras.

### La estructura:

```rust
#[derive(NonFungibleData)]
struct DatosNft {
    mensaje_detalle: String,
    fecha_boda: String,
    #[scrypto(mutable)]
    fecha_fin: String
}
```
Cuando creamos NFT podemos crear una estructura donde se guardaran los metadatos mutables e inmutables. Para ello Scrypto nos proporciona un *macro* espec铆fico: **#[derive(NonFungibleData)]**.

::: tip
- Por defecto todos los datos ser谩n inmutables.
- Si queremos que los datos sean mutables debemos escribir delante (en la l铆nea anterior) del dato **#[scrypto(mutable)]**
:::

En este caso el NFT que daran los novios como detalle por asistir a su boda y dejar un mensaje en este libro de visitas, contendr谩 un mensaje de los novios, la fecha del enlace, y la fecha de fin del matrimonio. Si te fijaste, este 煤ltimo dato, el de la fecha de fin, es mutable 馃檮.

```rust
blueprint! {
    struct ListaBoda {
        registro: HashMap<u128, (String,String,Decimal)>,
        regalos: Vault,
        boda_nft_def: ResourceDef,
        auth_nft: Vault,
        fecha_boda: String,
        fecha_fin: String,
        admin_def: ResourceDef,
        aprobacion: u8
    }
```
Con respecto a la unidad anterior, hemos agregado:
    1. Un contendedor (Vault) para guardar la insignia que tiene la autorizaci贸n para mintear nft en este componente.
    2. Declaramos dos variables para guardar las fechas de inicio y fin de boda, son de tipo String para ser mas sencillo su implementaci贸n desde el simulador. Cuando tengamos un front-end podremos cambiar este tipo de dato a uno mas concreto para guardar fechas. 
    3. Guardaremos en el componente la *Definici贸n" de la insignia de administrador con el fin de implementar un sistema de restricci贸n de accesos a ciertos m茅todos.
    4. Finalmente para poder crear un sistema, muy sencillito y primitivo, de consenso para sacar los XRD regalados guardamos la cantidad m铆nima de insignias que hemos creado como administradores para llegar a un consenso. 


### Funci贸n constructora *new*

```rust
pub fn new(descripcion_nft: String, fecha_boda: String) -> (Bucket, Component) {

    let admin: Bucket = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
        .metadata("name", "Admin Lista de Boda")
        .initial_supply_fungible(2);

    let auth_nft = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
                .metadata("name", "Autorizacion para crear nft boda")
                .initial_supply_fungible(1);

    let boda_nft_def = ResourceBuilder::new_non_fungible()
                        .metadata("name", descripcion_nft)
                        .flags(MINTABLE | INDIVIDUAL_METADATA_MUTABLE)
                        .badge(auth_nft.resource_def(), MAY_MINT | MAY_CHANGE_INDIVIDUAL_METADATA)
                        .no_initial_supply();
    
    let comp = Self {
        registro: HashMap::new(),
        regalos: Vault::new(RADIX_TOKEN),
        boda_nft_def,
        auth_nft: Vault::with_bucket(auth_nft),
        fecha_boda: String::from(fecha_boda),
        fecha_fin: String::new(),
        admin_def: admin.resource_def(),
        aprobacion: 2
    }
    .instantiate();

    (admin, comp)
}
```

En realidad en esta funci贸n constructora no hay nada que antes no hayamos codificado:
    1. Creamos un recurso fungible como control de acceso a ciertos m茅todos que solo podr谩n accionar los administradores de este componente.
    2. Creamos otro recurso fungible que, en esta ocasi贸n, actuar谩 como seguridad para la acu帽aci贸n y cambio de los metadatos de los NFT.
    3. Definiremos los NFT y guardarmos dicha definici贸n para despues poder acu帽ar y modificar los non fungible 'hijos' de esta definici贸n.
    4. Finalmente inicializamos las variables de la estructura. 


### M茅todo *nuevo*
```rust
pub fn nuevo(&mut self, nombre: String, comentario: String, mut regalo: Bucket) -> (Bucket, Bucket) {
    let mensaje = "Gracias por asistir a nuestra boda ".to_string() + &nombre;

    self.registro.insert(Uuid::generate(),(nombre,comentario, regalo.amount()));

    self.regalos.put(regalo.take(regalo.amount()));

    let nft_boda: Bucket = self.auth_nft.authorize(|badge| {
        self.boda_nft_def.mint_non_fungible(&NonFungibleKey::from(Uuid::generate()), DatosNft{
            mensaje_detalle: mensaje,
            fecha_boda: self.fecha_boda.to_string(),
            fecha_fin: String::new()
        }, badge)
    });

    (regalo, nft_boda)
}
```

Este m茅todo permite a los invitados crear un registro de visitas junto con un comentario y un regalo (XRD). Tales datos y recursos ser谩n enviados con la petici贸n, la cual recibir谩 de vuelta un NFT si todo va bien!!!

```rust
let mensaje = "Gracias por asistir a nuestra boda ".to_string() + &nombre;
```
Guardamos en una variable *mensajes* un mensaje prefijado junto con el nombre que la persona ha pasado como dato.

```rust
self.registro.insert(Uuid::generate(),(nombre,comentario, regalo.amount()));
```
Guardamos los datos que nos han pasado en la petici贸n como valor y como clave generamos un identificador 煤nico Uuid.
::: tip
- Para agregar nuevos registros en una colecci贸n como HashMap utilizamos el m茅todo **insert**.
:::
```rust
 self.regalos.put(regalo.take(regalo.amount()));
```
Tomamos (take) la cantidad completa del regalo en forma de XRD que fue enviado junto la petici贸n y lo guardamos (put) dentro del contenedor permanente *regalos*. Recuerda que con el m茅todo amount() podemos conocer la cantidad de un recurso dentro de un contenedor.

```rust
let nft_boda: Bucket = self.auth_nft.authorize(|badge| {
    self.boda_nft_def.mint_non_fungible(&NonFungibleKey::from(Uuid::generate()), DatosNft{
        mensaje_detalle: mensaje,
        fecha_boda: self.fecha_boda.to_string(),
        fecha_fin: String::new()
    }, badge)
});
```
Creamos el NFT a devolver gracias a que contamos con la autorizaci贸n guardad en el propio componente dentro del contenedor *auth_nft*. Tomamos la autorizaci贸n y la denominamos *badge* y ya podemos acu帽ar (mint_non_fungible) el NFT con la definici贸n que creamos (boda_nft_def) con la funci贸n *new*. Para acu帽ar es necesario primero generar/pasar un identificador que envolveremos dentro del tipo *NonFungibleKey*, que se ha creado Scrypto para facilitarnos este proceso. Segundo, pasaremos los datos mutables e inmutables al NFT que hemos dise帽ado en la estructura *DatosNft*. Y finalmente, tercero, utilizaremos la autorizaci贸n (*badge*) necesaria para poder generar el NFT.

```rust
(regalo, nft_boda)
```
Recuerda que no podemos dejar dentro del m茅todo ning煤n contenedor, los hemos de devolver todos o eliminar. En este caso devolvemos el contenedor *regalo* aunque este vacio y el contenedor que contiene el NFT generado *nft_boda*.

### M茅todo *fin_boda*
```rust
#[auth(admin_def)]
pub fn fin_boda(&mut self, fin: String) {
    self.fecha_fin = fin.to_string();
}
```
Este m茅todo viene precedido del macro *auth* que permite otorgar seguridad. En este caso estamos indicando que solo aquellos que posean una insignia con la definici贸n *admin_def* podr谩n ejecutar este m茅todo.

```rust
self.fecha_fin = fin.to_string();
```
Este m茅todo es sencillo, pasamos un String y actualizamos el dato *fecha_fin* que guardamos en el componente. El m茅todo *.to_string()* nos permite garantizar que el dato que pasamos es un dato tipo String. Recordemos que una de las claves de Rust es que permite un tipado fuerte.

### M茅todo *fin_boda*

```rust
pub fn actualizar_nft(&mut self, nft_boda: BucketRef) {
    let mut datos_nft: DatosNft = self.boda_nft_def.get_non_fungible_data(&nft_boda.get_non_fungible_key());
    datos_nft.fecha_fin = self.fecha_fin.to_string();

    self.auth_nft.authorize(|auth| self.boda_nft_def.update_non_fungible_data(&nft_boda.get_non_fungible_key(), datos_nft, auth));
}
```
Si el el componente ha actualizado el dato de fecha_fin de la boda, los NFT estar谩n desactualizados, con este m茅todo creamos un mecanismo por el cual alguien que quiera actualizarlos solo lo debe de ense帽ar su NFT para que el componente lo actualice. Recuerda que solo podremos actualizar los datos mutables de los recursos no fungibles.

```rust
let mut datos_nft: DatosNft = self.boda_nft_def.get_non_fungible_data(&nft_boda.get_non_fungible_key());
```

Primero extraemos los datos (get_non_fungible_data) del NFT para ello utilizaremos la referencia (nft_boda: BucketRef) del NFT que han pasado al realizar la llamada al m茅todo y utilizaremos su identificador (get_non_fungible_key()) . Guardamos los datos extraidos dentro de una variable mutable *datos_nft*.

```rust
 datos_nft.fecha_fin = self.fecha_fin.to_string();
```

Actualizamos el dato de *fecha_fin* que es el 煤nico mutable.

```rust
self.auth_nft.authorize(|auth| self.boda_nft_def.update_non_fungible_data(&nft_boda.get_non_fungible_key(), datos_nft, auth));
```
Finalmente actualizamos el NFT, para ello como ya hemos visto en unidades anteriores primero necesitamos la autorizaci贸n (auth) que se encuentra guardada dentro del contenedor *auth_nft*. Luego gracias al m茅todo de actualizaci贸n *update_non_fungible_data* que aplicamos sobre la definici贸n del NFT *boda_nft_def* podremos actualizar los datos del NFT que le indiquemos gracias a su indentificador 煤nico. 

### M茅todo *sacar_todo*

```rust
pub fn sacar_todo(&mut self, autorizacion: Bucket) -> (Bucket, Bucket) {
    assert_eq!(self.admin_def, autorizacion.resource_def(), "Debe entrar el recurso: Admin Lista de Boda");
    assert!(autorizacion.amount() == self.aprobacion, "No tienes el poder de consenso");
    (autorizacion, self.regalos.take_all())
}
```

Este m茅todo lo hemos actualizado incorporando un sencillo sistema de consenso o validaci贸n para que se solo quien posea las dos insignias creadas pueda retirar los fondos. Nota: La idea es que las insignias se repartan entre los contrayentes y as铆 solo si los dos est谩n de acuerdo pueden retirar los fondos. 

```rust
 assert_eq!(self.admin_def, autorizacion.resource_def(), "Debe entrar el recurso: Admin Lista de Boda");
```
Gracias al marco *assert_eq!* comprobamos si las definiciones coinciden. La definici贸n de la insignia *admin_def* con la que nos pasan en la llamada *autorizacion.resource_def()*.

```rust
 assert!(autorizacion.amount() = self.aprobacion, "No tienes el poder de consenso");
```
Aqu铆 validamos la cantidad de unidades de la insignia de administrador que est谩 pasando y comparamos con la cantidad que hemos declarado en el componente. En este caso hemos creado solo dos insignias y el consenso m铆nimo es del 100%.


### C贸digo completo:
```rust
use scrypto::prelude::*;

#[derive(NonFungibleData)]
struct DatosNft {
    mensaje_detalle: String,
    fecha_boda: String,
    #[scrypto(mutable)]
    fecha_fin: String
}

blueprint! {
    struct ListaBoda {
        registro: HashMap<u128, (String,String,Decimal)>,
        regalos: Vault,
        boda_nft_def: ResourceDef,
        auth_nft: Vault,
        fecha_boda: String,
        fecha_fin: String,
        admin_def: ResourceDef,
        aprobacion: u8
    }

    impl ListaBoda {
        pub fn new(descripcion_nft: String, fecha_boda: String) -> (Bucket, Component) {

            let admin: Bucket = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
                .metadata("name", "Admin Lista de Boda")
                .initial_supply_fungible(2);

            let auth_nft = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
                        .metadata("name", "Autorizacion para crear nft boda")
                        .initial_supply_fungible(1);

            let boda_nft_def = ResourceBuilder::new_non_fungible()
                                .metadata("name", descripcion_nft)
                                .flags(MINTABLE | INDIVIDUAL_METADATA_MUTABLE)
                                .badge(auth_nft.resource_def(), MAY_MINT | MAY_CHANGE_INDIVIDUAL_METADATA)
                                .no_initial_supply();
            
            let comp = Self {
                registro: HashMap::new(),
                regalos: Vault::new(RADIX_TOKEN),
                boda_nft_def,
                auth_nft: Vault::with_bucket(auth_nft),
                fecha_boda: String::from(fecha_boda),
                fecha_fin: String::new(),
                admin_def: admin.resource_def(),
                aprobacion: 2
            }
            .instantiate();

            (admin, comp)
        }

        pub fn nuevo(&mut self, nombre: String, comentario: String, mut regalo: Bucket) -> (Bucket, Bucket) {
            let mensaje = "Gracias por asistir a nuestra boda ".to_string() + &nombre;

            self.registro.insert(Uuid::generate(),(nombre,comentario, regalo.amount()));

            self.regalos.put(regalo.take(regalo.amount()));

            let nft_boda: Bucket = self.auth_nft.authorize(|badge| {
                self.boda_nft_def.mint_non_fungible(&NonFungibleKey::from(Uuid::generate()), DatosNft{
                    mensaje_detalle: mensaje,
                    fecha_boda: self.fecha_boda.to_string(),
                    fecha_fin: String::new()
                }, badge)
            });

            (regalo, nft_boda)
        }

        #[auth(admin_def)]
        pub fn fin_boda(&mut self, fin: String) {
            self.fecha_fin = fin.to_string();
        }

        pub fn actualizar_nft(&mut self, nft_boda: BucketRef) {
            let mut datos_nft: DatosNft = self.boda_nft_def.get_non_fungible_data(&nft_boda.get_non_fungible_key());
            datos_nft.fecha_fin = self.fecha_fin.to_string();

            self.auth_nft.authorize(|auth| self.boda_nft_def.update_non_fungible_data(&nft_boda.get_non_fungible_key(), datos_nft, auth));
        }

        pub fn leer_registros(&self) {
            let mut total_regalos = Decimal::zero();
            info!("Comentarios:");
            info!("--------------------------------------------------");
            for (_uuid, comentario) in &self.registro {
                info!("{:?}", comentario);
                total_regalos += comentario.2;
            };
            info!("--------------------------------------------------");
            info!("Total: {} xrd regalados" , total_regalos);
        }

        pub fn sacar_todo(&mut self, autorizacion: Bucket) -> (Bucket, Bucket) {
            assert_eq!(self.admin_def, autorizacion.resource_def(), "Debe entrar el recurso: Admin Lista de Boda");
            assert!(autorizacion.amount() == self.aprobacion, "No tienes el poder de consenso");
             (autorizacion, self.regalos.take_all())
        }
    }
}
```

### Compilaci贸n y ejecuci贸n
A estas alturas seguro que ya sabes publicar el package, instanciar el component y llamar a las funciones pasando un par谩metro.

::: details Pero... por si eres un Homo Emilius te dejo una ayudita 馃檲.
1. Limpiar el simulador
```rust
resim reset
```
2. Crear un Package
```
scrypto new-package LibroBoda
cd LibroBoda
```
3. Crear una cuenta (recuerda copiar la direcci贸n de los XRD de tu cuenta)
```rust
resim new-account
set acct [Address de la cuenta que acabamos de crear]
set pub [Guardamos la clave p煤blica de esta cuenta]
set xrd 030000000000000000000000000000000000000000000000000004
```
4. Copiar o escribir el c贸digo (recuerda guardar ctrl + s)
- Recuerda guardar el c贸digo de este ejercicio dentro del archivo lib.rs que has creado en la carpeta *\radixdlt-scrypto\LibroBoda\src\lib.rs*
5. Publicar y guardamos la direcci贸n del Package
```rust
resim publish .
set pack [New Package Reference]
```
6. Instanciar componente (recuerda que en este caso hay que a帽adir el argumento del precio) y guardar la direcci贸n del componente.
```rust
resim call-function $pack ListaBoda new "Boda de Miriam y Esther" "Dos de febrero de 2020"   
set comp [direcci贸n del componente]
```
7. Probar m茅todo *nuevo*
```rust
resim call-method $comp nuevo "Emilio Bitcoin" "Que boda mas bonita, os deseo muchos hijos..." 1000,$xrd
// Ojo que cuando pasamos un dato de tipo string y tiene mas de una palabra debemos entrecomillarlo.
// Podemos comprobar el NFT que nos han regalado
resim show $acct
```
8. Probar m茅todo *leer_registros*
```rust
resim call-method $comp leer_registros
```
9. Sacar los regalos del componente a la billetera que lo ejecuta
```rust
resim call-method $comp sacar_todo 2,[definici贸n del recurso fungible Admin Lista de Boda]
```
10. Comprobamos que hemos recibido los xrd
```rust
resim show $acct
```
11. Actualizamos la fecha de fin de boda
```rust
resim call-method $comp fin_boda "22 de febrero de 2025" 1,[definici贸n del recurso fungible Admin Lista de Boda]
```
12. Actualizamos el NFT
```rust
resim call-method $comp actualizar_nft 1,[definici贸n del nft que nos han regalado]
```
:::

馃槑 驴Qu茅 tal? 驴Como te sientes despu茅s de 10 unidades de aprendizaje? Yo solo puede decir que para mi ha sido un verdadero honor  haberte ayudado a emprender una ruta fant谩stica. El camino de la programaci贸n es un sendero de continuo aprendizaje de mucha practica e ilusi贸n por logra vencer los retos que nos marcamos. Hoy puedes decir que conoces los fundamentos de Radix y Scrypto, puedes decir que puedes leer, ejecutar e incluso escribir tus propios componentes. Hoy puedes decir que te has iniciado en la programaci贸n funcional de la mano de Rust. De verdad que me emociona pensar que esta academia ha sido tu punto de partida para amar como yo la programaci贸n y el dise帽o de tecnolog铆a aplicada. 

驴Te atreves? a seguir aprendiendo, e incluso realizando tus primeras pr谩cticas. Solo cuando empieces por ti mismo a realizar tus propios algoritmos podr谩s decir que eres un programador junior en Scrypto. 

驴Te apetece? Pues te invito a resolver una peque帽a [prueba de nivel](/ruta/basico/unidad10.md) que demuestre que has asimilado algunos conocimientos de programaci贸n y Scrypto. Solo as铆 podr谩s acceder al nivel intermedio donde empezar谩 lo verdaderamente emocionante!!!! No te arrepentir谩s...

Hagas lo que hagas... gracias, mil gracias por llegar hasta aqu铆. Solo las personas verdaderamente valientes terminan lo que empiezan y tu, sin duda, eres uno de ellos. De verdad gracias, para mi fue un placer.


