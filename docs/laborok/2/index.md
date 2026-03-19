# Streaming alapú adatfeldolgozás

A labor során egy streaming alapú adatfeldolgozásra épülő pipeline-t építünk ki. Szintetikus rendelési eseményeket (OrderEvent) generálunk, azokat Apache Kafka-ba publikáljuk, majd különböző fogyasztói mintákat valósítunk meg: sémavalidációt, consumer group-os párhuzamosítást, tumbling window aggregációt, Dead Letter Queue kezelést és Parquet formátumú exportot. A labor utolsó részében az összes tanult komponenst egy end-to-end pipeline-ba fogjuk össze.

## Előkészület

A feladatok megoldásához az alábbi telepített szoftverekre van szükség:

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) vagy egyéb Docker-konténer futtatására alkalmas környezet
* [Git](https://git-scm.com/)
* [VS Code](https://code.visualstudio.com/) vagy egyéb kódszerkesztő (opcionális)

!!! warning "Fontos"
    Ezen a laboron **nem kell Python-t** telepíteni a saját gépünkre. Minden kód Docker-ben futó JupyterLab-ban fut. A Python csomagokat a notebookokban telepítjük, közvetlenül az első cellában.

A feladatok megoldása során ne felejtsük el követni a feladatbeadás folyamatát, amiről [itt olvashatsz részletesen](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keressük meg a laborhoz tartozó meghívó URL-jét, és annak segítségével hozzuk létre a saját repository-nkat.
2. Várjuk meg, míg elkészül a repository, majd checkout-oljuk ki.
3. Hozzunk létre egy új ágat `megoldas` néven, és ezen az ágon dolgozzunk.
4. A neptun.txt fájlba írjuk bele a Neptun-kódunkat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun-kód 6 karaktere.

### Docker környezet indítása

5. Hozzunk létre egy `docker-compose.yaml` fájlt a repository gyökerében az alábbi tartalommal:

    ```yaml
    services:

      kafka:
        image: apache/kafka:3.9.2
        container_name: labor03-kafka
        environment:
          # KRaft mód – Zookeeper nélküli, beépített konszenzus-mechanizmus
          KAFKA_NODE_ID: 1
          KAFKA_PROCESS_ROLES: broker,controller              # Ez a node egyszerre broker és controller
          KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093       # A controller cluster tagjai
          # Listenerek
          KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
          KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092  # Ezt a címet hirdeti a clientek felé
          KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
          KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
          KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
          KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"             # Topic automatikus létrehozása küldéskor
          KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
          KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
        ports:
          - "9092:9092"                    # Kafka broker portja – erre csatlakoznak a producerek és consumerek
        healthcheck:
          test: ["CMD-SHELL", "/opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list"]
          interval: 15s                    # 15 másodpercenként ellenőrzi, hogy a broker válaszol-e
          timeout: 10s
          retries: 5

      kafka-ui:
        image: provectuslabs/kafka-ui:latest
        container_name: labor03-kafka-ui
        depends_on:
          kafka:
            condition: service_healthy     # Csak egészséges Kafka után indul el a UI
        ports:
          - "8080:8080"                    # A böngészőből ezen a porton érjük el a webes felületet
        environment:
          KAFKA_CLUSTERS_0_NAME: local                         # A cluster megjelenítési neve a UI-on
          KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092        # Melyik Kafka brokerhez csatlakozzon

      jupyter:
        image: jupyter/scipy-notebook:latest
        container_name: labor03-jupyter
        depends_on:
          kafka:
            condition: service_healthy     # JupyterLab is csak akkor indul, ha Kafka már kész
        ports:
          - "8888:8888"                    # A böngészőből ezen a porton nyitjuk meg a JupyterLabot
        environment:
          JUPYTER_TOKEN: "streaming"       # Bejelentkezési token – ezt kell megadni a böngészőben
          JUPYTER_ENABLE_LAB: "yes"        # A klasszikus Jupyter helyett JupyterLab felületet indít
        volumes:
          - ./notebooks:/home/jovyan/work  # A helyi notebooks/ mappa megjelenik a konténer /work mappájaként
    ```

    !!! note "A három service szerepe"
        * **kafka** – Az üzenetközvetítő broker. Az Apache Software Foundation hivatalos `apache/kafka` image-ét használja. KRaft módban fut, vagyis Zookeeper nélkül, beépített konszenzus-mechanizmussal koordinálja saját magát. A producerek ide küldik az üzeneteket, a consumerek innen olvassák őket.
        * **kafka-ui** – Webes adminisztrációs felület. Böngészőből nyomon követhetjük a topicokat, üzeneteket, consumer group-okat és offset-eket.
        * **jupyter** – JupyterLab notebookkörnyezet. A Python kódunkat itt írjuk és futtatjuk interaktívan.

6. Indítsuk el a konténereket:

    ```powershell
    PS> docker compose up -d
    ```

    A `-d` (detached) kapcsoló háttérben indítja a konténereket, így a terminálunk szabadon marad. Az első indítás tovább tarthat, mert le kell tölteni a Docker image-eket.

7. Ellenőrizzük, hogy mind a három konténer fut:

    ```powershell
    PS> docker compose ps
    ```

    A `State` oszlopban minden sornál `running` vagy `Up` feliratnak kell megjelennie. Ha valamelyik `Exit` állapotban van, nézzük meg a logját: `docker compose logs kafka`.

8. Hozzunk létre egy `notebooks/` mappát a repository gyökerében (ha a Docker még nem hozta létre), majd nyissuk meg a JupyterLab-ot böngészőben:

    **[http://localhost:8888](http://localhost:8888)** – Token: `streaming`

9. Nyissuk meg a Kafka UI-t is egy másik böngészőlapon:

    **[http://localhost:8080](http://localhost:8080)**

    !!! tip "Tipp"
        A Kafka UI-on a *Topics* menüpontban nyomon követhetjük, hogy az egyes feladatok során milyen üzenetek kerülnek a topicokba. A *Consumer Groups* fülön láthatjuk, melyik csoport hol tart az olvasásban (lag értékkel együtt). Érdemes ezt a lapot az egész labor során nyitva tartani.

---
