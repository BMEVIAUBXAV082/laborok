# Adatmodellezés és adattárolási absztrakciók

A labor során egy klasszikus adattárház-pipeline-t és modern Lakehouse megoldást építünk fel. Szintetikus OLTP adatokat generálunk PostgreSQL-be, majd DuckDB-ben csillag sémát tervezünk és töltünk fel. Ezt követően megismerjük a lassan változó dimenziók (SCD Type 2) kezelését, a Delta Lake-alapú Medallion Lakehouse architektúrát (Bronze → Silver → Gold), végül egy teljes dbt ELT pipeline-t futtatunk seeds, staging modellek, mart modellek, sématesztek és lineage dokumentáció segítségével.

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

      postgres:
        image: postgres:16
        container_name: labor04-postgres
        environment:
          POSTGRES_USER: labor             # Adatbázis-felhasználónév
          POSTGRES_PASSWORD: labor         # Jelszó (csak fejlesztői környezetben ilyen egyszerű!)
          POSTGRES_DB: oltp_source         # A létrehozandó adatbázis neve
        ports:
          - "5432:5432"                    # A PostgreSQL alapértelmezett portja
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U labor -d oltp_source"]
          interval: 10s                    # 10 másodpercenként ellenőrzi az állapotot
          timeout: 5s
          retries: 5

      jupyter:
        image: jupyter/scipy-notebook:latest
        container_name: labor04-jupyter
        depends_on:
          postgres:
            condition: service_healthy     # JupyterLab csak egészséges PostgreSQL után indul
        ports:
          - "8888:8888"                    # A böngészőből ezen a porton érjük el a JupyterLab-ot
          - "8081:8081"                    # dbt docs serve portja
        environment:
          JUPYTER_TOKEN: "datamodeling"    # Bejelentkezési token
          JUPYTER_ENABLE_LAB: "yes"        # JupyterLab felület (nem a klasszikus Jupyter)
        volumes:
          - ./notebooks:/home/jovyan/work  # A helyi notebooks/ mappa megjelenik /work-ként
    ```

    !!! note "A két service szerepe"
        * **postgres** – OLTP (Online Transaction Processing) forrásadatbázis. Ide kerülnek a szintetikus üzleti adatok (customers, products, orders). A laboron úgy kezeljük, mintha egy valós termelési rendszerből érkezne – mi csak olvasunk belőle az ETL során.
        * **jupyter** – JupyterLab notebookkörnyezet. A Python kódunkat itt írjuk és futtatjuk. A `/work` mappán belüli fájlok a `notebooks/` helyi mappában is megjelennek, így a képernyőképek beadásához nem kell belépni a konténerbe.

6. Indítsuk el a konténereket:

    ```powershell
    PS> docker compose up -d
    ```

    A `-d` (detached) kapcsoló háttérben indítja a konténereket. Az első indítás tovább tarthat, mert le kell tölteni az image-eket.

7. Ellenőrizzük, hogy mindkét konténer fut:

    ```powershell
    PS> docker compose ps
    ```

    A `State` oszlopban minden sornál `running` vagy `Up` feliratnak kell megjelennie. Ha a `labor04-postgres` `Exit` állapotban van, nézzük meg a logját: `docker compose logs postgres`.

8. Hozzunk létre egy `notebooks/` mappát a repository gyökerében (ha a Docker még nem hozta létre automatikusan).

9. Nyissuk meg a JupyterLab-ot böngészőben:

    **[http://localhost:8888](http://localhost:8888)** – Token: `datamodeling`

    !!! tip "Tipp"
        A JupyterLab bal oldali fájlböngészőjében a `/work` mappában dolgozunk. Az itt létrehozott notebookok a helyi `notebooks/` mappában is megjelennek – ezekből készítjük majd a beadandó képernyőképeket.

---




