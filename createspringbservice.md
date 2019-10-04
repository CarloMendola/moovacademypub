# Creare un servizio Spring Boot
### + 300 pt
## Prepariamo l'ambiente di lavoro
### ~ 20 min
Ora installeremo e verificheremo di avere tutto ciò che ci servirà per realizzare il nostro servizio
#### Di cosa avremo bisogno
* Il nostro IDE preferito
* OpenJDK8 installato
* Maven 3.2, o superiore, installato e configurato
#### Installazione OpenJDK 8
* [Per Windows](https://docs.aws.amazon.com/corretto/latest/corretto-8-ug/windows-7-install.html)
* [Per Mac](https://docs.aws.amazon.com/corretto/latest/corretto-8-ug/macos-install.html)
* [Per Linux](https://docs.aws.amazon.com/corretto/latest/corretto-8-ug/linux-info.html)
#### Installazione Maven
* [Download](https://maven.apache.org/download.cgi)
* [Installazione](https://maven.apache.org/install.html)

:fire: Cos'è [Apache Maven](https://maven.apache.org/what-is-maven.html)
## Creiamo il servizio
### ~ 30 min
Creiamo ora un semplice servizio Spring Boot che eseguiremo localmente e testeremo
#### Che cosa realizzeremo
Realizzeremo un servizio che accetterà richieste HTTP __GET__

:fire: _[Metodi di richiesta HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods)_
```
http://localhost:8080/greeting
```
e risponderà con la rappresentazione JSON di un saluto

:fire: _**JavaScript Object Notation** ovverosia [**JSON**](https://en.wikipedia.org/wiki/JSON)_
```json
{"id":1,"content":"Hello, World!"}
```
Sarà possibile personalizzare il messaggio di saluto con un parametro `name` (facoltativo) nella query string:
```
http://localhost:8080/greeting?name=User
```
Nel caso in cui sia presente il parametro `name`, il valore di quest'ultimo sovrascrive il valore di default `World`. Questo comportamento si riflette nel messaggio di risposta:
```
http://localhost:8080/greeting?name=User
```
```json
{"id":1,"content":"Hello, User!"}
```
#### Creiamo la struttura di directory necessaria
In una directory di progetto di tua scelta, crea la seguente struttura di sottodirectory

![Struttura di directory](./images/logo.png)

Creiamo poi il file `pom.xml` 

:fire: _Che cos'è un file [pom](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)?_

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-rest-service</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.jayway.jsonpath</groupId>
            <artifactId>json-path</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```
Soffermiamoci un secondo sullo `spring-boot-maven-plugin`. Questo plugin ci mette a disposizione alcune feature molto interessanti:
* raccoglie tutti i jars nel classpath e crea un unico "über-jar" eseguibile, il che rende più conveniente eseguire e trasportare il servizio
* cerca il metodo `public static void main()` all'interno di una classe per contrassegnarla come eseguibile
* fornisce un sistema di risoluzione di dipendenze integrato che imposta il numero di versione in modo tale che corrisponda alle dipendenze di Spring Boot. Sarà sempre possibile sovrascrivere qualsiasi versione si desideri, ma per impostazione predefinita sarà il set di versioni scelto da Spring Boot
#### Creiamo la classe che rappresenterà la risorsa
Il servizio gestirà le richieste __GET__ per il path `/greeting`, inoltre, facoltativamente, potrà essere utilizzato un parametro `name` nella query-string. La richiesta __GET__ dovrebbe restituire una risposta di __200 OK__ con __JSON__ nel corpo che rappresenta un saluto.

:fire: _[HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)_

Qualcosa di simile a
```json
{"id":1,"content":"Hello, World!"}
```
Il campo `id` è un identificatore univoco per il saluto e `content` è la rappresentazione testuale del saluto.

Per modellare l'oggetto saluto dovremo creare una classe che lo rappresenti. In questo caso sarà sufficiente modellare la nostra classe come un JavaBean.

:fire: *Cos'è un [JavaBean](https://it.wikipedia.org/wiki/JavaBean)*

`src/main/java/hello/Greeting.java`

```java
package hello;

public class Greeting {

    private final long id;
    private final String content;

    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent() {
        return content;
    }
}
```
#### Creiamo il controller
Nell'approccio di Spring alla creazione di servizi web RESTful, le richieste HTTP sono gestite da un controller. These components are easily identified by the `@RestController` annotation, and the `GreetingController` below handles **GET** requests for `/greeting` by returning a new instance of the `Greeting` class:

`src/main/java/hello/GreetingController.java`

```java
package hello;

import java.util.concurrent.atomic.AtomicLong;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @RequestMapping("/greeting", method = GET)
    public Greeting greeting(@RequestParam(value="name", defaultValue="World") String name) {
        return new Greeting(counter.incrementAndGet(),
                            String.format(template, name));
    }
}
```

:fire: _Vuoi saperne di più sulla classe [AtomicLong](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/atomic/AtomicLong.html)?_

Questo controller è conciso e semplice, ma dietro le quinte c'è molto di più di quello che potrebbe sembrare. Analizziamolo passo dopo passo.
* The `@RequestMapping` annotation ensures that HTTP requests to `/greeting` are mapped to the `greeting()` method.
* `@RequestParam` associa il valore del parametro `name` della query-string al parametro `name` del metodo `greeting()`. Se il parametro `name` è assente, viene utilizzato il valore predefinito di `Mondo`.
* Una differenza fondamentale tra un controller **MVC** tradizionale e un controller **REST** è il modo in cui viene creato il corpo della risposta HTTP. Anziché basarsi su una tecnologia di visualizzazione per eseguire il rendering lato server dei dati in **HTML**, questo controller semplicemente popola e restituisce un oggetto **JSON**.
* Questo codice utilizza l'annotazione `@RestController`, che contrassegna la classe come controller in cui ogni metodo restituisce un oggetto di dominio anziché una vista. È una scorciatoia per l'utilizzo delle annotazioni `@Controller` e `@ResponseBody` messe insieme.
* L'oggetto `Greeting` deve essere convertito in **JSON**. Grazie al supporto del convertitore di messaggi HTTP di Spring, non è necessario eseguire questa conversione manualmente. Poiché Jackson 2 si trova nel classpath, il converter  `MappingJackson2HttpMessageConverter` di Spring viene automaticamente scelto per convertire l'istanza di saluto in **JSON**.

:fire: _Parliamo di [REST](https://medium.com/extend/what-is-rest-a-simple-explanation-for-beginners-part-1-introduction-b4a072f8740f)_
#### Rendiamo la nostra applicazione eseguibile
Sebbene sia possibile pacchettizzare questo servizio come un file WAR tradizionale per per il deploy su di un application server, l'approccio che vedremo ora sarà quello di creare un'applicazione autonoma e indipendente, distribuita tramite in un unico file JAR eseguibile, quest'ultimo pilotato da un buon vecchio metodo `main()`. Vedremo come tilizzare il supporto di Spring per incorporare le librerie di Tomcat come runtime HTTP, anziché distribuirlo a un'istanza esterna.

`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
`@SpringBootApplication` è un'annotazione che, in maniera sintetica, riassume i comportamenti delle annotazioni:
* `@Configuration`: contrassegna la classe come fonte di definizioni bean per il contesto dell'applicazione.
* `@EnableAutoConfiguration`: indica a Spring Boot di iniziare ad aggiungere bean in base alle impostazioni del classpath, altri bean e varie impostazioni delle proprietà. Ad esempio, se spring-webmvc si trova nel classpath, questa annotazione contrassegna l'applicazione come un'applicazione Web e attiva comportamenti chiave, come l'impostazione di un `DispatcherServlet`.
* `@ComponentScan`: indica a Spring di cercare altri componenti, configurazioni e servizi nel pacchetto `Hello`, consentendogli di trovare i controller

Il metodo `main()` invoca il metodo `SpringApplication.run()` di Spring Boot per avviare un'applicazione. Come si può notare non è stato necessario utilizzare una sola riga di **XML** o di configurazione.
#### Creiamo il nostro JAR eseguibile
È possibile eseguire l'applicazione dalla riga di comando con **Gradle** o **Maven**. È inoltre possibile creare un singolo file **JAR** eseguibile che contenga tutte le dipendenze, le classi e le risorse necessarie ad eseguirlo. La creazione di un unico **JAR** eseguibile semplifica la gestione del servizio durante il suo intero ciclo di vita.

Sarà ora possibile creare il file **JAR** tramite il comando `mvn clean package` e successivamente eseguire il file **JAR**, come segue:
```
java -jar target/gs-rest-service-0.1.0.jar
```
Entro pochi secondi verrà visualizzato l'output del log e il servizio sarà essere attivo e funzionante.

:fire: _Capire il formato [JAR](https://docs.oracle.com/javaee/6/tutorial/doc/bnaby.html) e le sue alternative_
#### Testiamo il servizio
Ora che il servizio è attivo digitiamo l'indirizzo `http://localhost:8080/greeting` dove vedremo
```json
{"id":1,"content":"Hello, World!"}
```
Aggiungendo e valorizzando il parametro `name` alla query-string `http://localhost:8080/greeting?name=User` potremmo vedere come il contenuto della risposta cambierà:
```json
{"id":2,"content":"Hello, User!"}
```
Questa differenza nel risultato dimostra come l'aggiunta del `@RequestParam` in `GreetingController` funzioni come previsto. Al parametro `name` è stato assegnato un valore predefinito `World`, ma può sempre essere esplicitamente ignorato tramite la query-string.

Notare anche come l'attributo `id` sia cambiato da `1` a `2`. Ciò dimostra che si sta lavorando sulla stessa istanza di `GreetingController` anche attraverso più richieste richieste e che il campo contatore viene incrementato ad ogni chiamata, come previsto.
