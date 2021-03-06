# Django - Migrations.

Las migraciones son la forma que tiene Django de propagar los cambios hechos en el esquema de los modelos de la aplicación.

Es una especie de versión de controles para las bases de datos. __makemigrations__ se encarga de empaquetar en un archivo _migrate_ todos los cambios realizados a los modelos, análogo a un _commit_, y __migrate__ es responsable de aplicar dichos cambios a la base de datos de la APP.

## Workflow.

Al ejecutar el comando `python manage.py makemigrations` el modelo, no la base de datos, será escaneado y comparado con la versión actual contenida en los archivos de migraciones, y luego será escrito un nuevo archivo de migración con los nuevos cambios detectados en el código de los modelos.

Una vez tengamos el nuevo archivo _migrate_ podremos aplicar los nuevos cambios a la base de datos utilizando el comando `python manage.py migrate`.

Una vez aplicada la migración es necesario hacer un _commit_ con la migración y el modelo modificado, para que así otras personas puedan tener los mismos cambios que nosotros y trabajar sobre los mismos modelos y la misma base de datos, pero esto solo después de cerciorarnos de que no existen conflictos, que el registro mantiene la coherencia y que las ramas están actualizadas.

Cuando dos ramas de desarrollo se unen, es posible que sea necesario unir las migraciones a mano, con el fin de evitar que se generen inconsistencias en la secuencia lineal que tienen las migraciones, ya que Django necesita que se siga la consistencia del historial.

los detalles a tener en cuenta a la hora de crear y aplicar migraciones son los siguiente: 

* Durante el trabajo será necesario crear múltiples migraciones para que Django puede ejecutarse correctamente, sin embargo hay que tener cuidado de no agregar estas nuevas migraciones a los _commits_ que creemos, ya que deberemos eliminarlas luego.

* Antes de hacer hacer _merge_ con la rama _develop_ (la cual procuraremos que siempre esté actualizada) podemos revertir todas las migraciones aplicadas hasta la ultima coincidente con la rama _develop_ utilizando el comando:

   `python manage.py migrate <app> <migration_code>` 

  para tener total seguridad de no generar ningún conflicto al volver a tener nuestra base de datos idéntica a la de la rama _develop_. 

* Al hacer _merge_, si la rama _develop_ tiene un nuevo archivo de migración, lo primero que debemos hacer será aplicar dicha migración, para luego crear un nuevo archivo de migración con nuestros nuevos cambios y volver a aplicar las migraciones, es decir, al hacer el merge y resolver los conflictos, utilizamos `python manage.py migrate`, aplicamos los nuevos cambios y ahora utilizamos `python manage.py makemigrations` para registrar nuestros nuevos cambios y posteriormente aplicarlos de nuevo con `python manage.py migrate`.

De este modo nos aseguramos de no romper la secuencia en el registro de migraciones y de evitar generar conflictos respecto a los cambios que puedan hacer otros miembros del equipo.



```mermaid
graph TD;
En_rama_de_trabajo --> Modificar_los_modelos
Modificar_los_modelos --> Revertir_nuevas_migraciones;
Revertir_nuevas_migraciones --> Eliminar_nuevas_migraciones;
Eliminar_nuevas_migraciones --> merge;
merge --> migrate;
migrate --> makemigrations;
makemigrations --> migrate;
makemigrations --> push;

En_rama_dev --> hacer_pull
hacer_pull --> checkout_rama_de_trabajo
checkout_rama_de_trabajo --> merge
```

---

```mermaid
sequenceDiagram
working_branch -->> working_branch: Modificar modelos.
working_branch -->> working_branch: Revertir migraciones nuevas.
working_branch -->> working_branch: Eliminar migraciones nuevas.
dev_branch -->> dev_branch: pull.
dev_branch ->> working_branch: merge.
working_branch -->> working_branch: Resolver conflictos.
working_branch -->> working_branch: migrate.
working_branch -->> working_branch: makemigrations.
working_branch -->> working_branch: migrate.
working_branch ->> dev_branch: push y merge request.
```



## Dependencias.

Si bien las migraciones son por aplicación, las tablas y relaciones implícitas en los modelos son demasiado complejas como para analizar una app a la vez, ya que si tenemos un modelo con una _ForeignKey_, se debe verificar la tabla referenciada primero, ya que si Django identifica que la tabla referenciada no existe retornará un error.

## Añadir migraciones a una aplicación especifica.

```shell
$ python manage.py makemigrations <app_name>
```

## Revertir migraciones.

```shell
$ python manage.py migrate <app_name> <migration_number>
```

Si tu intención es revertir todas las migraciones aplicadas para una aplicación, utiliza el comando `zero`. 

```shell
$ python manage.py migrate <app_name> zero
```

Si una migración contiene alguna operación irreversible, esta, al intentar revertirse, arrojará un `IrreversibleError`.

## [Reducir migraciones.](https://docs.djangoproject.com/en/4.0/topics/migrations/#squashing-migrations)

Django está optimizado para poder lidiar con cientos de migraciones sin que esto afecte al rendimiento, sin embargo, en el caso de que tengamos más migraciones de las que deseamos, es posible reducirlas al simplificar las operaciones incluidas en ellas y eliminando las que resulten redundantes o contradictorias. 

```shell
$ python manage.py squashmigrations <app_name> <app_number>
```

Este código reducirá todas las migraciones previas al numero que definamos, optimizando las operaciones y reduciendo su complejidad.

Una vez ejecutado el `squachmigrations` será necesario realizar la transición entre las migraciones antiguas hacia la nueva optimizada.

- Eliminar todas las migraciones que el nuevo archivo remplaza.
- Actualizar todas las migraciones que dependan de las migraciones eliminadas para que en su lugar dependan de la migración comprimida.
- Remover el atributo `replaces` en la clase `Migration` de la migración comprimida. Esta es la manera en la que Django comunica que es una migración comprimida.

> Una vez hayas comprimida una migración, no deber re-comprimirla hasta que hayas llevado a cabo toda la transición hacia una migración normal.

## [Serialización de valores.](https://docs.djangoproject.com/en/4.0/topics/migrations/#serializing-values)

Las migraciones en si mismas son una especie de registro que guarda la serialización de cada objeto dentro de un modelo, y puede serializar una gran cantidad de tipos de datos, sin embargo me limitará a señalar aquellos que NO puede serializar:

- Clases anidadas.
- Instancias arbitrarias de clases. 
- Lambdas o funciones anónimas.

> Si quieres saber todos los datos que las migraciones son capaces de serializar puedes recurrir a la documentación de Django.
>
> También es posible crear [serializadores personalizados](https://docs.djangoproject.com/en/4.0/topics/migrations/#custom-serializers).
