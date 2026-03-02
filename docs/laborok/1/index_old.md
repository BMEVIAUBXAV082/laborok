# Adatforrások, adatgenerálás

A labor során megismerkedsz a leggyakoribb adatforrásokkal és adatformátumokkal, gyakorlatot szerzel adatkinyerésben különböző forrásokból, valamint megtanulod, hogyan lehet valósághű szintetikus adatokat generálni. A labor célja, hogy tapasztalatot szerezz a data engineering gyakorlati alkalmazásában.

## Előkészület

A feladatok megoldásához az alábbi telepített szoftverekre van szükség (alternatívaként használhatjuk a [BME Cloud](https://cloud.bme.hu/) egyik virtuális gépét):

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) vagy egyéb Docker-konténer futtatására alkalmas környezet
* [Python](https://www.python.org/) futtatókörnyezet (3.12-es verzió)
* [VS Code](https://code.visualstudio.com/) vagy egyéb kódszerkesztő
* [Git](https://git-scm.com/)
* [Poetry](https://python-poetry.org/) vagy egyéb függőségkezelő (opcionális, de erősen javasolt)

!!! warning "Technológiák"
    A labor során több (számodra akár teljesen új) technológiával is kell dolgoznod. A data engineering világában a fejlesztők rendszeresen találkoznak különböző adatforrásokkal, formátumokkal és eszközökkel, amelyekhez gyorsan kell alkalmazkodniuk. Előfordulhat, hogy bizonyos lépések értelmét csak később látod meg, de így könnyebben megérted majd, hogyan működik egy valós adatfeldolgozó rendszer.

A feladatok megoldása során ne felejtsd el követni a feladatbeadás folyamatát, amiről [itt olvashatsz részletesen](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keresd meg a laborhoz tartozó meghívó URL-jét, és annak segítségével hozd létre a saját repository-dat.
2. Várd meg, míg elkészül a repository, majd checkout-old ki.
3. Hozz létre egy új ágat `megoldas` néven, és ezen az ágon dolgozz.
4. A neptun.txt fájlba írd bele a Neptun-kódodat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun-kód 6 karaktere.

### Virtuális környezet létrehozása

Már most érdemes létrehozni egy virtuális környezetet a projekt gyökerében, és azt aktiválni. A virtuális környezet célja, hogy a Python csomagokat, amiket használni fogsz, ne globálisan kelljen feltelepíteni, hanem csak az adott projektre vonatkozóan.

!!! tip "Tipp"
    A virtuális környezet létrehozásához az alábbi leírás a `poetry`-t javasolja. Ettől saját felelősségre el lehet térni, de erősen ajánlott ennek használata. Ha elakadnátok, bátran kérjetek segítséget.

1. Nyiss meg egy parancssort a projekt gyökerében. Mivel a projektet üresen hoztuk létre, elsőként inicializáljuk a Poetry-t:

    ```powershell
    PS> poetry init --no-interaction --python "^3.12"
    ```

    Ez létrehozza a `pyproject.toml` konfigurációs fájlt.

2. Hozd létre a virtuális környezetet és aktiváld:

    ```powershell
    PS> poetry install
    PS> .\.venv\Scripts\activate
    ```

    !!! note "Megjegyzés"
        A parancssoros környezettől függően ez eltérő lehet, tájékozódjunk az opciókról a [dokumentációban](https://python-poetry.org/docs/managing-environments/#activating-the-environment). Az útmutató példái Windows-on PowerShell használatára építenek, ezeket az utasításokat mindig a projekt gyökerében kell kiadni. Ha a `PS>` előtagot látjátok, az a parancssorra utal, ezt nem kell beírni.

3. Sikeres aktiváció után a parancssorban megjelenik a virtuális környezet neve a sor elején.

4. Ezután minden parancsot ebben a parancssorban adjunk ki, különben egyes komponenseket lehet, hogy nem fog megtalálni a rendszer.

5. Hozz létre egy `data` mappát a projekt gyökerében, ide fogjuk menteni az adatfájlokat.

    ```powershell
    PS> mkdir data
    ```

## 1. feladat: Adatformátumok

A data engineering egyik alapvető feladata az adatok különböző formátumokban való kezelése. Ebben a feladatban megismerkedsz a három legelterjedtebb adatformátummal: CSV, JSON és Parquet. Megtanulod, hogyan lehet közöttük konvertálni, és megérted, mikor melyiket érdemes használni.

### Függőségek telepítése

1. Telepítsd a szükséges csomagokat:

    ```powershell
    PS> poetry add pandas pyarrow
    ```

    * A [`pandas`](https://pandas.pydata.org/) a Python legelterjedtebb adatkezelő könyvtára.
    * A [`pyarrow`](https://arrow.apache.org/docs/python/) az Apache Arrow Python implementációja, amely a Parquet formátum kezeléséhez szükséges.

### Mintaadatok létrehozása

2. Hozz létre egy `formats.py` fájlt a projekt gyökerében. Ebben a fájlban fogunk dolgozni az adatformátumokkal.

    ```python
    def main():
        # Ide kerül a logika

    if __name__ == '__main__':
        main()
    ```

3. Készíts egy minta adatkészletet `pandas` segítségével. Legyen ez egy egyszerű termékadatbázis:

    ```python
    import pandas as pd

    data = {
        "id": range(1, 11),
        "name": [
            "Laptop", "Egér", "Billentyűzet", "Monitor", "Webkamera",
            "Fejhallgató", "USB Hub", "Pendrive", "SSD", "Nyomtató"
        ],
        "category": [
            "Számítógép", "Periféria", "Periféria", "Kijelző", "Periféria",
            "Audio", "Periféria", "Tárolás", "Tárolás", "Periféria"
        ],
        "price": [350000, 8500, 15000, 120000, 12000, 25000, 5500, 3200, 28000, 45000],
        "stock": [15, 120, 85, 30, 60, 45, 200, 300, 75, 20],
        "created_at": pd.date_range("2025-01-01", periods=10, freq="D"),
    }
    df = pd.DataFrame(data)
    print(df)
    print(f"\nAdattípusok:\n{df.dtypes}")
    ```

4. Futtasd a scriptet és ellenőrizd a kimenetet:

    ```powershell
    PS> python formats.py
    ```

### Mentés különböző formátumokban

5. Mentsd el az adatokat CSV, JSON és Parquet formátumban:

    ```python
    csv_path = "data/products.csv"
    json_path = "data/products.json"
    parquet_path = "data/products.parquet"

    df.to_csv(csv_path, index=False)
    df.to_json(json_path, orient="records", indent=2, force_ascii=False)
    df.to_parquet(parquet_path, index=False)
    ```

    !!! note "JSON orient paraméter"
        A `to_json()` metódus `orient` paramétere határozza meg a JSON struktúrát. A `"records"` érték egy listát hoz létre, ahol minden elem egy-egy sor. Más lehetőségek: `"columns"` (oszlopok szerinti csoportosítás), `"index"` (index szerinti), `"split"` (külön fejléc és adat). A `force_ascii=False` paraméter biztosítja, hogy az ékezetes karakterek helyesen jelenjenek meg.

### Fájlméret összehasonlítás

6. Hasonlítsd össze a három formátum fájlméretét:

    ```python
    import os

    for path, fmt in [(csv_path, "CSV"), (json_path, "JSON"), (parquet_path, "Parquet")]:
        size = os.path.getsize(path)
        print(f"{fmt:>10}: {size:>6} byte")
    ```

    !!! tip "Miért különböznek a méretek?"
        A CSV egyszerű szöveges formátum, a JSON szöveges de strukturáltabb (emiatt általában nagyobb), a Parquet pedig bináris, oszlopos formátum, ami tömörítést is alkalmaz. Kis adathalmazoknál a különbség nem feltűnő, de nagy (millió+ soros) adatkészleteknél a Parquet akár 10x-es méretcsökkenést is eredményezhet a CSV-hez képest.

### Visszaolvasás és sebesség mérés

7. Olvasd vissza az adatokat mindhárom formátumból, és mérd meg az olvasási sebességet:

    ```python
    import time

    def measure_read(read_func, path, n=100):
        start = time.time()
        for _ in range(n):
            read_func(path)
        elapsed = (time.time() - start) / n * 1000
        return elapsed

    csv_time = measure_read(pd.read_csv, csv_path)
    json_time = measure_read(pd.read_json, json_path)
    parquet_time = measure_read(pd.read_parquet, parquet_path)

    print(f"\nÁtlagos olvasási idő ({n} iteráció):")
    print(f"  CSV:     {csv_time:.2f} ms")
    print(f"  JSON:    {json_time:.2f} ms")
    print(f"  Parquet: {parquet_time:.2f} ms")
    ```

### Parquet séma vizsgálat

8. Vizsgáld meg a Parquet fájl sémáját:

    ```python
    import pyarrow.parquet as pq

    schema = pq.read_schema(parquet_path)
    print(f"\nParquet séma:")
    print(schema)
    print(f"\nMetaadatok: {schema.metadata}")
    ```

    !!! note "Mikor melyik formátumot használjuk?"
        * **CSV**: Egyszerű, emberileg olvasható, univerzálisan támogatott. Ideális kis adathalmazokhoz és adatcseréhez. Hátránya, hogy nem tartalmaz típusinformációt és nem tömörít.
        * **JSON**: Hierarchikus adatok tárolására alkalmas, API-k alapértelmezett formátuma. Hátránya, hogy a redundáns kulcsnevek miatt nagyobb fájlméretet eredményez.
        * **Parquet**: Oszlopos, bináris formátum tömörítéssel és séma-információval. Ideális nagy adathalmazokhoz és analitikai lekérdezésekhez. Hátránya, hogy nem emberileg olvasható.

9. Commitold a változtatásokat.

### Beadandó

!!! example "1. feladat beadandó"
    * Commitold a változtatásokat.
    * Készíts egy képernyőképet a fájlméret-összehasonlítás és olvasási sebesség kimenetéről, és mentsd el a repository gyökerébe **`f1.png`** néven.

## 2. feladat: Adatkinyerés REST API-ból

Az adatforrások jelentős része ma már REST API-kon keresztül érhető el. Ebben a feladatban az [Open-Meteo](https://open-meteo.com/) ingyenes időjárás API-t fogod használni, amely regisztráció nélkül elérhető. Megtanulod, hogyan kell API-hoz kapcsolódni, a válaszokat feldolgozni, és az adatokat elmenteni.

### Egyszerű API lekérdezés

1. Telepítsd a `requests` csomagot:

    ```powershell
    PS> poetry add requests
    ```

2. Hozz létre egy `api_extract.py` fájlt a projekt gyökerében:

    ```python
    def main():
        # Ide kerül a logika

    if __name__ == '__main__':
        main()
    ```

3. Kérdezd le Budapest óránkénti időjárási adatait egy hétre:

    ```python
    import requests

    url = "https://api.open-meteo.com/v1/forecast"
    params = {
        "latitude": 47.4979,
        "longitude": 19.0402,
        "hourly": "temperature_2m,precipitation,wind_speed_10m",
        "start_date": "2025-01-01",
        "end_date": "2025-01-07",
    }

    response = requests.get(url, params=params, timeout=10)
    response.raise_for_status()
    data = response.json()
    ```

    !!! note "API paraméterek"
        Az Open-Meteo API számos paramétert támogat. A `latitude` és `longitude` a lekérdezni kívánt helyszín koordinátái, a `hourly` a kívánt mérési értékek (hőmérséklet, csapadék, szélsebesség), a `start_date` és `end_date` pedig az időtartomány. A teljes API dokumentáció elérhető a [https://open-meteo.com/en/docs](https://open-meteo.com/en/docs) címen.

4. Vizsgáld meg a válasz struktúráját:

    ```python
    import json

    print(json.dumps(data, indent=2)[:500])
    ```

### Válasz feldolgozása

5. Alakítsd a válaszban kapott adatokat `pandas` DataFrame-mé:

    ```python
    import pandas as pd

    hourly = data["hourly"]
    df = pd.DataFrame({
        "time": pd.to_datetime(hourly["time"]),
        "temperature_c": hourly["temperature_2m"],
        "precipitation_mm": hourly["precipitation"],
        "wind_speed_kmh": hourly["wind_speed_10m"],
    })
    print(df.head(10))
    print(f"\nÖsszesen {len(df)} óránkénti mérés.")
    ```

### Több város lekérdezése

6. Kérdezd le több közép-európai város adatait is. Definiálj egy városlistát, és iterálj végig rajta:

    ```python
    cities = {
        "Budapest": {"latitude": 47.4979, "longitude": 19.0402},
        "Bécs": {"latitude": 48.2082, "longitude": 16.3738},
        "Prága": {"latitude": 50.0755, "longitude": 14.4378},
        "Pozsony": {"latitude": 48.1486, "longitude": 17.1077},
        "Zágráb": {"latitude": 45.8150, "longitude": 15.9819},
    }

    all_data = []
    for city_name, coords in cities.items():
        params = {
            **coords,
            "hourly": "temperature_2m,precipitation,wind_speed_10m",
            "start_date": "2025-01-01",
            "end_date": "2025-01-07",
        }
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        city_data = response.json()

        hourly = city_data["hourly"]
        city_df = pd.DataFrame({
            "city": city_name,
            "time": pd.to_datetime(hourly["time"]),
            "temperature_c": hourly["temperature_2m"],
            "precipitation_mm": hourly["precipitation"],
            "wind_speed_kmh": hourly["wind_speed_10m"],
        })
        all_data.append(city_df)
        print(f"{city_name}: {len(city_df)} mérés lekérdezve.")

    combined_df = pd.concat(all_data, ignore_index=True)
    print(f"\nÖsszesített adathalmaz: {len(combined_df)} sor")
    ```

    !!! warning "API rate limiting"
        Bár az Open-Meteo API nagyvonalú limitekkel rendelkezik, éles környezetben mindig figyeljünk az API rate limitjeire. Érdemes késleltetést (pl. `time.sleep(0.5)`) beiktatni a kérések közé, ha sok lekérdezést végzünk egymás után. Az API válaszában található HTTP fejlécek gyakran tartalmazzák a hátralévő kérések számát.

### Historikus adatok lekérése

7. Az Open-Meteo rendelkezik egy Archive API-val is, amellyel historikus adatokat lehet lekérni. Kérdezd le Budapest 2024-es éves hőmérsékleti adatait:

    ```python
    archive_url = "https://archive-api.open-meteo.com/v1/archive"
    archive_params = {
        "latitude": 47.4979,
        "longitude": 19.0402,
        "daily": "temperature_2m_max,temperature_2m_min,precipitation_sum",
        "start_date": "2024-01-01",
        "end_date": "2024-12-31",
    }

    response = requests.get(archive_url, params=archive_params, timeout=30)
    response.raise_for_status()
    archive_data = response.json()

    daily = archive_data["daily"]
    archive_df = pd.DataFrame({
        "date": pd.to_datetime(daily["time"]),
        "temp_max_c": daily["temperature_2m_max"],
        "temp_min_c": daily["temperature_2m_min"],
        "precipitation_mm": daily["precipitation_sum"],
    })
    print(f"\nHistorikus adatok: {len(archive_df)} nap")
    print(archive_df.describe())
    ```

### Adatok mentése

8. Mentsd el az összegyűjtött adatokat az 1. feladatban megismert formátumokban:

    ```python
    combined_df.to_csv("data/weather_cities.csv", index=False)
    combined_df.to_parquet("data/weather_cities.parquet", index=False)
    archive_df.to_csv("data/weather_budapest_2024.csv", index=False)
    archive_df.to_parquet("data/weather_budapest_2024.parquet", index=False)
    ```

    !!! tip "Best practice: API lekérdezések"
        * Mindig állíts be `timeout` paramétert a `requests.get()` hívásban, hogy a program ne akadjon meg, ha az API nem válaszol.
        * Használd a `response.raise_for_status()` metódust a HTTP hibakódok automatikus kezelésére.
        * Éles környezetben érdemes `requests.Session()` objektumot használni, ami újrahasznosítja a kapcsolatot, így gyorsabb lesz a kommunikáció.
        * Az API válaszokat érdemes lokálisan cache-elni, hogy ne kelljen minden alkalommal újra lekérdezni.

9. Commitold a változtatásokat.

### Beadandó

!!! example "2. feladat beadandó"
    * Commitold a változtatásokat.
    * Készíts egy képernyőképet a több városból összegyűjtött adatokat tartalmazó DataFrame-ről, és mentsd el a repository gyökerébe **`f2.png`** néven.

## 3. feladat: Adatkinyerés adatbázisból

A vállalati környezetben az adatok jelentős része relációs adatbázisokban található. Ebben a feladatban egy PostgreSQL adatbázist fogsz futtatni Docker konténerben, adatokat töltesz bele, és Python segítségével kérdezel le belőle.

### PostgreSQL indítása Docker-ben

1. Hozz létre egy `docker-compose.yaml` fájlt a projekt gyökerében az alábbi tartalommal:

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

    volumes:
      pgdata:
    ```

    !!! note "Docker Compose"
        A Docker Compose lehetővé teszi, hogy egy YAML fájlban definiáljuk a szükséges szolgáltatásokat, és egyetlen paranccsal elindítsuk őket. A `volumes` bejegyzés biztosítja, hogy az adatbázis tartalma megmaradjon a konténer újraindítása után is. A `ports` bejegyzés a konténer belső portját teszi elérhetővé a gazda gépen.

2. Indítsd el a Docker Desktop alkalmazást, majd indítsd el az adatbázis konténert:

    ```powershell
    PS> docker-compose up -d postgres
    ```

3. Ellenőrizd, hogy a konténer fut:

    ```powershell
    PS> docker-compose ps
    ```

### Kapcsolódás Python-ból

4. Telepítsd a szükséges csomagokat:

    ```powershell
    PS> poetry add psycopg2-binary sqlalchemy
    ```

    * A [`psycopg2`](https://www.psycopg.org/) a PostgreSQL Python adaptere.
    * Az [`SQLAlchemy`](https://www.sqlalchemy.org/) egy Python SQL toolkit és ORM, ami egységes interfészt biztosít különböző adatbázisokhoz.

5. Hozz létre egy `db_extract.py` fájlt a projekt gyökerében:

    ```python
    def main():
        # Ide kerül a logika

    if __name__ == '__main__':
        main()
    ```

6. Kapcsolódj az adatbázishoz SQLAlchemy segítségével:

    ```python
    from sqlalchemy import create_engine

    connection_string = "postgresql://dataeng:dataeng@localhost:5432/labor"
    engine = create_engine(connection_string)
    ```

    !!! note "Connection string"
        A connection string felépítése: `driver://felhasználónév:jelszó@host:port/adatbázisnév`. Ez az formátum univerzális, szinte minden adatbázis-kezelő rendszerrel használható (MySQL, SQLite, Oracle stb.), csak a driver neve változik.

### Adatok betöltése

7. Töltsd be a korábban létrehozott termékadatokat az adatbázisba:

    ```python
    import pandas as pd

    df = pd.read_csv("data/products.csv")
    df.to_sql("products", engine, if_exists="replace", index=False)
    print(f"{len(df)} sor betöltve a 'products' táblába.")
    ```

8. Töltsd be az időjárási adatokat is:

    ```python
    weather_df = pd.read_csv("data/weather_cities.csv")
    weather_df.to_sql("weather", engine, if_exists="replace", index=False)
    print(f"{len(weather_df)} sor betöltve a 'weather' táblába.")
    ```

### SQL lekérdezések

9. Végezz SQL lekérdezéseket Python-ból a `pandas.read_sql()` segítségével:

    1. Egyszerű SELECT — az összes termék lekérdezése:

        ```python
        result = pd.read_sql("SELECT * FROM products", engine)
        print(result)
        ```

    2. Szűrés (WHERE) — 20 000 Ft feletti termékek:

        ```python
        expensive = pd.read_sql(
            "SELECT name, price, category FROM products WHERE price > 20000 ORDER BY price DESC",
            engine,
        )
        print(f"\n20 000 Ft feletti termékek:\n{expensive}")
        ```

    3. Aggregáció (GROUP BY) — kategóriánkénti átlagár és darabszám:

        ```python
        stats = pd.read_sql("""
            SELECT
                category,
                COUNT(*) as count,
                ROUND(AVG(price)) as avg_price,
                SUM(stock) as total_stock
            FROM products
            GROUP BY category
            ORDER BY avg_price DESC
        """, engine)
        print(f"\nKategória statisztikák:\n{stats}")
        ```

    4. Időjárási aggregáció — városonkénti átlaghőmérséklet:

        ```python
        weather_stats = pd.read_sql("""
            SELECT
                city,
                ROUND(AVG(temperature_c)::numeric, 1) as avg_temp,
                ROUND(MAX(temperature_c)::numeric, 1) as max_temp,
                ROUND(MIN(temperature_c)::numeric, 1) as min_temp,
                ROUND(SUM(precipitation_mm)::numeric, 1) as total_precipitation
            FROM weather
            GROUP BY city
            ORDER BY avg_temp DESC
        """, engine)
        print(f"\nVárosok időjárási összesítése:\n{weather_stats}")
        ```

    !!! tip "SQL és pandas"
        A `pd.read_sql()` metódus közvetlenül DataFrame-be tölti a lekérdezés eredményét, így az SQL erejét és a pandas adatelemző képességeit együtt tudjuk használni. Nagyobb adathalmazok esetén érdemes a szűrést és aggregációt SQL oldalon elvégezni (WHERE, GROUP BY), és csak az eredményt betölteni Python-ba.

### Adatok exportálása

10. Exportáld az egyik lekérdezés eredményét Parquet formátumba:

    ```python
    weather_stats.to_parquet("data/weather_stats.parquet", index=False)
    print("\nExportálva: data/weather_stats.parquet")
    ```

11. Commitold a változtatásokat.

### Beadandó

!!! example "3. feladat beadandó"
    * Commitold a változtatásokat.
    * Készíts egy képernyőképet egy SQL lekérdezés eredményéről (DataFrame formátumban), és mentsd el a repository gyökerébe **`f3.png`** néven.

## 4. feladat: Szintetikus adatgenerálás

Fejlesztés és tesztelés során gyakran van szükség nagy mennyiségű valósághű adatra. Valós adatok használata sokszor nem lehetséges (adatvédelem, GDPR), vagy egyszerűen nem áll rendelkezésre elegendő adat. Ilyenkor szintetikus adatgenerálást alkalmazunk. Ebben a feladatban a [Faker](https://faker.readthedocs.io/) könyvtárat fogod használni.

### Faker alapok

1. Telepítsd a Faker csomagot:

    ```powershell
    PS> poetry add faker
    ```

2. Hozz létre egy `generate.py` fájlt a projekt gyökerében:

    ```python
    def main():
        # Ide kerül a logika

    if __name__ == '__main__':
        main()
    ```

3. Ismerkedj meg az alapvető generátorokkal:

    ```python
    from faker import Faker

    fake = Faker()

    print(f"Név:      {fake.name()}")
    print(f"Cím:      {fake.address()}")
    print(f"Email:    {fake.email()}")
    print(f"Dátum:    {fake.date()}")
    print(f"Cég:      {fake.company()}")
    print(f"Szöveg:   {fake.text(max_nb_chars=50)}")
    ```

### Magyar lokalizáció

4. A Faker támogat lokalizációt, így magyar nyelvű adatokat is tudunk generálni:

    ```python
    fake_hu = Faker("hu_HU")

    print(f"\nMagyar név:    {fake_hu.name()}")
    print(f"Magyar cím:    {fake_hu.address()}")
    print(f"Telefonszám:   {fake_hu.phone_number()}")
    ```

    !!! note "Lokalizáció"
        A `Faker("hu_HU")` konstruktor magyar nyelvi beállítással hoz létre egy generátort. A nevek, címek és egyéb adatok az adott nyelvi és kulturális sajátosságoknak megfelelően generálódnak. Az elérhető lokalizációk listája a [Faker dokumentációban](https://faker.readthedocs.io/en/master/locales.html) található.

### Egyedi adatgenerátor

5. Készíts egy függvényt, ami megadott számú felhasználó rekordot generál:

    ```python
    import pandas as pd
    from datetime import datetime

    def generate_users(n, fake):
        records = []
        for _ in range(n):
            records.append({
                "name": fake.name(),
                "email": fake.email(),
                "phone": fake.phone_number(),
                "city": fake.city(),
                "birth_date": fake.date_of_birth(minimum_age=18, maximum_age=80),
                "registered_at": fake.date_time_between(
                    start_date=datetime(2020, 1, 1),
                    end_date=datetime(2025, 6, 1),
                ),
            })
        return pd.DataFrame(records)

    users_df = generate_users(20, fake_hu)
    print(users_df.head(10))
    ```

6. Készíts egy másik generátort tranzakciós adatokhoz:

    ```python
    import random

    def generate_transactions(n, fake, user_ids):
        categories = ["Élelmiszer", "Elektronika", "Ruházat", "Sport", "Könyv", "Egyéb"]
        records = []
        for _ in range(n):
            records.append({
                "transaction_id": fake.uuid4(),
                "user_id": random.choice(user_ids),
                "amount": round(random.uniform(500, 150000), 2),
                "category": random.choice(categories),
                "timestamp": fake.date_time_between(
                    start_date=datetime(2024, 1, 1),
                    end_date=datetime(2025, 6, 1),
                ),
            })
        return pd.DataFrame(records)

    user_ids = list(range(1, 21))
    transactions_df = generate_transactions(50, fake_hu, user_ids)
    print(f"\nTranzakciók:\n{transactions_df.head(10)}")
    ```

### Nagyméretű adathalmaz generálása

7. Generálj nagyobb mennyiségű adatot, és mentsd el:

    ```python
    large_users = generate_users(10000, fake_hu)
    large_transactions = generate_transactions(50000, fake_hu, list(range(1, 10001)))

    large_users.to_csv("data/users.csv", index=False)
    large_users.to_parquet("data/users.parquet", index=False)
    large_transactions.to_parquet("data/transactions.parquet", index=False)

    print(f"\nGenerált felhasználók: {len(large_users)}")
    print(f"Generált tranzakciók: {len(large_transactions)}")
    ```

    !!! tip "Mikor használjunk szintetikus adatot?"
        * **Fejlesztés és tesztelés**: A fejlesztők reális méretű és formátumú adatokkal tudják tesztelni a kódjukat.
        * **Adatvédelem (GDPR)**: Valós személyes adatok helyett szintetikus adatokat használunk, így nem kockáztatjuk az adatvédelmi szabályok megsértését.
        * **Prototípusok**: Amikor az alkalmazás még fejlesztés alatt áll, és a valós adat még nem elérhető.
        * **Terheléstesztelés**: Nagy mennyiségű adat generálásával tesztelhetjük a rendszer teljesítményét.

### Ismételhetőség

8. Állíts be seed értéket, hogy a generált adatok reprodukálhatók legyenek:

    ```python
    Faker.seed(42)
    random.seed(42)

    reproducible_users = generate_users(5, fake_hu)
    print(f"\nReprodukálható adatok:\n{reproducible_users}")
    ```

    !!! warning "Ismételhetőség fontossága"
        Ha a generált adatokat tesztekhez vagy bemutatókhoz használod, elengedhetetlen, hogy a seed értéket beállítsd. Így minden futtatáskor ugyanazokat az adatokat kapod, ami reprodukálhatóvá teszi az eredményeket.

### Adatok betöltése az adatbázisba

9. Töltsd be a generált adatokat a korábban létrehozott PostgreSQL adatbázisba:

    ```python
    from sqlalchemy import create_engine

    engine = create_engine("postgresql://dataeng:dataeng@localhost:5432/labor")

    large_users.to_sql("users", engine, if_exists="replace", index=False)
    large_transactions.to_sql("transactions", engine, if_exists="replace", index=False)

    print(f"\nBetöltve az adatbázisba:")
    print(f"  users: {len(large_users)} sor")
    print(f"  transactions: {len(large_transactions)} sor")
    ```

10. Ellenőrizd egy gyors lekérdezéssel:

    ```python
    result = pd.read_sql("""
        SELECT u.city, COUNT(*) as trx_count, ROUND(AVG(t.amount)::numeric, 0) as avg_amount
        FROM transactions t
        JOIN users u ON t.user_id = (u.ctid::text::point)[0]::int
        GROUP BY u.city
        ORDER BY trx_count DESC
        LIMIT 10
    """, engine)
    print(f"\nTop 10 város tranzakciók szerint:\n{result}")
    ```

11. Commitold a változtatásokat.

### Beadandó

!!! example "4. feladat beadandó"
    * Commitold a változtatásokat.
    * Készíts egy képernyőképet a generált adathalmaz első néhány soráról és az összesítő statisztikáról, és mentsd el a repository gyökerébe **`f4.png`** néven.

## 5. feladat: Web scraping

Nem minden adat érhető el API-n vagy adatbázisból. Gyakran weboldalakról kell adatokat kinyernünk strukturált formába. Ebben a feladatban a [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/) könyvtárat fogod használni HTML oldalak elemzésére.

!!! warning "Etikus web scraping"
    Mielőtt bármilyen weboldalról adatot gyűjtenél, mindig ellenőrizd:

    * A weboldal **`robots.txt`** fájlját (pl. `https://example.com/robots.txt`), ami meghatározza, mely oldalak scrape-elhetők.
    * A weboldal **Felhasználási feltételeit** (Terms of Service), ami tilthatja az automatikus adatgyűjtést.
    * Tarts be **illedelmesen rövid szüneteket** a kérések között, hogy ne terheld túl a szervert.
    * Mindig azonosítsd magad a `User-Agent` fejléccel.

### Beautiful Soup telepítése

1. Telepítsd a szükséges csomagot:

    ```powershell
    PS> poetry add beautifulsoup4
    ```

2. Hozz létre egy `scraper.py` fájlt a projekt gyökerében:

    ```python
    def main():
        # Ide kerül a logika

    if __name__ == '__main__':
        main()
    ```

### HTML oldal letöltése és elemzése

3. Ebben a példában a [Books to Scrape](https://books.toscrape.com/) weboldalt fogjuk használni, ami kifejezetten web scraping gyakorláshoz készült.

    ```python
    import requests
    from bs4 import BeautifulSoup

    url = "https://books.toscrape.com/"
    headers = {"User-Agent": "DataEngLab/1.0 (university project)"}

    response = requests.get(url, headers=headers, timeout=10)
    response.raise_for_status()

    soup = BeautifulSoup(response.text, "html.parser")
    print(f"Oldal címe: {soup.title.string}")
    ```

    !!! note "HTML parser"
        A `BeautifulSoup` konstruktor második paramétere a használni kívánt parser. A `"html.parser"` a Python beépített parsere, ami a legtöbb esetben elegendő. Komplexebb oldalakhoz érdemes a `"lxml"` parsert használni (külön telepítés szükséges).

### Adatok kinyerése

4. Keresd meg az oldalon található könyveket, és nyerd ki a nevüket, árukat és értékelésüket:

    ```python
    books = []
    articles = soup.find_all("article", class_="product_pod")

    for article in articles:
        title = article.h3.a["title"]
        price = article.find("p", class_="price_color").text
        rating_class = article.find("p", class_="star-rating")["class"][1]
        books.append({
            "title": title,
            "price": price,
            "rating": rating_class,
        })

    print(f"Talált könyvek száma: {len(books)}")
    for book in books[:5]:
        print(f"  {book['title'][:40]:<40} | {book['price']} | {book['rating']}")
    ```

5. Tisztítsd meg az áradatot, hogy numerikus értékké alakítsd:

    ```python
    import pandas as pd

    df = pd.DataFrame(books)
    df["price_numeric"] = df["price"].str.replace("£", "").astype(float)

    rating_map = {"One": 1, "Two": 2, "Three": 3, "Four": 4, "Five": 5}
    df["rating_numeric"] = df["rating"].map(rating_map)

    print(f"\nTisztított adatok:\n{df.head()}")
    ```

### Több oldal feldolgozása

6. A weboldal lapozott. Gyűjtsd össze az első 5 oldal könyveinek adatait:

    ```python
    all_books = []

    for page in range(1, 6):
        if page == 1:
            page_url = "https://books.toscrape.com/"
        else:
            page_url = f"https://books.toscrape.com/catalogue/page-{page}.html"

        response = requests.get(page_url, headers=headers, timeout=10)
        response.raise_for_status()
        soup = BeautifulSoup(response.text, "html.parser")

        articles = soup.find_all("article", class_="product_pod")
        for article in articles:
            title = article.h3.a["title"]
            price = article.find("p", class_="price_color").text
            rating_class = article.find("p", class_="star-rating")["class"][1]
            all_books.append({
                "title": title,
                "price": price,
                "rating": rating_class,
            })

        print(f"Oldal {page}: {len(articles)} könyv feldolgozva.")
    ```

    !!! tip "Lapozás kezelése"
        Lapozott weboldalaknál figyeld meg az URL struktúrát. A legtöbb esetben az oldalszám az URL-ben jelenik meg (pl. `page-2.html`, `?page=2`). Mindig ellenőrizd, hogy van-e következő oldal, mielőtt továbblépnél. A `soup.find("li", class_="next")` segítségével ellenőrizheted, hogy létezik-e következő oldal.

### Adatok tisztítása és mentése

7. Tisztítsd és mentsd el az összegyűjtött adatokat:

    ```python
    full_df = pd.DataFrame(all_books)
    full_df["price_numeric"] = full_df["price"].str.replace("£", "").astype(float)

    rating_map = {"One": 1, "Two": 2, "Three": 3, "Four": 4, "Five": 5}
    full_df["rating_numeric"] = full_df["rating"].map(rating_map)

    print(f"\nÖsszegyűjtött könyvek: {len(full_df)}")
    print(f"\nStatisztikák:")
    print(f"  Átlagos ár: £{full_df['price_numeric'].mean():.2f}")
    print(f"  Átlagos értékelés: {full_df['rating_numeric'].mean():.1f} / 5")
    print(f"  Legdrágább: {full_df.loc[full_df['price_numeric'].idxmax(), 'title']}")

    full_df.to_csv("data/books.csv", index=False)
    full_df.to_parquet("data/books.parquet", index=False)
    print("\nAdatok mentve: data/books.csv, data/books.parquet")
    ```

8. Commitold a változtatásokat.

### Beadandó

!!! example "5. feladat beadandó"
    * Commitold a változtatásokat.
    * Készíts egy képernyőképet a webről kinyert és DataFrame-be rendezett adatokról, és mentsd el a repository gyökerébe **`f5.png`** néven.
