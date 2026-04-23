# Orchestration – Airflow, Prefect, Dagster és Great Expectations

A labor során az adatpipeline-ok ütemezésének és felügyeletének négy modern eszközét ismerjük meg. Az **Apache Airflow** az iparág de facto szabványa: DAG-alapú, deklaratív, gazdag UI-jal rendelkező orkesztrátor. A **Prefect** egy újabb generációs megközelítés, ahol a Python kód az elsődleges, a workflow struktúra dekorátorokból épül fel. A **Dagster** az asset-alapú (adattermék-orientált) paradigmát képviseli. A **Great Expectations** egy adat-minőség-ellenőrző keretrendszer, amellyel deklaratív elvárásokat fogalmazunk meg az adatokra, és automatikusan érvényesítjük őket a pipeline minden lépésénél.

## Előkészület

A feladatok megoldásához az alábbi telepített szoftverekre van szükség:

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) vagy egyéb Docker-konténer futtatására alkalmas környezet
* [Git](https://git-scm.com/)
* [VS Code](https://code.visualstudio.com/) vagy egyéb kódszerkesztő (opcionális)

!!! warning "Fontos"
    Ezen a laboron **nem kell Python-t** telepíteni a saját gépünkre. A Prefect, Dagster és Great Expectations kód JupyterLab-ban fut; az Airflow DAG-ok az Airflow konténerekben hajtódnak végre.

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

      postgres-oltp:
        image: postgres:16
        container_name: labor05-postgres-oltp
        environment:
          POSTGRES_USER: labor
          POSTGRES_PASSWORD: labor
          POSTGRES_DB: oltp_source
        ports:
          - "5432:5432"
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U labor -d oltp_source"]
          interval: 10s
          timeout: 5s
          retries: 5

      postgres-airflow:
        image: postgres:16
        container_name: labor05-postgres-airflow
        environment:
          POSTGRES_USER: airflow
          POSTGRES_PASSWORD: airflow
          POSTGRES_DB: airflow
        ports:
          - "5433:5432"
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U airflow -d airflow"]
          interval: 10s
          timeout: 5s
          retries: 5

      airflow-init:
        image: apache/airflow:2.9.2
        container_name: labor05-airflow-init
        depends_on:
          postgres-airflow:
            condition: service_healthy
        environment:
          AIRFLOW__CORE__EXECUTOR: LocalExecutor
          AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres-airflow/airflow
          AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
        volumes:
          - ./dags:/opt/airflow/dags
        entrypoint: /bin/bash
        command:
          - -c
          - |
            airflow db migrate &&
            airflow users create \
              --username admin \
              --password admin \
              --firstname Admin \
              --lastname User \
              --role Admin \
              --email admin@example.com

      airflow-webserver:
        image: apache/airflow:2.9.2
        container_name: labor05-airflow-webserver
        depends_on:
          postgres-airflow:
            condition: service_healthy
          airflow-init:
            condition: service_completed_successfully
        environment:
          AIRFLOW__CORE__EXECUTOR: LocalExecutor
          AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres-airflow/airflow
          AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
          AIRFLOW__WEBSERVER__SECRET_KEY: 'labor05secret'
          _PIP_ADDITIONAL_REQUIREMENTS: 'pandas sqlalchemy psycopg2-binary pyarrow great_expectations'
        volumes:
          - ./dags:/opt/airflow/dags
          - ./logs:/opt/airflow/logs
          - ./data:/opt/airflow/data
        ports:
          - "8080:8080"
        command: airflow webserver
        healthcheck:
          test: ["CMD", "curl", "--fail", "http://localhost:8080/health"]
          interval: 30s
          timeout: 10s
          retries: 5
          start_period: 60s

      airflow-scheduler:
        image: apache/airflow:2.9.2
        container_name: labor05-airflow-scheduler
        depends_on:
          postgres-airflow:
            condition: service_healthy
          airflow-init:
            condition: service_completed_successfully
        environment:
          AIRFLOW__CORE__EXECUTOR: LocalExecutor
          AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres-airflow/airflow
          AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
          _PIP_ADDITIONAL_REQUIREMENTS: 'pandas sqlalchemy psycopg2-binary pyarrow great_expectations'
        volumes:
          - ./dags:/opt/airflow/dags
          - ./logs:/opt/airflow/logs
          - ./data:/opt/airflow/data
        command: airflow scheduler

      jupyter:
        image: jupyter/scipy-notebook:latest
        container_name: labor05-jupyter
        depends_on:
          postgres-oltp:
            condition: service_healthy
        ports:
          - "8888:8888"
        environment:
          JUPYTER_TOKEN: "orchestration"
          JUPYTER_ENABLE_LAB: "yes"
        volumes:
          - ./notebooks:/home/jovyan/work
          - ./dags:/home/jovyan/dags
          - ./data:/home/jovyan/data
    ```

    !!! note "A hat service szerepe"
        * **postgres-oltp** – OLTP forrásadatbázis (customers, orders, stb.). Minden pipeline ebből olvas.
        * **postgres-airflow** – Airflow metaadat-adatbázis: DAG definíciók, futási állapotok, XCom értékek. Külső portja `5433`, hogy ne ütközzön az OLTP-vel.
        * **airflow-init** – Egyszeri inicializáló: `airflow db migrate` + admin user létrehozás. `Exit 0` állapot normális – ne aggódjunk.
        * **airflow-webserver** – Airflow UI, port 8080. Csak az init befejezése után indul.
        * **airflow-scheduler** – Háttérfolyamat, 30 másodpercenként beolvassa a DAG fájlokat.
        * **jupyter** – Prefect, Dagster, GE kódok futtatási környezete. A `./dags` és `./data` mappák mindkét helyre mountoltak.

6. Hozzuk létre a szükséges helyi mappákat:

    ```powershell
    PS> mkdir dags, logs, data, notebooks
    ```

    !!! note "Miért kell előre létrehozni?"
        Ha a mappák nem léteznek `docker compose up` előtt, a Docker root tulajdonossal hozza létre őket. Az Airflow konténer nem-root userként fut, ezért a logs mappába nem tudna írni. Saját userrel előre létrehozva ez a probléma elkerülhető.

7. Indítsuk el a konténereket:

    ```powershell
    PS> docker compose up -d
    ```

    Az első indítás **10–15 percet** vehet igénybe (image-ek letöltése + `_PIP_ADDITIONAL_REQUIREMENTS` telepítése).

8. Kövessük az inicializálás állapotát:

    ```powershell
    PS> docker compose logs -f airflow-init
    ```

    Sikeres, ha a log végén: `Admin user admin created`.

9. Ellenőrizzük az összes service állapotát:

    ```powershell
    PS> docker compose ps
    ```

10. Nyissuk meg a webes felületeket:

    * **Airflow UI:** **[http://localhost:8080](http://localhost:8080)** – `admin` / `admin`
    * **JupyterLab:** **[http://localhost:8888](http://localhost:8888)** – Token: `orchestration`

    !!! tip "Airflow lassan indul"
        Az első induláskor 3–5 percet vesz igénybe a `_PIP_ADDITIONAL_REQUIREMENTS` telepítése. Kövessük: `docker compose logs -f airflow-webserver`.

---
