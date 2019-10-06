# Kafka: Creare un cluster Kafka con docker
### + 500 pt

## Setup Portainer
### ~ 10 min
Prima di entrare a piene mani sulla creazione del cluster kafka, andiamo ad introdurre un'interfaccia di gestione grafica (frontend) per docker che ci facilita nelle interazioni. 
Docker e' una tecnologia che permette di rendere le applicazioni facilmente distribuibili e funzionanti in brevissimo tempo, senza gli oneri di installazione e configurazione che hanno sempre richiesto ore di lavoro di tipo sistemistico.
Portainer e' un applicazione che permette di gestire le operazioni verso un docker engine, e si puo' avviare in pochi comandi attraverso docker. 
Tutto quello che serve per eseguire un applicazione in docker consiste in:
- un immagine dell'applicazione (reperibile su dockerhub)
- eseguire un comando docker per impostare qualche variabile di ambiente o volume per configurare l'applicazione
Una volta eseguiti questi passaggi per deployare Portainer, si potra' operare in futuro attraverso un interfaccia grafica web.
I passaggi per deployare Portainer possono cambiare in funzione del sistema operativo su cui e' installato Docker. Per i dettagli di tutti gli ambienti supportati, fare riferimento a: 
[Deployment Portainer](https://portainer.readthedocs.io/en/stable/deployment.html):fire: 
Di seguito un esempio per deployare portainer su un ambiente Linux.
```
$ docker volume create portainer_data
$ docker pull portainer/portainer:latest
$ docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```
Da questo momendo e' possibile accedere a portainer puntando da browser l'indirizzo ip su cui e' attivo docker alla porta 9000 (o quella configurata al momento del docker run)

------------------------------
## Introduzione a docker-compose
### ~ 15 min
Come abbiamo appena visto, tramite pochi semplici comandi docker e' possibile deployare un'applicazione, ma se avessimo a che fare con tecnologie complesse e articolate, potremmo avere necessita' di deployare gruppi di applicazioni che interagiscono tra loro.
Eseguire i singoli passaggi tramite ripetizione di comandi docker e' sicuramente una possibilita' che permette di prendere possesso della tecnologia, ma per questioni di velocita', una volta preso possesso della tecnologia, si puo' ricorrere all'utilizzo di docker-compose.
'docker-compose' e' un comando che permette di deployare rapidamente un gruppo di applicazioni, che sono state prima descritte accuratamente attraverso un file YAML. 
Per approfondimenti su compose file si faccia riferimento a: 
[Compose file version 2 reference](https://docs.docker.com/compose/compose-file/compose-file-v2/) :fire:
```
Nota:
Il link riporta informazioni sulla versione 2 di file compose perche' al momento e' l'unica supportata tramite portainer. 
Per chi non dovesse operare tramite portainer e' possibile utilizzare l'ultima versione. 
```
La sezione di portainer che consente di operare attraverso docker-compose si trova alla sezione "Stacks" dal menu di sinistra (necessaria una versione recente di portainer come la 1.21.0)
## Descrizione servizi per Kafka Cluster su file YAML
### ~ 10 min
Entrando tramite portainer, dalla sezione "Stacks", e' possibile accedere alla sezione "Create stack" dove e' disponibile un editor di test web dove dichiarare il nome e i servizi da deployare su docker.
In particolare, i servizi che andremo a dichiarare nella prima versione del file yaml sono:
- zookeeper (servizio necessario a kafka per distribuzione delle configurazioni dei broker)
- broker-kafka (uno o piu' broker che costituiscono il cluster kafka)

Il file yaml sara' quindi composto come sotto:
```
version: "2"

services:
  zookeeper:
    image: confluentinc/cp-zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_SERVER_ID: "1"
      ZOOKEEPER_CLIENT_PORT: "2181"
      ZOOKEEPER_TICK_TIME: "2000"
    restart: unless-stopped
  kafka-1:
    image: confluentinc/cp-kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    volumes:
      - /kafkadata/1:/var/lib/kafka/data
    restart: unless-stopped
    environment:
      KAFKA_ZOOKEEPER_CONNECT: "10.121.204.70:2181"
      KAFKA_BROKER_ID: "1"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://10.121.204.70:9092,PLAINTEXT://10.121.204.70:9093,PLAINTEXT://10.121.204.70:9094"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: 50000012
      KAFKA_MESSAGE_MAX_BYTES: 50000012
  kafka-2:
    image: confluentinc/cp-kafka
    depends_on:
      - zookeeper
    ports:
      - "9093:9093"
    volumes:
      - /kafkadata/2:/var/lib/kafka/data
    restart: unless-stopped
    environment:
      KAFKA_ZOOKEEPER_CONNECT: "10.121.204.70:2181"
      KAFKA_BROKER_ID: "2"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://10.121.204.70:9092,PLAINTEXT://10.121.204.70:9093,PLAINTEXT://10.121.204.70:9094"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: 50000012
      KAFKA_MESSAGE_MAX_BYTES: 50000012
  kafka-3:
    image: confluentinc/cp-kafka
    depends_on:
      - zookeeper
    ports:
      - "9094:9094"
    volumes:
      - /kafkadata/3:/var/lib/kafka/data
    restart: unless-stopped
    environment:
      KAFKA_ZOOKEEPER_CONNECT: "10.121.204.70:2181"
      KAFKA_BROKER_ID: "3"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://10.121.204.70:9092,PLAINTEXT://10.121.204.70:9093,PLAINTEXT://10.121.204.70:9094"      
      KAFKA_SOCKET_REQUEST_MAX_BYTES: 50000012
      KAFKA_MESSAGE_MAX_BYTES: 50000012
```

Definito lo YAML, il nome dello stack (nel nostro caso "kafkacluster"), procendendo su "Deploy Stack" e il cluster sara' creato.

## Modifica dello stack "clusterkafka" per Kafdrop
### ~ 5 min
Per essere agevolati nella verifica di quanto prodotto fino ad ora, andiamo a modificare lo stack "clusterkafka" aggiungendo un nuovo servizio.
Kafdrop e' un servizio che mette a disposizione un interfaccia grafice web per visualizzare metadati relativi al cluster kafka e il contenuto dei topic Kafka.
Il nuovo servizio puo' essere deployato nello stack appena creato, e' infatti possibile modificare il file yaml tramite "Editor" dalla sezione "Stacks" selezionando "kafkacluster".
Aggiungere la dichiarazione del nuovo servizio assicurandosi che lo yaml sia come quanto riportato sotto e selezionare "Update the stack".
```
version: "2"

services:
  zookeeper:
    image: confluentinc/cp-zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_SERVER_ID: "1"
      ZOOKEEPER_CLIENT_PORT: "2181"
      ZOOKEEPER_TICK_TIME: "2000"
    restart: unless-stopped
  kafka-1:
    image: confluentinc/cp-kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    volumes:
      - /kafkadata/1:/var/lib/kafka/data
    restart: unless-stopped
    environment:
      KAFKA_ZOOKEEPER_CONNECT: "10.121.204.70:2181"
      KAFKA_BROKER_ID: "1"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://10.121.204.70:9092"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: 50000012
      KAFKA_MESSAGE_MAX_BYTES: 50000012
  kafka-2:
    image: confluentinc/cp-kafka
    depends_on:
      - zookeeper
    ports:
      - "9093:9093"
    volumes:
      - /kafkadata/2:/var/lib/kafka/data
    restart: unless-stopped
    environment:
      KAFKA_ZOOKEEPER_CONNECT: "10.121.204.70:2181"
      KAFKA_BROKER_ID: "2"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://10.121.204.70:9093"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: 50000012
      KAFKA_MESSAGE_MAX_BYTES: 50000012
  kafka-3:
    image: confluentinc/cp-kafka
    depends_on:
      - zookeeper
    ports:
      - "9094:9094"
    volumes:
      - /kafkadata/3:/var/lib/kafka/data
    restart: unless-stopped
    environment:
      KAFKA_ZOOKEEPER_CONNECT: "10.121.204.70:2181"
      KAFKA_BROKER_ID: "3"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://10.121.204.70:9094"      
      KAFKA_SOCKET_REQUEST_MAX_BYTES: 50000012
      KAFKA_MESSAGE_MAX_BYTES: 50000012
  kafdrop:
    image: thomsch98/kafdrop:latest
    depends_on:
      - zookeeper
    ports:
      - "10000:9000"
    restart: unless-stopped
    environment:
      ZK_HOSTS: "10.121.204.70:2181"
```
## Test scrittura su topic
### 5 min
TODO: scrittura su topic tramite console_producer
## Visualizzazione cluster kafka tramite Kafdrop
### 15 min
TODO: Visualizzazione cluster e topic
