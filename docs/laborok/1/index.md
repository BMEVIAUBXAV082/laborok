# Batch adatfeldolgozó pipeline

A labor során egy valósághű e-commerce batch adatpipeline-t építünk fel a nulláról. Szintetikus webshop adatokat generálunk, azokat PostgreSQL adatbázisba töltjük, majd DuckDB segítségével staging, cleaned és gold rétegeket alakítunk ki. Megvizsgáljuk az adatminőség validálásának módszereit, particionált Parquet fájlokat írunk és mérjük a predicate pushdown hatását, végül haladó OLAP aggregációkat futtatunk. A labor utolsó részében egy Slowly Changing Dimension Type 2 dimenziótáblát implementálunk, és összehasonlítjuk a PostgreSQL és a DuckDB lekérdezési teljesítményét.

## Előkészület

A feladatok megoldásához az alábbi telepített szoftverekre van szükség:

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) vagy egyéb Docker-konténer futtatására alkalmas környezet
* [Git](https://git-scm.com/)
* [VS Code](https://code.visualstudio.com/) vagy egyéb kódszerkesztő (opcionális)

!!! warning "Fontos"
    Ezen a laboron **nem kell Python-t** telepíteni a saját gépünkre. Minden kód Docker-ben futó JupyterLab-ban fut. A Python csomagokat a notebookban telepítjük, közvetlenül az első cellában.

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
        environment:
          POSTGRES_USER: dataeng
          POSTGRES_PASSWORD: dataeng
          POSTGRES_DB: labor
        ports:
          - "5432:5432"
        volumes:
          - pgdata:/var/lib/postgresql/data

      jupyter:
        image: jupyter/scipy-notebook:latest
        ports:
          - "8888:8888"
        environment:
          JUPYTER_ENABLE_LAB: "yes"
          JUPYTER_TOKEN: "labor"
        volumes:
          - ./notebooks:/home/jovyan/work
        depends_on:
          - postgres

    volumes:
      pgdata:
    ```

    !!! note "A `volumes` bejegyzés szerepe"
        A `./notebooks:/home/jovyan/work` bejegyzés biztosítja, hogy a notebookjaink a repository `notebooks/` mappájában tárolódjanak. Így `git commit`-tal beadhatók, és a Docker újraindítása után is megmaradnak.

6. Indítsuk el a konténereket:

    ```powershell
    PS> docker compose up -d
    ```

7. Ellenőrizzük, hogy mindkét konténer fut:

    ```powershell
    PS> docker compose ps
    ```

8. Hozzunk létre egy `notebooks/` mappát a repository gyökerében (ha a Docker még nem hozta létre), majd nyissuk meg a JupyterLab-ot böngészőben:

    **[http://localhost:8888](http://localhost:8888)** – Token: `labor`

    !!! tip "Tipp"
        Ezentúl minden feladatot a JupyterLab felületen belül végezzünk el. Mentés után a notebookokat a `notebooks/` mappából commitoljuk a Git repositoryba.

---