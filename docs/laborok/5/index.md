# Labor 06 – Smart City IoT Cloud Pipeline: boto3 + Terraform IaC

## Célok

A labor végére képes leszel:

- AWS Learner Lab környezetben dolgozni (credentials, IAM, LabRole)
- **boto3** segítségével programozottan létrehozni és vezérelni AWS erőforrásokat
- S3 Data Lake zónákat kialakítani Hive-partícionált struktúrával
- Szimulált IoT szenzor adatokat generálni és feltölteni AWS-re
- Glue Crawler-t és ETL job-ot indítani boto3 API-n keresztül
- Athena SQL lekérdezéseket futtatni boto3-mal a feldolgozott adatokon
- Valós idejű szenzor eseményeket küldeni Kinesis Data Streams-be
- Lambda függvényt deployolni és Kinesis triggerrel összekötni boto3-ból
- **Terraform** IaC-cal ugyanezt az infrastruktúrát kóddal leírni
- Terraform workspace-eket és modul-struktúrát alkalmazni

---

## Előfeltételek

- AWS Academy Learner Lab hozzáférés
- Docker Desktop telepítve és futó
- Git telepítve
- A labor segéd fájljai: `week07/labor/scripts/` és `week07/labor/docker/`

!!! note "Terraform – nem szükséges lokális telepítés"
    A Terraform a `hashicorp/terraform:1.9` Docker image-ből fut. Minden Terraform parancsot `docker compose run --rm terraform ...` formában adj ki a `week07/labor/docker/` könyvtárból.

---

## AWS Academy Learner Lab – gyorsútmutató

### 1. Bejelentkezés és session indítása

1. Nyisd meg az [AWS Academy](https://www.awsacademy.com/vforcesite/LMS/User/Login) oldalt, és lépj be a kurzushoz tartozó fiókkal.
2. A bal oldali menüben válaszd ki a kurzust → **Modules** → **Learner Lab**.
3. Kattints a **Start Lab** gombra (bal felső sarok). A gomb mellett egy kör jelenik meg:
   - 🔴 Piros → a lab indul, várj (30–60 mp)
   - 🟡 Sárga → folyamatban
   - 🟢 Zöld → a session aktív, használható

!!! warning "Session TTL ~4 óra"
    A zöld jelző megjelenésétől számítva kb. 4 óra áll rendelkezésre. Ha lejár, az erőforrások megmaradnak, de a credentials érvénytelen lesz — új sessiont kell indítani és a credentials-t frissíteni. **A labor végén mindig futtasd le a cleanup/destroy lépéseket**, hogy ne használj fel felesleges kreditet.

### 2. AWS Console megnyitása

- A zöld jelző melletti **AWS** feliratra kattintva nyílik meg az AWS Management Console (automatikus bejelentkezéssel, us-east-1 régióban).
- A Console-t a feladatok ellenőrzéséhez és screenshotoknál használd.

---

## Fontos: Learner Lab korlátok

!!! warning "Olvasd el figyelmesen!"
    Az AWS Academy Learner Lab valódi AWS-t használ, de szigorú korlátokkal:

| Korlát | Érték | Következmény |
|--------|-------|-------------|
| **Session TTL** | ~4 óra | `terraform destroy` kötelező a labor végén! |
| **IAM** | Csak LabRole | Custom IAM role NEM hozható létre |
| **Glue** | Max 1 concurrent job | F3 crawler és F4 ETL job NE fusson egyszerre |
| **Lambda** | Max 10 concurrent | BatchSize=25 elegendő |
| **Kinesis** | Max 10 shard/stream | shard_count=1 maradjon! |
| **Régió** | us-east-1 ONLY | Minden boto3 kliensbe: `region_name='us-east-1'` |
| **S3 backend** | Nem elérhető | `backend "s3"` blokk TILOS Terraform-ban |
| **Bucket nevek** | Globálisan egyediek | Neptun-kód prefix kötelező |
| **Credentials TTL** | Session-kor lejár | Új session esetén `~/.aws/credentials` újra kell másolni |

!!! danger "Terraform Destroy"
    A labor VÉGÉN minden esetben futtasd (mindkét workspace-ben!):  
    `docker compose run --rm terraform destroy`  
    Ha elmulasztod és a session lejár, az erőforrások tovább futnak → kredit elfogyhat.

---

## Architektúra áttekintése

```
Smart City IoT Platform
        │
        ├─ Batch Pipeline (boto3 → Terraform)
        │       │
        │   Szenzor CSV (Faker, seed=42)
        │       │
        │   S3 raw/sensors/sensor_type=*/year=*/month=*/
        │       │
        │   Glue Crawler ──→ Glue Data Catalog
        │       │
        │   Glue ETL Job (PySpark, bronze_to_silver.py)
        │       │
        │   S3 processed/sensors/ (Parquet, Snappy)
        │       │
        │   Athena SQL (PM2.5, zajszint, forgalom)
        │
        └─ Streaming Pipeline (boto3 → Terraform)
                │
            Python Producer (streaming_producer.py)
                │
            Kinesis Data Streams (1 shard)
                │
                ├─ Lambda Processor (validálás, severity)
                │       │
                │   CloudWatch Logs
                │
                └─ Kinesis Firehose
                        │
                    S3 streaming/sensors/ (GZIP, 60s buffer)
```

**IoT szenzor domain:** 5 Budapest kerület × 4 szenzortípus × több metrika

| Szenzortípus | device_id prefix | Metrikák |
|---|---|---|
| Légminőség | AQ-001…AQ-005 | PM2.5, PM10, NO2 (µg/m³) |
| Zajszint | NS-001…NS-005 | noise_db (dB) |
| Forgalom | TR-001…TR-005 | vehicle_count (jármű/h), avg_speed (km/h) |
| Kombinált | CM-001…CM-005 | PM2.5, noise_db, vehicle_count |

---

## Környezet indítása

### 1. AWS credentials másolása

Learner Lab Console-ban: **AWS Details** → **AWS CLI** → másold a credentials-t:

```bash
# Windows PowerShell
mkdir -p $env:USERPROFILE\.aws
# Másold be a Learner Lab által adott credentials-t:
notepad $env:USERPROFILE\.aws\credentials
```

A fájl tartalma (Learner Lab adja):
```ini
[default]
aws_access_key_id     = ASIA...
aws_secret_access_key = ...
aws_session_token     = ...
```

### 2. Konténerek indítása

```bash
cd week07/labor/docker
docker compose up -d jupyter
```

Böngészőben: **http://localhost:8888** → token: `labor06`


### 3. Csomagok telepítése (első Jupyter cella)

```python
!pip install -q boto3 faker pandas pyarrow
```

Ellenőrzés:
```python
import boto3
print(boto3.__version__)
```

---