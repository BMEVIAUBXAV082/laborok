# Opcionális Házi Feladat

## Általános információk

**Specifikáció határidő**: 2026.04.12. 23:59   
**Beadási határidő**: 2026.05.24. 23:59  
**Beadás módja**: GitHub repository linkje a tárgy Moodle-felületén  

---

## Feladat leírása

A házi feladat célja egy **teljes, end-to-end data engineering pipeline** megtervezése és megvalósítása, amely lefedi a data engineering életciklus összes fő elemét: adatgenerálást, betöltést, tárolást, transzformációt, ütemezést és kiszolgálást.

A hallgatónak egy **valós vagy szimulált adatforrásból** kiindulva kell egy működő pipeline-t felépítenie, amelynek végén az adat elemzésre kész formában áll rendelkezésre. A feladat megoldható lokálisan (Docker-alapú környezetben) vagy felhőszolgáltatóra (AWS, GCP, Azure) telepítve.

---

## A pipeline kötelező elemei

### 1. Adatforrás és adatbetöltés
- Legalább **két különböző adatforrás** használata (pl. REST API, CSV/JSON fájl, relációs adatbázis, szimulált stream)
- Az adatbetöltés legyen **batch** és/vagy **streaming** alapú
- A forrásadatok legalább egy esetben tartalmazzanak strukturálatlan vagy szemisturkturált adatot (pl. JSON, Parquet)

### 2. Adattárolás
- Nyers (raw) adatok tárolása egy **landing zone**-ban (pl. S3-kompatibilis objektumtároló, lokális fájlrendszer)
- Transzformált adatok tárolása egy **adattárházban vagy data lakehouse** megoldásban (pl. DuckDB, PostgreSQL, Snowflake, BigQuery, Delta Lake)
- Az adatmodell legalább egy **csillag sémát** tartalmazzon (ténytábla + legalább két dimenziótábla)

### 3. Transzformáció
- Legalább egy **batch transzformációs lépés** megvalósítása (pl. Pandas, Spark, dbt, SQL)
- A transzformáció során legyen **adattisztítás** (null kezelés, típuskonverzió) és legalább egy **aggregáció**
- Opcionálisan: streaming transzformáció megvalósítása (pl. Kafka + Flink vagy Spark Structured Streaming)

### 4. Orchestration és ütemezés
- A pipeline lépéseit egy **orkesztrációs eszköz** vezérelje (pl. Apache Airflow, Dagster, Mage, Prefect)
- A DAG tartalmazza a függőségeket, és legyen képes **újrafuttatásra (idempotens)**
- Legyen legalább egy **ütemezett futtatás** (pl. napi batch)

### 5. Infrastruktúra
- A teljes környezet legyen **reprodukálható**: Docker Compose, vagy Infrastructure as a Code eszköz (pl. Terraform) segítségével
- Szükséges egy `README.md`, amely tartalmazza a futtatáshoz szükséges összes lépést

### 6. Adatkiszolgálás
- Az elkészült adatmodell alapján legyen lekérdezhető az adat: pl. SQL nézetekkel, egy egyszerű BI dashboarddal (pl. Metabase, Superset, Grafana), vagy egy REST API végponton keresztül
- Legalább **3 értelmes analitikai lekérdezés** (pl. SQL) dokumentálva

---

## Témaötletek

Az alábbi témák egyike választható, de a saját ötlet preferált:

- **E-kereskedelmi rendelések pipeline-ja**: webshop rendelési és termékadat-stream feldolgozása, valós idejű értékesítési dashboard
- **Időjárás-adatok analitikája**: publikus meteorológiai API-ból betöltött adatok historikus elemzése, anomáliák detektálása
- **Pénzügyi tranzakciók**: szimulált banki tranzakciók batch feldolgozása, fraud-detection aggregációk
- **Közösségi média aktivitás**: Twitter/Reddit API (vagy szimulált adat), sentiment-aggregáció időablakokban
- **IoT szenzoradatok**: szimulált eszközadatok streaming pipeline-on keresztüli feldolgozása és vizualizációja

---

## Pontozás

| Elem | Pontszám |
|---|---|
| Adatforrás és betöltés (min. 2 forrás, dokumentált) | 3 pont |
| Adattárolás (landing zone + adatmodell, csillag séma) | 3 pont |
| Transzformáció (tisztítás, aggregáció, batch) | 3 pont |
| Orchestration (DAG, ütemezés, idempotencia) | 3 pont |
| Infrastruktúra (Docker / IaC, reprodukálhatóság) | 3 pont |
| Adatkiszolgálás (lekérdezések / dashboard / API) | 3 pont |
| Kódminőség, dokumentáció, README | 2 pont |
| **Összesen** | **20 pont** |


---

## További értékelési szempontok

- A pipeline **valóban fusson**: a README alapján, friss környezetben elindítható és lefut hiba nélkül
- Az egyes lépések **egymástól elkülöníthetők** legyenek (pl. külön Airflow task-ok vagy Dagster asset-ek)
- A kód legyen **olvasható és kommentezett**; kerülni kell a hardcoded értékeket (használj `.env` fájlt vagy konfigurációs réteget)
- Az adatmodell **tervezési döntései** legyenek indokolva a dokumentációban (min. fél oldal architekturális leírás)

---

## Beadandók

1. **GitHub repository** (publikus vagy oktatóval megosztott privát) az összes forráskóddal
2. **`README.md`** a telepítési és futtatási utasításokkal, architektúra-ábrával
3. **Rövid technikai dokumentáció** (2-3 oldal), amely leírja:
    - a választott architektúrát és annak indoklását
    - az adatmodellt (ER vagy csillag séma ábra)
    - a pipeline futásának eredményét (pl. screenshot vagy lekérdezési eredmény)

