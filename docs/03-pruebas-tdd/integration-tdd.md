# Práctica 3: Integración con GitHub Actions y TDD

En esta práctica 3 de la asignatura realizaremos dos tareas principales:

- Configuraremos un sistema de integración continua usando las
  _actions_ del repositorio de GitHub. En este sistema se lanzarán los
  tests automáticamente en cada _pull request_. Después definiremos
  una nueva configuración del proyecto en la que se lanzarán los tests
  sobre la base de datos [PostgreSQL](https://www.postgresql.org).
- Añadiremos nuevas funcionalidades usando la práctica XP de TDD
  (_Test Driven Design_).

!!! Important "Importante"
    Lee con cuidado todo el enunciado y dedica especial atención a los
    apartados con el título `Pasos a seguir`. Ahí están
    especificadas las acciones que debes realizar en la práctica.

La duración de la práctica es de 3 semanas y la fecha límite de
entrega es el día 9 de noviembre.

## 1. Desarrollo de la _release_ 1.2.0 ##

En esta práctica vamos a desarrollar la versión 1.2.0 de la aplicación
`ToDoList`. A todos los _issues_ y _pull requests_ les debes poner este
_milestone_, indicando que el objetivo es resolverlos y entregarlos en
esta _release_.

### Pasos a seguir ###

- Cambia el número de versión (en el fichero _Acerca De_ y en el
  `pom.xml`) a `1.2.0-SNAPSHOT` para indicar que lo que hay en main
  es la versión 1.2.0 **en progreso**. Esta versión la lanzaremos al
  final del desarrollo de la práctica, en su entrega.


## 2. Integración continua con GitHub Actions ##

[GitHub
Actions](https://docs.github.com/en/free-pro-team@latest/actions) es
un servicio de GitHub que permite realizar integración continua
en su propia web, sin necesidad de configurar un servidor propio de integración
continua.

Puedes consultar el funcionamiento de GitHub Actions leyendo su documentación,
comenzando por las páginas [Quickstart for GitHub
Actions](https://docs.github.com/en/free-pro-team@latest/actions/quickstart),
[Introduction to GitHub
Actions](https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions/introduction-to-github-actions)
y [Building and testing Java with
Maven](https://docs.github.com/en/free-pro-team@latest/actions/guides/building-and-testing-java-with-maven). 

En la práctica vamos a comenzar configurando GitHub Actions para que
todos los _pull requests_ deban pasar los tests de integración antes
de realizar el _merge_ con `main`.

### Tests en los pull requests ###

Usando GitHub Actions es posible configurar el repositorio de GitHub
para que todos los _pull requests_ deban pasar los tests de
integración en el servicio.

En la siguiente imagen vemos el aspecto en GitHub de un _pull request_
estando activa la integración con Actions. Una vez abierto el PR,
se lanzan los flujos de trabajo (_workflows_) definidos en 
el directorio `.github/workflows`. 

GitHub comprueba si la integración de main con la rama pasa los tests
definidos en el workflow. Sólo si los tests pasan es posible realizar
el _merge_ del PR en main.

<img src="imagenes/pull-request-actions.png" width="600px"/>


### El fichero de configuración  ###

Para configurar GitHub Actions basta con añadir un fichero de flujo de
trabajo en el directorio `.github/workflows`.

El fichero con el flujo de trabajo inicial lo llamaremos `developer-tests.yml`:

**Fichero `.github/workflows/developer-tests.yml`**

```yml
name: Tests

on: push

jobs:
  # El nombre del job es launch-test
  launch-tests:
    runs-on: ubuntu-latest
    # Todos los pasos se ejecutan en el contenedor openjda:8-jdk-alpine
    container: openjdk:8-jdk-alpine

    steps:
      # Hacemos un checkout del código del repositorio 
      - uses: actions/checkout@v2
      # Y lanzamos los tests
      - name: Launch tests with Maven
        run:  ./mvnw test
```

Puntos interesantes a destacar:

- El nombre del flujo de trabajo es `Tests`.
- El nombre del job es `launch-tests`.
- Con la palabra clave `on` se define el evento que causa que se lance
  el flujo de trabajo. Es en cualquier commit subido a GitHub  (push).
- En `jobs` se definen los trabajos en paralelo a realizar por el
  flujo de trabajo. En nuestro caso sólo habrá uno.
- En `runs-on` se define la máquina base sobre la que se van a
  ejecutar los siguientes pasos del flujo.
- En `container` se especifica el contenedor Docker que se va a usar
  para ejecutar los pasos del flujo de trabajo.
- En `uses: actions/checkout@v2` se especifica que se obtenga la
  versión 2 de la acción llamada `actions/checkout`. Esta acción se
  descarga el repositorio en la máquina especificada anteriormente y
  lo deja listo para ejecutar los tests o cualquier otra acción.
- Por último, con el comando `run: ./mvnw test` se indica que el flujo
  de trabajo debe lanzar este comando, que es el que lanza los
  tests. 
- Los nombres `Tests` y `launch-tests` son nombres arbitrarios que
  indicamos y que después aparecen en la interfaz de Actions y nos
  sirven para localizar los distintos pasos.
  

### Builds en Actions ###

En la pestaña `Actions` de GitHub tenemos toda la información de los
_builds_. Es posible visualizarla mientras que se está realizando el
build o cuando ya ha terminado. Allí podremos ver el detalle de la
ejecución de los tests y consultar la salida de los mismos para
comprobar su resultado.

En la siguiente imagen se ha capturado el build en ejecución. El color
naranja significa que el proceso está en ejecución.

<img src="imagenes/builds-github-actions.png" width="600px"/>


### Pasos a seguir ###

- Crea un _issue_ llamado `Integración continua con GitHub
  Actions`. Abre una rama `integracion-continua-actions`, súbela a GitHub y abre un pull
  request.

- Añade el fichero `.github/workflows/developer-tests.yml`. Haz un
  commit y súbelo a GitHub.

- Comprueba que se pasan los tests y que se marca como correcto el
  _pull request_.
  
- Modifica un test para que falle y sube un nuevo commit. Comprueba
  que el commit aparece como erróneo en GitHub cuando el _build_
  falla. Vuelve a realizar los cambios para corregirlos, vuelve a
  subir el commit y comprueba que el nuevo commit y el PR pasan
  correctamente.

- Cierra el _pull request_, mezclándolo con `main`. Se volverán a
  lanzar los tests en GitHub y el commit aparecerá marcado como
  correcto. Baja los cambios al repositorio local y borra la rama.

    ```
    $ (integracion-continua-actions) git checkout main
    $ (main) git pull
    $ (main) git branch -d integracion-continua-actions
    $ (main) git remote prune origin
    ```



## 3. Configuración de la aplicación para usar una BD PostgreSQL ##

Hasta ahora hemos trabajado con la aplicación en una configuración
local con nuestro ordenador de desarrollo trabajando sobre una base de
datos H2 en memoria. Pero el objetivo final es poner la aplicación en
producción, en un servidor en Internet y usando una base de datos
PostgreSQL en producción. 

Además, te habrás dado cuenta de que es muy engorroso probar la
aplicación con la base de datos de memoria. Tienes que volver a
introducir todos los datos de prueba cada vez que paramos y ponemos en
marcha la aplicación.

En esta práctica vamos a ver cómo configurar la aplicación para poder
trabajar con una base datos PostgreSQL, tanto en su ejecución como en
los tests.

Para configurar la aplicación vamos a utilizar los denominados
_perfiles_. Definiremos, además del perfil base, un perfil adicional
para lanzar la aplicación y los tests usando la base de datos
PostgreSQL.

La configuración de tests con base de datos PostgreSQL la utilizaremos
para ejecutar los tests de integración sobre la base de datos PostgreSQL
en el proceso de integración continua de GitHub Actions.

Para lanzar un servidor de base de datos PostgreSQL usaremos Docker, de
forma que no tendremos que realizar ninguna instalación en nuestro
ordenador.

<img src="imagenes/app-contenedor.png" width="500px" />

### Ficheros de configuración de la aplicación ###

Ya hemos comentado que la configuración de la aplicación se define en
el fichero `application.properties`. Ahí se definen distintas
propiedades de la ejecución de la aplicación que se pueden modificar
(puerto en el que se lanza la aplicación, base de datos con la que
conectarse, etc.). 

Tenemos dos ficheros `application.properties`: uno en el directorio
`src/main/resources` que define la configuración de ejecución y otro
en el directorio `src/test/resources` que define la configuración que
se carga cuando se lanzan los tests.

Spring Boot permite definir ficheros de configuración adicionales
que pueden **sobreescribir y añadir propiedades** a las definidas en el
fichero de configuración por defecto. El nombre de estos ficheros de
configuración debe ser `application-xxx.properties` donde `xxx` define
el nombre del perfil. En nuestro caso definiremos los ficheros
`application-postgres.properties` (uno en el directorio `main` y otro
en `test`) para definir las configuraciones de ejecución y de test con
PostgreSQL.

Estos ficheros de configuración adicionales se cargan después de
cargar la configuración por defecto definida en `application.properties`.

### Pasos a seguir ###

1. Crea un nuevo _issue_ llamado `Añadir perfiles y permitir trabajar
  con PostgreSQL`. Crea una rama nueva (llámala `perfiles`, por ejemplo) y
  abre un pull request. 
  
    ```
    $ (main) git checkout -b perfiles
    $ (perfiles) git push -u origin perfiles
    ```

2. Copia el siguiente fichero en `src/main/resources/application-postgres.properties`:

    ```
    spring.datasource.url=jdbc:postgresql://localhost:5432/mads
    spring.datasource.username=mads
    spring.datasource.password=mads
    spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.PostgreSQL9Dialect
    ```

    Este va a ser el perfil que activemos para utilizar la conexión
    con la BD PostgreSQL.
    
    En este fichero de configuración se define la URL de conexión a la
    base de datos `mads`, su usuario (`mads`) y contraseña (`mads`) y
    el dialecto que se va a utilizar para trabajar desde JPA con la
    base de datos (`org.hibernate.dialect.PostgreSQL9Dialect`).
   
    La propiedad `spring.datasource.initialization-mode=never` indica
    que no se debe cargar ningún fichero de datos inicial. Por esto,
    el fichero `data.sql` no se va a cargar en la base de datos,
    deberás registrar un usuario inicial para poder probar la
    aplicación. La ventaja es que al trabajar con la base de datos
    real todos los datos van a quedar grabados aunque se pare la
    aplicación.
    
3. Vamos ahora a añadir el perfil de test. Copia el siguiente fichero
  en `src/test/resources/application-postgres.properties`:
    
    ```
    spring.datasource.url=jdbc:postgresql://localhost:5432/mads_test
    spring.datasource.username=mads
    spring.datasource.password=mads
    spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.PostgreSQL9Dialect
    ```
     
    En este perfil la conexión se hace con una base de datos
    diferente: `mads_test`, para no sobreescribir la base de datos
    definida en el perfil de ejecución.

    Recuerda que en el perfil por defecto
    `resources/application.properties` se define el valor de
    `spring.jpa.hibernate.ddl-auto` como `create`. De esta forma la
    base de datos se inicializa antes de cargar los datos de los tests
    y de ejecutarlos. También usamos una base de datos distinta
    (`mads_test`) para no sobreescribir la base de datos definida en
    el perfil de ejecución.

4. Añade la siguiente dependencia en el fichero `pom.xml` para que se
   descargue el driver `postgresql:42.2.22`. También añade las líneas
   para poder especificar perfiles desde línea de comando. La variable
   `profiles` se definirá desde línea de comando cuando se llame a
   Maven:

    Fichero `pom.xml`:

    ```xml hl_lines="4 5 6 7 8 17 18 19"
                <artifactId>h2</artifactId>
                 <scope>runtime</scope>
             </dependency>
             <dependency>
                 <groupId>org.postgresql</groupId>
                 <artifactId>postgresql</artifactId>
                 <version>42.2.22</version>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>

            ...
            
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <configuration>
                        <profiles>${profiles}</profiles>
                    </configuration>
                </plugin>
    ```

5. Para lanzar la aplicación necesitarás un servidor PostgreSQL en el
   puerto 5432 con el usuario `mads`, la contraseña `mads` y la base
   de datos `mads`. Es muy sencillo descargarlo y ejecutarlo si tienes
   instalado Docker. Ejecuta desde el terminal:

    ```
    docker run -d -p 5432:5432 --name postgres-develop -e POSTGRES_USER=mads -e POSTGRES_PASSWORD=mads -e POSTGRES_DB=mads postgres:13
    ```

    Docker se descarga la imagen `postgres:13` y lanza el contenedor (una
    instancia en marcha de una imagen) conectado al puerto 5432 (no debe
    estar ocupado) y sobre la base de datos `mads`. Le da como nombre
    `postgres-develop`.
   
    Puedes ejecutar los siguientes comandos de Docker:
    
      ```
      $ docker container ls -a (comprueba todos los contenedores en marcha)
      $ docker container stop <nombre o id de contenedor> (para un contenedor)
      $ docker container start <nombre o id de contenedor> (pone en marcha un contenedor)
      $ docker container logs <mombre o id de contenedor> (muestra logs del contenedor)
      $ docker container rm nombre o id de contenedor> (elimina un contenedor)
      ```

6. Arranca la aplicación con el siguiente comando:

    ```
    ./mvnw spring-boot:run -D profiles=postgres
    ```

    Se activará el perfil `postgres` y se cargarán las preferencias de
    `src/main/resource/application.properties` y
    `src/main/resource/application-postgres.properties`.

    Prueba a introducir datos en la aplicación y comprueba que se
    están guardando en la base de datos utilizando por ejemplo el
    panel `Database` de _IntelliJ_:

    <img src="imagenes/panel-database.png" width="700px"/>

    También podemos arrancar la aplicación con el perfil de postgres
    lanzando directamente el fichero JAR de la siguiente forma:
    
    ```
    $ ./mvnw package
    $ java -Dspring.profiles.active=postgres -jar target/*.jar 
    ```

    Para lanzar la aplicación desde _IntelliJ_ trabajando con el nuevo
    perfil podemos seleccionar la opción `Edit Configurations...` del
    menú de configuraciones, duplicar la configuración `Application`,
    renombrándola por `Application PostgreSQL` y añadir en el campo
    `Active profiles` el nombre del perfil nuevo que acabamos de crear
    `postgres`.

7. Cierra la aplicación y vuelve a abrirla. Comprueba que los datos
   que se han creado en la ejecución anterior siguen estando. 
   
    Podemos también parar el contenedor y volverlo a reiniciar y los
    datos se conservarán. Al parar el contenedor no se eliminan los
    datos, sólo al borrarlo.

8. Cierra la aplicación. Paramos el contenedor con la base de datos de
   desarrollo haciendo `docker container stop postgres-develop`:

    ```
    $ docker container ls -a 
    CONTAINER ID        IMAGE        ...    NAME
    520fee61d51e        posgres:13   ...    postgres-develop
    $ docker container stop postgres-develop
    ```

    Además de por línea de comando, también es posible gestionar los
    contenedores usando la aplicación _Docker Desktop_ que se
    encuentra en la propia instalación de Docker.

9. Vamos ahora a ver cómo lanzar los tests sobre una base de datos
   PostgreSQL. Lanzamos ahora otro contenedor con la base de datos de test (`mads_test`):

    ```
    docker run -d -p 5432:5432 --name postgres-test -e POSTGRES_USER=mads -e POSTGRES_PASSWORD=mads -e POSTGRES_DB=mads_test postgres:13
    ```

    Y lanzamos los tests usando el perfil `postgres` con la base de datos PostgreSQL con el siguiente comando:
  
      ```
      ./mvnw -D spring.profiles.active=postgres test
      ```
  
10. Podemos lanzar también los tests desde _IntelliJ_ editando la
    configuración de lanzamiento de test y añadiendo la variable de
    entorno `spring.profiles.active=postgres`. Podríamos, por ejemplo,
    llamar a esta configuración `Tests con PostgreSQL`.

11. Dado que las configuraciones de test y de ejecución utilizan
    distintas bases de datos, debemos tener en funcionamiento la base
    de datos correspondiente a lo que queremos hacer en cada
    momento. Esto es muy fácil usando los contenedores de Docker. Por
    ejemplo, podemos parar el contenedor PostgreSQL con la base de datos
    de test y arrancar el contenedor con la base de datos de
    desarrollo:
  
    ```
    $ docker container ls -a 
    $ docker container stop postgres-test
    $ docker container start postgres-develop
    ```

12. Realiza un commit con los cambios, súbelos a la rama y cierra el
    pull request para integrarlo en `main`:
  
      ```
      $ (perfiles) git add .
      $ (perfiles) git commit -m "Añadidos perfiles para trabajar con PostgreSQL"
      $ (perfiles) git push
      // Mezclamos el Pull Request en GitHub
      $ (perfiles) git checkout main
      $ (main) git pull
      $ (main) git branch -d perfiles
      $ (main) git remote prune origin
      ```

## 4. Tests de integración en GitHub Actions ##

Vamos a modificar la configuración de GitHub Actions para conseguir un
sistema de integración continua que ejecute los tests de integración
usando la base de datos real PostgreSQL.

La ejecución de los tests usando la base de datos de memoria H2 será
responsabilidad del desarrollador y se hará en el entorno de trabajo
local, tal y como se ha hecho desde la primera práctica.

### Tests del desarrollador vs. tests de integración ###

Podemos considerar los tests que usan la base de datos real como
_tests de integración_ y los tests que usan la base de datos en
memoria como _tests del desarrollador_.

No usamos el nombre de _tests unitarios_ de forma consciente, para
evitar conflictos con la nomenclatura. Cuando hablamos de _tests del
desarrollador_ nos referimos a tests que van a ejecutar continuamente
los desarrolladores en su equipo local cuando están trabajando con la
aplicación y añadiendo funcionalidades. Son tests rápidos, que se
pueden lanzar desde el propio IDE, y que deben ser ejecutados antes de
cada commit.

Frente a estos tests, los tests de integración necesitan una
configuración adicional (poner en marcha la base de datos de test en
nuestro caso) y se ejecutan menos frecuentemente.

Vamos a actualizar GitHub Actions para que se lancen allí los tests
usando la base de datos PostgreSQL. De esta forma nosotros lanzaremos en
local los tests que usan la BD de memoria y los tests de integración
se lanzarán en GitHub cada vez que vaya a mezclarse un pull request.

### Acción para lanzar los tests con la BD postgres ###

Para lanzar los tests de integración en GitHub debemos modificar el
fichero de configuración del flujo de trabajo para que lance un
contenedor de PostgreSQL y después se ejecuten los tests sobre ese
contenedor.

Para tener más flexibilidad en la configuración de la conexión con
PostgreSQL vamos a modificar el perfil de Spring Boot, añadiendo unas
variables con unos valores por defecto que se pueden modificar
definiendo su valor en variables de entorno con el mismo nombre.

En concreto, definimos las variables `POSTGRES_HOST`, `POSTGRES_PORT`,
`DB_USER` y `DB_PASSWD`.

**Fichero `src/test/resources/application-postgres.properties`**:

```
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
DB_USER=mads
DB_PASSWD=mads
spring.datasource.url=jdbc:postgresql://${POSTGRES_HOST}:${POSTGRES_PORT}/mads_test
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWD}
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.PostgreSQL9Dialect
```

Ya podemos añadir un nuevo fichero de flujo de trabajo. Lo llamamos `integration-tests.yml`

**Fichero `.github/workflows/integration-tests.yml`**:

```yml
name: Integration tests

on: push

jobs:
  container-job:
    runs-on: ubuntu-latest
    container: openjdk:8-jdk-alpine
    services:
      # Etiqueta usada para acceder al contenedor del servicio
      postgres:
        # Imagen Docker Hub
        image: postgres:13
        # Variables para arrancar PostgreSQL
        env:
          POSTGRES_USER: mads
          POSTGRES_PASSWORD: mads
          POSTGRES_DB: mads_test
        # Definimos chequeos para esperar hasta que postgres ya ha comenzado
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v2
      - name: Launch tests with Maven
        run:  ./mvnw test -D spring.profiles.active=postgres
        env:
          POSTGRES_HOST: postgres
```

Vemos que en la última línea se actualiza el parámetro `POSTGRES_HOST`
usado por el perfil `postgres` para que la conexión se realice con el
host `postgres` que es el que nombre que se ha definido en el
servicio.


### Pasos a seguir ###

1. Crea un nuevo issue llamado `Tests de integración en GitHub
Actions`. Crea la rama `integracion-gh-actions`.

    ```
    $ git checkout -b integracion-gh-actions
    $ git push -u origin integracion-gh-actions
    ```
    
    
2. Modifica el fichero del perfil postgres de test tal y como se
   indica anteriormente, para usar variables de configuración que
   puedan ser definidas mediante variables de entorno.

3. Comprueba que siguen funcionando los tests lanzados sobre la base
   de datos usando los valores por defecto de las variables de
   entorno.
   
    ```
    // Nos aseguramos de que la base de datos que está en marcha
    // es la de test
    $ docker container ls
    CONTAINER ID   IMAGE         PORTS                    NAMES
    411d8f2ea46c   postgres:13   0.0.0.0:5432->5432/tcp   postgres-test
    ./mvnw -D spring.profiles.active=postgres test
    ```

4. Comprueba que podemos modificar los parámetros definidos en las
   variables de entorno. Por ejemplo, si se cambia el nombre del host
   de la conexión con la base de datos los tests deben de fallar:
   
    ```
    $ ./mvnw -D spring.profiles.active=postgres -D POSTGRES_HOST=postgres test
    // Aparecerán errores debidos a que no se puede conectar con el
    // host postgres:
    org.postgresql.util.PSQLException: El intento de conexión falló.
    ...
    Caused by: java.net.UnknownHostException: postgres
    ```
   
5. Crea un commit, súbelo a GitHub y crea el Pull Request

    ```
    $ git add .
    $ git commit -m "Añadidas variables al perfil de test postgres"
    $ git push
    ```

6. Añade el fichero del flujo de trabajo de la acción de GitHub,
   tal y como se indica anteriormente. Haz un commit, súbelo a GitHub
   y comprueba que los tests pasan correctamente y se lanzan allí
   usando la base de datos postgres.

    <img src="imagenes/pr-tests-integracion.png" width="600px"/>

    <img src="imagenes/pr-tests-integracion-2.png" width="600px"/>

    <img src="imagenes/pr-tests-integracion-3.png" width="600px"/>

    <img src="imagenes/pr-tests-integracion-4.png" width="750px"/>

7. Una vez comprobado que funcionan los tests de integración en
   GitHub, mezclamos el pull request y lo descargamos a
   local. Comprobamos que también se lanzan los tests en el commit de
   merge en GitHub. 
   
8. Con esto ya tenemos completado un sistema de integración continua y
   GitHub se encargará de ejecutar todos los tests en un modo de
   integración, usando la base de datos PostgreSQL.


## 5. TDD ##

En la segunda parte de la práctica desarrollaremos, usando TDD (_Test
Driven Design_), una nueva _feature_ de la aplicación: la posibilidad
de definir definir equipos a los que puedan pertenecer los usuarios.

Descomponemos la _feature_ en las siguientes historias de usuario.

- 008 Listado de equipos
- 009 Gestionar pertenencia al equipo
- 010 Gestión de equipos (opcional)

**008 Listado de equipos**: Como usuario podré consultar el listado de
los equipos existentes y los participantes en cada uno de ellos para
poder consultar la estructura de la empresa y los proyectos en marcha
y comprobar si estoy en los equipos correctos.

**009 Gestionar pertenencia al equipo**: Como usuario podré crear
nuevos equipos y añadirme y eliminarme de cualquiera de ellos para poder
participar y dejar de participar en ellos.

**010 Gestión de equipos (opcional)**: Como administrador podré
cambiar el nombre y eliminar los equipos para adaptarlos a los
proyectos y estructura de la empresa.

Vamos a hacer de forma guiada la primera historia y dejamos las
siguientes para que las hagas por tu cuenta.

### 008 Listado de equipos ###

La descripción de la historia de usuario es la siguiente:

```text
Listado de equipos

Como usuario podré consultar el listado de
los equipos existentes y los participantes en cada uno de ellos para
poder consultar la estructura de la empresa y los proyectos en marcha
y comprobar si estoy en los equipos correctos.

Detalles

    * En el menú aparcerá una opción `Equipos` que llevará a un
    listado con los nombres de todos los equipos existentes.
    * El listado de equipos estará ordenado por orden alfabético.
    * Pinchando en el enlace del nombre del equipo nos iremos a una
    página con un listado de todos los usuarios que lo componen.
    * Un usuario podrá pertenecer a más de un equipo.
```

Vamos a utilizar la técnica de TDD para construir la funcionalidad
**de dentro a fuera** (desde el repository hasta el
controller). Comenzaremos con tests que construyan la capa de modelo
(clases de entidad y repository) y después pasaremos a tests que
construyan la capa de servicio.

Por último, una vez implementados los métodos de servicios necesarios,
deberás implementar (lo haremos sin tests) las vistas y
controllers. Las vistas y controllers los probaremos de forma manual,
sin tests automáticos.

!!! Important "Importante"

    Los controllers no deben implementar ningún código adicional, sólo
    llamar al método de servicio necesario. De esta forma nos
    aseguramos que todo el código importante para la funcionalidad está
    testeado y ha sido creado mediante TDD.

Recuerda que los pasos seguir la técnica de TDD:

- **Test**: Primero debes escribir el test.
- **Code**: Después debes escribir el código que hace pasar el test (**únicamente el código
necesario, no puedes escribir código de más**)
- **Refactor**: Y, si es necesario, realizar una refactorización del código (los
  tests deben seguir pasando después de la refactorización).

Deberás hacer **un commit por cada fase Test-Code**. Si haces
refactorización deberás hacerlo en otro commit adicional.


### Pasos a seguir ###

- Crea la historia de usuario `008 Listado de equipos` en el tablero Trello.

- Crea dos _issues_ correspondientes a esta historia:
    - Servicio y modelo listado de equipos.
    - Vista y controller listado de equipos.

- Crea una rama para desarrollar el primer _issue_ (llámala
  `servicio-equipos`, por ejemplo) y pásalo en el
  tablero a `In progress`.

Este primer _issue_ lo haremos de forma guiada usando TDD con los
tests que enumeraremos a continuación. El otro _issue_ lo deberás
implementar por ti mismo. 

#### Primer commit - Test y código Entidad `Equipo` ####

El primer test es para crear la entidad `Equipo`. Por ahora sólo
creamos la clase Java, sin las anotaciones JPA. Un equipo

**Fichero `src/test/java/madstodolist/EquipoTest.java`**:
```java
package madstodolist;

// imports

@SpringBootTest
@Sql(scripts = "/clean-db.sql", executionPhase = AFTER_TEST_METHOD)
public class EquipoTest {

    @Test
    public void crearEquipo() {
        Equipo equipo = new Equipo("Proyecto P1");
        assertThat(equipo.getNombre()).isEqualTo("Proyecto P1");
    }
}
```

Escribe el código necesario para que pase el test. **No debes escribir
código de más, sólo el código mínimo para que el test pase**. Haz un
_commit_ que contenga el test y el código y súbelo a la rama remota.

#### Segundo test - Entidad en base de datos ####

Con el segundo test queremos conseguir que funcione JPA con la entidad
`Equipo` y que podamos usar una tabla de equipos en la base de datos,
en la que podamos guardar entidades `equipo`.

Para comprobar que la entidad se ha guardado correctamente,
comprobaremos se ha actualizando su identificador. Lo hacemos
añadiendo test `grabarEquipo`. Actualizamos también el fichero
`clean-db.sql` para que se borre la tabla `equipos` al final de cada
test. 

```java
    @Autowired
    private EquipoRepository equipoRepository;

    @Test
    @Transactional
    public void grabarEquipo() {
        // GIVEN
        Equipo equipo = new Equipo("Proyecto P1");

        // WHEN
        equipoRepository.save(equipo);

        // THEN
        assertThat(equipo.getId()).isNotNull();
    }
```

**Fichero `src/test/resources/clean-db.sql`**:

```sql
DELETE FROM tareas;
DELETE FROM equipos;
DELETE FROM usuarios;
```

Escribe el código necesario para se pase el test y haz un commit.

#### Tercer test - Definición de igualdad entre equipos ####

Ahora que hemos introducido el `id` del equipo escribimos un test
para comprobar que dos equipos son iguales. Debes escribir el código
de los métodos `equals` y `hashCode` (necesario este último para que
funcione correctamente la comprobación de igualdades en las
colecciones).

Hacemos los tests para que el `equals` funcione de la siguiente forma:

> Si alguno de los dos equipos no tiene `id` (es `null`), entonces se
> deben comparar sus nombres. Ahora bien, si los dos equipos tienen
> `id`, entonces se deben comparar esos `id`".

Puedes guiarte por la implementación de `equals` y `hashCode` en
`Usuario`.

```java
    @Test
    public void comprobarIgualdadEquipos() {
        // GIVEN
        // Creamos tres equipos sin id, sólo con el nombre
        Equipo equipo1 = new Equipo("Proyecto P1");
        Equipo equipo2 = new Equipo("Proyecto P2");
        Equipo equipo3 = new Equipo("Proyecto P2");

        // THEN
        // Comprobamos igualdad basada en el atributo nombre
        assertThat(equipo1).isNotEqualTo(equipo2);
        assertThat(equipo2).isEqualTo(equipo3);

        // WHEN
        // Añadimos identificadores y comprobamos igualdad por identificadores
        equipo1.setId(1L);
        equipo2.setId(1L);
        equipo3.setId(2L);

        // THEN
        // Comprobamos igualdad basada en el atributo nombre
        assertThat(equipo1).isEqualTo(equipo2);
        assertThat(equipo2).isNotEqualTo(equipo3);
    }
```

Escribe el código necesario para se pase el test y haz un commit.

#### Cuarto test - Añadir y buscar equipo en base de datos ####

Escribimos ahora un test que pruebe los métodos de añadir y buscar un
equipo de la clase repository.

Test:

```java
    @Test
    public void comprobarRecuperarEquipo() {
        // GIVEN
        // Un equipo en la base de datos
        Equipo equipo = new Equipo("Proyecto Cobalto");
        equipoRepository.save(equipo);
        Long equipoId = equipo.getId();

        // WHEN

        Equipo equipoBD = equipoRepository.findById(equipoId).orElse(null);

        // THEN
        assertThat(equipo).isNotNull();
        assertThat(equipo.getId()).isEqualTo(equipoId);
        assertThat(equipo.getNombre()).isEqualTo("Proyecto Cobalto");
    }
```

Comprueba el test y si es necesario escribe el código estríctamente
necesario para que pase.

Haz un commit en la rama y súbelo a GitHub.

#### Quinto test - Relación muchos-a-muchos entre equipos y usuarios ####

Vamos ahora a diseñar un test que introduzca la relación entre equipos
y usuarios. Debe ser una relación muchos-a-muchos: un equipo contiene
muchos usuarios y un usuario puede pertenecer a 0, 1 o muchos equipos.

En el test hacemos varias cosas: creamos un equipo y un usuario,
añadimos el usuario al equipo y comprobamos que las relaciones se han
actualizado en la base de datos.

```java
    @Autowired
    private UsuarioRepository usuarioRepository;
    
    @Test
    @Transactional
    public void comprobarRelacionBaseDatos() {
        // GIVEN
        // Un equipo y un usuario en la BD
        Equipo equipo = new Equipo("Proyecto Cobalto");
        equipoRepository.save(equipo);
        Long equipoId = equipo.getId();

        Usuario usuario = new Usuario("user@ua");
        usuarioRepository.save(usuario);
        Long usuarioId = usuario.getId();

        // WHEN
        // Añadimos el usuario al equipo

        equipo.addUsuario(usuario);

        // THEN
        // La relación entre usuario y equipo queda actualizada en BD

        Equipo equipoBD = equipoRepository.findById(equipoId).orElse(null);
        Usuario usuarioBD = usuarioRepository.findById(usuarioId).orElse(null);

        assertThat(equipo.getUsuarios()).hasSize(1);
        assertThat(equipo.getUsuarios()).contains(usuario);
        assertThat(usuario.getEquipos()).hasSize(1);
        assertThat(usuario.getEquipos()).contains(equipo);
    }
```


Para que este test funcione hay que crear la relación muchos-a-muchos
entre equipos y usuarios.  Es necesario definir la anotación
`@ManyToMany` para indicar a JPA cómo construir las tablas en la base
de datos. Vamos a crear esta relación como `LAZY`, porque si fuera
`EAGER` la recuperación de equipos de la base de datos sería muy
costosa, traería a memoria todos sus usuarios (con sus tareas incluidas).

En `Equipo.java` definimos la tabla en la que se va a guardar la
relación, e indicamos el papel de cada una de sus dos columnas.

También creamos el getter para obtener los usuarios. 

**Fichero `src/main/java/madstodolist/model/Equipo.java`**:
```diff
+    private String nombre;
+    // Declaramos el tipo de recuperación como LAZY.
+    // No haría falta porque es el tipo por defecto en una
+    // relación a muchos.
+    // Al recuperar un equipo NO SE RECUPERA AUTOMÁTICAMENTE
+    // la lista de usuarios. Sólo se recupera cuando se accede al
+    // atributo 'usuarios'; entonces se genera una query en la
+    // BD que devuelve todos los usuarios del equipo y rellena el
+    // atributo.
+     
+    @ManyToMany(fetch = FetchType.LAZY)
+    @JoinTable(name = "equipo_usuario",
+            joinColumns = { @JoinColumn(name = "fk_equipo") },
+            inverseJoinColumns = {@JoinColumn(name = "fk_usuario")})
+    Set<Usuario> usuarios = new HashSet<>();

...

    public void setId(Long id) {
        this.id = id;
    }

+    public Set<Usuario> getUsuarios() {
+        return usuarios;
+    }

...
+    public void addUsuario(Usuario usuario) {
+        this.getUsuarios().add(usuario);
+        usuario.getEquipos().add(this);
`    }
```

En el fichero `Usuario.java` definimos la parte inversa de la
relación. El `mappedBy` indica que la especificación de la tabla join
está en el otro lado de la relación. Esta relación la definimos como
`EAGER` porque el otro lado de la relación es `LAZY`. Al recuperar un
usuario solo se van a traer a memoria la información de sus equipos.

**Fichero `src/main/java/madstodolist/model/Usuario.java`**:
```diff
    @OneToMany(mappedBy = "usuario", fetch = FetchType.EAGER)
    Set<Tarea> tareas = new HashSet<>();

+    @ManyToMany(mappedBy = "usuarios")
+    Set<Equipo> equipos = new HashSet<>();

...

+    public Set<Equipo> getEquipos() {
+        return equipos;
+    }
```

Y por último añadimos en `Equipo.java` el método que actualiza la
relación:

```java
    public void addUsuario(Usuario usuario) {
        this.getUsuarios().add(usuario);
        usuario.getEquipos().add(this);
    }
```

Y actualizamos el fichero de limpieza de datos al final de cada test:

**Fichero `src/test/resources/clean-db.sql`**:

```sql
DELETE FROM equipo_usuario;
DELETE FROM tareas;
DELETE FROM equipos;
DELETE FROM usuarios;
```

Comprueba el test, haz un commit en la rama y súbelo a GitHub.

#### Sexto test - listado de equipos ####

Vamos ahora a definir un test para obtener una lista de equipos en el
_repository_. Queremos que el tipo devuelto por el _repository_
sea _List_.

```java
    @Test
    @Transactional
    public void comprobarFindAll() {
        // GIVEN
        // Dos equipos en la base de datos
        equipoRepository.save(new Equipo("Proyecto Cobalto"));
        equipoRepository.save(new Equipo("Proyecto Níquel"));

        // WHEN
        List<Equipo> equipos = equipoRepository.findAll();

        // THEN
        assertThat(equipos).hasSize(2);
    }
```

La solución consiste en añadir el método `findAll` en la interfaz
p`EquipoRepository`, definiendo el tipo devuelto como _List_. Spring
Boot se encarga de construir automáticamente la implementación de este
método.


**Fichero `EquipoRepository.java`**:

```diff
+ import java.util.List;

public interface EquipoRepository extends CrudRepository<Equipo, Long> {
+     public List<Equipo> findAll();
}

```

#### Séptimo test - Método de servicio para el listado de equipos ####

¡Y por fin llegamos a la capa de servicio!

Creamos el fichero `EquipoServiceTest.java` con el código para añadir
dos equipos de prueba y el test para recuperarlos ordenados por nombre:

**Fichero `src/test/java/madstodolist/EquipoServiceTest.java`**:

```java
package madstodolist;

// imports

@SpringBootTest
@Sql(scripts = "/clean-db.sql", executionPhase = AFTER_TEST_METHOD)
public class EquipoServiceTest {

    @Autowired
    EquipoService equipoService;

    // Añade dos equipos a la base de datos 
    public void addEquiposBD() {
        Equipo equipo1 = equipoService.crearEquipo("Proyecto Cobalto");
        Equipo equipo2 = equipoService.crearEquipo("Proyecto Níquel");
    }
    
    @Test
    public void obtenerListadoEquipos() {
        // GIVEN
        // Dos equipos en la base de datos
        addEquiposBD();

        // WHEN
        List<Equipo> equipos = equipoService.findAllOrderedByName();

        // THEN
        assertThat(equipos).hasSize(2);
        assertThat(equipos.get(0).getNombre()).isEqualTo("Proyecto Cobalto");
        assertThat(equipos.get(1).getNombre()).isEqualTo("Proyecto Níquel");
    }
}
```

Escribe el código estríctamente necesario para que pase. Haz un commit
en la rama y súbelo a GitHub.

<!-- TODO: Ordenar los tests que siguen.. 

- Primero debería añadirse un test para actualizar y recuperar la relación
- Y después comprobar todo el tema de EAGER y LAZY
--->

#### Octavo test - Método de servicio para recuperar un equipo ####

Vamos a centrar este test en la forma de traer a memoria los objetos
que participan en la relación `USUARIO-EQUIPO`. 

En JPA hay dos formas de definir una relación a-muchos:

- `EAGER`: Si una relación a-muchos es `EAGER`, cuando la clase
  repository devuelve un objeto (ya sea al recuperarlo
  individualmente, o en una consulta en la que se recupera una
  colección), se obtienen también de la base de datos todos los
  objetos con los que está relacionado. Por ejemplo, en la práctica
  tenemos definida de esta forma la relación entre usuarios y tareas.

- `LAZY`: Si una relación a-muchos es `LAZY`, cuando la clase
  repository devuelve un objeto, no recupera de la base de datos los
  objetos relacionados. Sólo lo hace cuando se accede a la colección
  que contiene la relación. Entonces es cuando se realiza la consulta
  a la base de datos y se traen estos objetos a memoria. Si estos
  objetos tienen otras relaciones se traerán a memoria o no
  dependiendo de si son `EAGER` o `LAZY`.
  
  Para que funcione la recuperación perezosa debe estar abierta la
  conexión con la base de datos en el momento en que se accede a la
  colección. Para ello es muy importante la etiqueta
  `@Transactional`. Cuando ponemos esta etiqueta en los métodos de las
  clases de servicio se garantiza que todo el método se realiza en una
  única transacción. Por ello, al finalizar el método se cerrará la
  conexión con la base de datos y el objeto que se devolverá al
  _controller_ **estará desconectado de la base de datos**, por lo que
  la recuperación perezosa no funcionará en el _controller_.

En el caso de la relación USUARIO-EQUIPO hemos definido el siguiente
diseño:

- La relación entre un usuario y sus equipos es `EAGER`. Cuando
  recuperamos un usuario, recuperaremos también la información de
  todos los equipos en los que participa.
  
- La relación entre un equipo y sus usuarios es `LAZY`. Esto es muy
  importante. Si no lo hiciéramos así ¡podríamos fácilmente traernos a
  memoria toda la base de datos!. Un equipo recuperaría todos sus
  usuarios, que también pueden estar en otros equipos, que a su vez
  también se traerían a memoria. 

Actualizamos el fichero de test para que el método `addEquiposBD`
añada además un usuario al primer equipo y devuelva los
identificadores de los dos equipos y del usuario:

**Fichero `src/test/java/madstodolist/EquipoServiceTest.java`**:

```java
@SpringBootTest
@Sql(scripts = "/clean-db.sql", executionPhase = AFTER_TEST_METHOD)
public class EquipoServiceTest {

    @Autowired
    EquipoService equipoService;

    @Autowired
    UsuarioService usuarioService;

    class TresIds {
        Long equipo1Id;
        Long equipo2Id;
        Long usuarioId;
        TresIds(Long equipo1Id, Long equipo2Id, Long usuarioId) {
            this.equipo1Id = equipo1Id;
            this.equipo2Id = equipo2Id;
            this.usuarioId = usuarioId;
        }
    }

    // Añade dos equipos a la base de datos y un usuario en el primer equipo.
    // Devuelve el identificador de los equipos y de los usuarios.
    public TresIds addEquiposBD() {
        Equipo equipo1 = equipoService.crearEquipo("Proyecto Cobalto");
        Equipo equipo2 = equipoService.crearEquipo("Proyecto Níquel");
        Usuario usuario = new Usuario("user@ua");
        usuario.setPassword("123");
        usuario = usuarioService.registrar(usuario);
        equipoService.addUsuarioEquipo(usuario.getId(), equipo1.getId());
        return new TresIds(equipo1.getId(), equipo2.getId(), usuario.getId());
    }

...
}
```

Y definimos un test que sirve para crear el método de
servicio que recupera un equipo y que se asegura de que la relación
entre equipos y usuarios es `LAZY`.

**Fichero `src/test/java/madstodolist/EquipoServiceTest.java`**:

```java
    @Test
    public void obtenerEquipo() {
        // GIVEN
        // Un equipo en la base de datos
        Long equipoId = addEquiposBD().equipo1Id;

        // WHEN
        Equipo equipoBD = equipoService.findById(equipoId);

        // THEN
        assertThat(equipoBD.getNombre()).isEqualTo("Proyecto Cobalto");
        // Comprobamos que la relación con Usuarios es lazy: al
        // intentar acceder a la colección de usuarios se debe lanzar una
        // excepción
        assertThatThrownBy(() -> {
            equipoBD.getUsuarios().size();
        }).isInstanceOf(LazyInitializationException.class);
    }
```

Comprueba si hay que modificar el código, haz un commit y súbelo a
GitHub.

#### Noveno test - comprobación de recuperación _eager_ de equipos ####

Hacemos ahora un test para que un usuario recupere de forma _eager_
sus equipos:

**Fichero `src/test/java/madstodolist/EquipoServiceTest.java`**:

```java
    @Test
    public void comprobarRelacionUsuarioEquipos() {
        // GIVEN
        // Un equipo con un usuario en la BD
        Long usuarioId = addEquiposBD().usuarioId;

        // WHEN
        // Recuperamos el usuario de la base de datos
        Usuario usuarioBD = usuarioService.findById(usuarioId);

        // THEN
        // Se recuperan también los equipos del usuario,
        // porque la relación entre usuarios y equipos es EAGER
        assertThat(usuarioBD.getEquipos()).hasSize(1);
    }
```

Modifica el código para que el test pase, haz un commit y súbelo a GitHub.

#### Décimo test - Método de servicio para obtener los usuarios de un equipo ####

El último test que sirve para definir el método de servicio
`usuariosEquipo(Long idEquipo)` que devuelve la lista de usuarios de
un equipo.

Después de comprobar que la lista que se devuelve es correcta,
volvemos a comprobar que la relación entre usuarios y equipos es
`EAGER`, esto es, que desde un usuario se puede obtener la lista de
equipos a los que pertenece.

```java
    @Test
    public void obtenerUsuariosEquipo() {
        // GIVEN
        // Un equipo con un usuario en la BD
        Long equipoId = addEquiposBD().equipo1Id;

        // WHEN
        // Recuperamos los usuarios del equipo
        List<Usuario> usuarios = equipoService.usuariosEquipo(equipoId);

        // THEN
        // Se actualizan correctamente las relaciones
        assertThat(usuarios).hasSize(1);
        assertThat(usuarios.get(0).getEmail()).isEqualTo("user@ua");
        // Comprobamos que la relación entre usuarios y equipos es eager
        // Primero comprobamos que la colección de equipos tiene 1 elemento
        assertThat(usuarios.get(0).getEquipos()).hasSize(1);
        // Y después que el elemento es el equipo Proyecto Cobalto
        assertThat(usuarios.get(0).getEquipos().stream().findFirst().get().getNombre()).isEqualTo("Proyecto Cobalto");
    }
```

Escribe el código necesario para que pase. Haz un commit en la rama y
súbelo a GitHub.

#### Cierre del _issue_ ####

Cuando hayas terminado todos los ciclos de TDD anteriores habrás
terminado el _issue_ y testeado e implementado los métodos necesarios
para la clase de servicio que gestiona el listado de equipos y
usuarios de esos equipos.

- Crea un pull request que cierre el _issue_, comprueba que GitHub
Actions pasa correctamente los tests e intégralo en `main` en
GitHub. Baja los cambios al repositorio local.


#### Vista y controller listado de equipos ####

- Abre un nuevo _issue_ para implementar el controller y las vistas
que permitan listar los equipos y consultar sus miembros (por ejemplo,
pulsando en un enlace en el nombre del equipo o con un botón en el listado).

- Realiza el desarrollo del _issue_ usando varios commits en los que
  añadas las funcionalidades poco a poco. No hace falta que hagas TDD,
  pero añade al menos un test por cada método del controller.

### Resto de historias de usuario ###

Debes implementar la historia de usuario de la misma forma que hemos
implementado la anterior.

- **009 Gestionar pertenencia al equipo**: Como usuario podré crear
nuevos equipos y añadirme y eliminarme de cualquiera de ellos para poder
participar y dejar de participar en ellos.


!!! Important "Importante detalle de implementación"
    En una relación muchos-a-muchos como la que
    existe entre `Usuario` y `Equipo` cuando se añade un usuario a un
    equipo hay que actualizar ambos lados de la relación, porque
    JPA/Hibernate no lo hace automáticamente. Hay que añadir el
    usuario a la colección de usuarios del equipo y también añadir el
    equipo a la colección de equipos del usuario.
  
    Lo mismo habría que hacer cuando se elimina un usuario de un equipo.
  
- **010 Gestión de equipos (opcional)**: Como administrador cambiar el
nombre y eliminar los equipos para adaptarlos a los proyectos y
estructura de la empresa.

### Pasos a seguir ###

- Implementa cada historia de usuario usando el mismo proceso que
  hemos utilizado para la historia 008. Deberás pensar qué servicios
  son necesarios para la historia y cómo implementarlos haciendo TDD.

  Para cada historia haz dos _issues_: uno con TDD para implementar la
  capa de servicio y repository y otro sin TDD para la capa de
  controller y vista.
  
  Cuando estés haciendo TDD completa el código para pasar los tests,
  uno a uno, **haciendo un commit después de cada fase test-code** y
  otro commit en la fase **refactor** (en el caso en que tengas que
  hacer refactorización). 
  
  Los incrementos de código introducidos por los tests deben ser
  pequeños. Debe haber **entre 15 y 25 líneas de código** añadidas en
  las fases de codificación (sin contar el código de los tests). No
  tomes este número de forma demasiado estricta; si en algún ciclo hay
  que añadir 35 líneas no pasa nada. Tampoco si haces menos
  de 15. Pero estaría mal tener que añadir 70 líneas para resolver un
  test.
  
- Cuando termines las historias de usuario (ve moviéndolas también en
  el tablero de Trello) haz el release 1.2.0 con la entrega final de
  la práctica.

  
## 6. Documentación, entrega y evaluación ##

Deberás añadir una página de documentación `/doc/practica3.md` en la
que, al igual que en la práctica anterior, debes realizar una **breve
documentación técnica** de entre 500 y 800 palabras sobre lo
implementado en las historias de usuario 009 y 010.

En la documentación debes incluir también una **captura de pantalla**
en la que se muestren las tablas de la base de datos de desarrollo
PostgreSQL en la versión final de la aplicación. Puedes mostrar, por
ejempo, una pantalla con el panel `Database` de _IntelliJ_ o la
herramienta que hayas utilizado. Basta solo con una captura de la base
de datos de desarrollo, no hace falta mostrar la base de datos de
test.

Por ejemplo, puedes incluir en la documentación lo siguiente. Los
puntos 2 en adelante son sobre las **historias de usuario 009 y 010**.

1. Pantalla de la base de datos PostgreSQL.
2. Rutas (endpoints) definidas para las acciones y, para cada endpoint o grupo de endpoints,
   explicación sobre:
    1. Clases y métodos
    2. Plantillas thymeleaf
    3. Tests
3. Explicación de algunos fragmentos de código fuente que consideres
   interesante en las nuevas funcionalidades implementadas.

Intenta que el documento tenga un formato limpio y se pueda leer
fácilmente. Para eso utiliza los bloques de código de Markdown. Puedes
mirar como ejemplo el código Markdown de estas prácticas.

Por ejemplo, el código Markdown de la [introducción a Spring
Boot](https://github.com/domingogallardo/practicas-mads/blob/main/docs/01-intro-spring-boot/intro-spring-boot.md)
se puede ver pulsando el botón `Raw`. Verás el [texto Markdown](https://raw.githubusercontent.com/domingogallardo/practicas-mads/main/docs/01-intro-spring-boot/intro-spring-boot.md).

- La práctica tiene una duración de 3 semanas y la fecha límite de
  entrega es el martes 10 de noviembre.
- La parte obligatoria puntúa sobre 8 y la opcional sobre 2 puntos.
- La calificación de la práctica tiene un peso de un 25% en la nota
  final de prácticas. 
- Para realizar la entrega se debe subir a Moodle un ZIP que contenga
  todo el proyecto, incluyendo el directorio `.git` que contiene la
  historia Git. Para ello comprime tu directorio local del proyecto
  **después de haber hecho un `./mvnw clean`** para eliminar el
  directorio `target` que contiene los binarios compilados. Debes
  dejar también en Moodle la URL del repositorio en GitHub.

Para la evaluación se tendrá en cuenta:

- Desarrollo continuo (los _commits_ deben realizarse a lo largo de
  las 3 semanas y no dejar todo para la última semana).
- Correcto desarrollo de la metodología.
- Diseño e implementación del código y de los tests de las
  características desarrolladas.
- Documentación.
