# MLOps Labor

A labor során önállóan fogsz elkészíteni egy egyszerű osztályozó modellt, illetve a hozzá tartozó MLOps pipeline-t. A labor célja, hogy tapasztalatot szerezz az MLOps gyakorlati alkalmazásában.

## Előkészület

A feladatok megoldásához az alábbi telepített szoftverekre van szükség (alternatívaként használhatjuk a [BME Cloud](https://cloud.bme.hu/) egyik virtuális gépét):

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) vagy egyéb Docker-konténer futtatására alkalmas környezet
* [Python](https://www.python.org/) futtatókörnyezet (3.12-es verzió)
* [VS Code](https://code.visualstudio.com/) vagy egyéb kódszerkesztő
* [Git](https://git-scm.com/)
* [Poetry](https://python-poetry.org/) vagy egyéb függőségkezelő (opcionális, de erősen javasolt)

!!! warning "Technológiák"
    A labor során több (számodra akár teljesen új) technológiával is kell dolgoznod. Ez nem öncélú bonyolítás, az MLOps világában a fejlesztők általában kész, összetett céges infrastruktúrával dolgoznak, amelyet külön IT csapatok tartanak karban. Mivel ilyen környezetet most nem tudunk készen adni, kisebb léptékben szimuláljuk. Emiatt neked kell néhány komponenst telepíteni és összekapcsolni. Előfordulhat, hogy bizonyos lépések értelmét csak később látod meg, de így könnyebben megérted majd, hogyan működik egy valós MLOps rendszer, és mire számíthatsz a munka világában.

A feladatok megoldása során ne felejtsd el követni a feladatbeadás folyamatát, amiről [itt olvashatsz részletesen](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keresd meg a laborhoz tartozó meghívó URL-jét, és annak segítségével hozd létre a saját repository-dat.
2. Várd meg, míg elkészül a repository, majd checkout-old ki.
3. Hozz létre egy új ágat `megoldas` néven, és ezen az ágon dolgozz.
4. A neptun.txt fájlba írd bele a Neptun-kódodat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun-kód 6 karaktere.

### Virtuális környezet létrehozása

Már most érdemes létrehozni egy virtuális környezetet a projekt gyökerében, és azt aktiválni. A virtuális környezet célja, hogy a Python csomagokat, amiket használni fogsz, ne globálisan kelljen feltelepíteni, hanem csak az adott projektre vonatkozóan. Így nem fog problémát okozni az, hogy ebben a projektben az egyik csomagból az egyik verziót használod, míg egy másikban ugyanabból a csomagból egy másikat.

!!! tip "Tipp"
    A virtuális környezet létrehozásához az alábbi leírás a `poetry`-t javasolja. Ettől saját felelősségre el lehet térni, de erősen    ajánlott ennek használata. Ha elakadnátok, bátran kérjetek segítséget.

1. Nyiss meg egy parancssort a projekt gyökerében, és add ki a `poetry install` parancsot. Ez létrehozza az előre konfigurált virtuális környezetet a `.venv` mappában.

2. A környezet aktiválásához a `.\.venv\Scripts\activate` parancsot kell futtatni.

    !!! note "Megjegyzés"
        A parancssoros környezettől függően ez eltérő lehet, tájékozódjunk az opciókról a [dokumentációban](https://python-poetry.org/docs/managing-environments/#activating-the-environment). Az útmutató példái Windows-on PowerShell használatára építenek, ezeket az utasításokat mindig a projekt gyökerében kell kiadni. Ha a `PS>` előtagot látjátok, az a parancssorra utal, ezt nem kell beírni.

3. Sikeres aktiváció után a parancssorban megjelenik a virtuális környezet neve a sor elején (pl.: `(lab03-mlops-py3.12) PS ... > `)

4. Ezután minden parancsot ebben a parancssorban adjunk ki, különben egyes komponenseket lehet, hogy nem fog megtalálni a rendszer.

### Az elkészítendő pipeline

A labor során egy egyszerű osztályozó modellt kell készítened. Az egyszerűség kedvéért készítsünk egy Iris osztályozót. A modell bemenetként 4 db számot fog megkapni (a szirom- és csészelevél szélessége és hosszúsága), és a modellnek meg kell mondania, hogy a 3 osztály közül (Iris-setosa, Iris-virginica és Iris-versicolor) melyikhez tartozik.

!!! tip "Tipp"
    A feladat egy tipikus példafeladat, amiben könnyen lehet jó eredményeket elérni. Egy hasonló példát lehet találni a [demó kódok](https://github.com/VIAUBXAV081/demok/) között is.

A modell elkészítésén felül az alábbi témakörökkel kell megbirkózni:

* Verziókezelés
* Üzembe helyezés
* Monitorozás
* Automatizálás

## Kiinduló projekt felépítése

A kiinduló projekt számos fájlt tartalmaz, így fontos, hogy átlássuk, mit hol kell keresni. Itt egy rövid összefoglaló, hogy az egyes mappák és fájlok milyen célt szolgálnak.

* `.github` mappa: A labor beadásához és ellenőrzéséhez szükséges konfigurációs fájlok.
* `api` mappa: Ide kerül az API megvalósításához szükséges kód.
* `data` és `models` mappa: Adatokat és modelleket tartalmazó mappa, ezek tartalmát többnyire nem commitoljuk.
* `docker-compose.yaml`: A Docker-konténerek egyszerű indításához és összekapcsolásához szükséges leíró fájl.
* `main.py`: A projekt elsődleges belépési pontja.
* `poetry.lock`, `poetry.toml`, `pyproject.toml`: A projekt és csomagkezelő konfigurációs fájljai, itt találjuk majd a függőségeket. Kézzel nem kell őket módosítani.
* `prometheus.yml`: A monitorozó eszköz konfigurációs fájlja.

A virtuális környezet létrehozása után találkozni fogunk még a `.poetry` és a `.venv` mappákkal is:

* `.poetry` mappa: A Poetry belső fájljait tartalmazza.
* `.venv` mappa: A Python virtuális környezet, amely a telepített csomagokat tartalmazza.

## 1. feladat: Adatok előfeldolgozása és verziózása

### DVC inicializálása

Szoftverek fejlesztése során a forráskód verziókezelésére fektetjük általában a legnagyobb hangsúlyt. MI-alkalmazások fejlesztése során azonban az adat is hasonló fontossággal bír, melynek verziókezelése nagyobb kihívással jár. Ennek oka elsősorban az, hogy a hagyományos verziókezelők kis méretű szöveges fájlokra optimalizáltak. Az adatok verziókezelésére egy remek eszköz a [DVC](https://dvc.org/doc/start), ami a gittel összehangolva működik. Elve, hogy az adatfájlokra mutató leíró fájlokat hoz létre a repository-ban, melyeket hagyományosan tudunk verziózni. A leíró fájlok alapján a DVC azonosítani tudja az adat megfelelő verzióját, és azt egy külön tárból tudja betölteni. A DVC előnye, hogy a hasonló Git LFS-hez képest rugalmasabban konfigurálható, hogy milyen tárat választunk az adatoknak.

1. Telepítsd az eszközt a `poetry add dvc` paranccsal.
2. Inicializáld a projektet a `dvc init` parancs segítségével. Ez létrehoz egy `.dvc` mappát, amit majd fontos lesz commitolni is, hisz ez tartalmazza majd az adatfájlok tényleges elérési útvonalát.
3. A `dvc remote add -d myremote C:\Users....` parancs segítségével állíts be egy tetszőleges lokális mappát az adatok tárának. Olyan mappát válasszunk, ami a projekt mappáján kívül helyezkedik el. A parancsban a `myremote` név tetszőlegesen változtatható.

!!! note "Megjegyzés"
    A valóságban adattárnak javasolt valamilyen távoli szerver útvonalat, vagy Google Drive tárhelyet megadni, de ezek konfigurálása komplexebb, ezért használjuk a lokális mappát ebben a példában.

### Adatok beszerzése

A feladat első lépéseként szerezd be az adatkészletet, amin tanítani fogod a modellt. Az Iris adatkészlet elérhető az alábbi linken: [https://archive.ics.uci.edu/dataset/53/iris](https://archive.ics.uci.edu/dataset/53/iris)

1. Töltsd le az adatkészletet és csomagold ki. Az archívum több fájlt is tartalmaz, ezek közül csak néhányra lesz szükséged.

2. Másold az `iris.data` fájlt a projekt `data` könyvtárába `data.csv` néven.

3. Javítsd ki az adatkészlet 2 hibás sorát. Ezekről az adatkészlet `iris.names` fájljában tudsz részletesen olvasni.

    1. A 35. mintánál a 4. tulajdonság hibás: `4.9,3.1,1.5,0.2,"Iris-setosa"`,
    2. A 38. mintánál pedig a 2. és 3. tulajdonság a hibás: `4.9,3.6,1.4,0.1,"Iris-setosa"`

4. Add hozzá a fájlt a verziókezelt adatfájlokhoz:

    1. A `dvc add data/data.csv`-vel tudod az adott fájlra beállítani a verziókövetést. Ez fogja létrehozni a `data.csv.dvc` leíró fájlt.
    2. A `dvc push` paranccsal tudod az adatot feltölteni a korábban beállított adattárba a githez hasonlóan.

        !!! note "Megjegyzés"
            A `dvc push` ellentéte a `dvc pull`. Ezt a parancsot akkor használjuk, ha egy meglévő, verziókezeléshez hozzáadott fájlt szeretnénk lokálisan elérni az adott projektben. Például, ha egy projektet egy másik gépen szeretnél folytatni, akkor miután a git repository-t letöltötted, a `dvc pull` parancs a leíró fájlok alapján letölti az adatokat is. Természetesen ennek feltétele, hogy a `remote` egy olyan elérési út legyen, ami az adott projektből látható (a lokális mappa másik számítógépen nem fog működni).

            Ezt akár ki is tudod próbálni, ha törlöd a `data.csv` fájlt (a `data.csv.dvc` fájlt ne!). Ezután a `dvc pull` parancs kiadásával a fájlt vissza tudod állítani.

5. Hozd létre az első commitot a git segítségével.

    !!! warning "Verziókezelt fájlok"
        Figyelj rá, hogy az adatfájlok esetén csak az adatfájlhoz tartozó `*.dvc` kiterjesztésű fájlokat commitoljuk, magát az adatot ne, továbbá, hogy a `.dvc` mappa is verziókezelve legyen.

### Adatok előfeldolgozása

Mielőtt felhasználnád az adatokat, szükség van előfeldolgozásra. A DVC egy nagyon hasznos funkciója, a data pipeline-ok létrehozása. Ennek segítségével létre tudunk hozni egy automatikus előfeldolgozó folyamatot. Ennek a folyamatnak meg tudunk adni függőségeket és kimeneteket, és csak akkor fog lefutni, ha a függőségek közül valami módosult, vagy ha a kimenetek közül valamelyik nem létezik. A kimeneti fájlok automatikusan bekerülnek a verziókövetett fájlok közé, ezeket nem kell hozzáadni manuálisan. A pipeline leírása a `dvc.yaml` fájlba kerül, a verziókövetett fájlok pedig a `dvc.lock` fájlba.

1. Telepítsd a `pandas` és `scikit-learn` csomagokat a `poetry add pandas scikit-learn` paranccsal.

2. Hozz létre egy `preprocess.py` fájlt a projekt gyökerébe, ebbe a fájlba készül el az előfeldolgozó script:

    1. Érdemes a scriptet egy `main` utasításba csomagolni, amit a fájl végén található elágazásban hívunk meg. Így biztosítható, hogy a kód csak akkor fusson le automatikusan, ha közvetlenül indítjuk a fájlt, és ne akkor, amikor más modul importálja azt. Ez átláthatóbbá és újrahasznosíthatóbbá teszi a programot.

        ```python
        def main():
            # Ide kerül az előfeldolgozó logika

        if __name__ == '__main__':
            main()
        ```

    2. Töltsd be az adatkészletet `pandas` segítségével, és bontsd szét oszlopok szerint bemenő és kimenő tulajdonságokra.

        ```python
        import pandas as pd

        data_path = "data/data.csv"
        df = pd.read_csv(data_path, header=None)
        X = df.iloc[:, 0:4]
        y = df.iloc[:, 4]
        ```

        !!! warning "Kód másolás"
            Az **import** utasítások mindig a fájl elejére kerülnek, a kódrészleteket pedig a `main()` függvényen belülre rakjuk egymás után.

    3. Cseréld ki az osztályneveket indexekre 0–2 között, ehhez használhatod a `scikit-learn` csomag [`LabelEncoder`](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.LabelEncoder.html) osztályát.

        ```python
        from sklearn.preprocessing import LabelEncoder

        label_encoder = LabelEncoder()
        y_encoded = pd.DataFrame(label_encoder.fit_transform(y), columns=[y.name])
        ```

    4. Válaszd szét az adatkészletet véletlenszerűen tanító- és tesztadatokra 80:20 arányban.

        ```python
        from sklearn.model_selection import train_test_split

        test_size = 0.2
        X_train, X_test, y_train, y_test = train_test_split(X, y_encoded, test_size=test_size)
        ```

    5. Mentsd ki az adatkészletet `train.csv` és `test.csv` néven a `data` mappába.

        ```python
        train_path = "data/train.csv"
        test_path = "data/test.csv"
        pd.concat((X_train, y_train), axis=1).to_csv(train_path, index=False, header=False)
        pd.concat((X_test, y_test), axis=1).to_csv(test_path, index=False, header=False)
        ```

    6. Mentsd ki az osztályneveket `classes.json` néven a `data` mappába, hogy az indexek később visszaállíthatók legyenek.

        ```python
        class_path = "data/classes.json"
        class_names = label_encoder.classes_.tolist()
        with open(class_path, 'w') as f:
            json.dump(class_names, f, indent=4)
        ```

3. Add hozzá a `preprocess.py` fájlt a DVC data pipeline-hoz:

    * A `dvc stage add` paranccsal lehet hozzáadni lépéseket,
    * A `-n` kapcsolóval a lépés nevét,
    * A `-d` kapcsolóval a függőségeket,
    * A `-o` kapcsolóval pedig a kimeneteket adjuk meg,
    * A további argumentumok a lépés futtatásának módját írják le.

    ```powershell
    PS> dvc stage add -n preprocess `
                -d preprocess.py `
                -d data/data.csv `
                -o data/train.csv `
                -o data/test.csv `
                -o data/classes.json `
                python preprocess.py
    ```

4. Futtasd a pipeline-t a `dvc repro` paranccsal, és ellenőrizd a létrejövő fájlok tartalmát.

5. Commitold a változásokat git segítségével.

    !!! warning "Megjegyzés"
        Ne felejtsd ki a `dvc.yaml` és `dvc.lock` fájlokat sem.

### Beadandó

!!! example "1. feladat beadandó"
    * Commitold a változtatásokat.
    * Készíts egy képernyőképet a verziókövetett fájlok listájáról (`dvc ls -R --dvc-only .`) és mentsd el a repository gyökerébe **`f1.png`** néven.

## 2. feladat: Osztályozó modell elkészítése

### Modell készítés

Készítsünk egy egyszerű osztályozó modellt, felhasználva az előző lépésben készített adathalmazt. Ebben a példában a feladat egyszerűsége miatt a [`scikit-learn`](https://scikit-learn.org/stable/) csomagot fogjuk használni.

!!! tip "Tipp"
    Az előadás demók között találhatsz egy nagyon [hasonló példát](https://github.com/VIAUBXAV081/demok/tree/main/CustomAIServiceIntegration/1_SimpleModel), ami a **TensorFlow** használatát mutatja be.

1. A modell létrehozásához szükséges kódot a `main.py` fájlban írd meg.

2. Tanítsd be a modellt az előkészített tanító adathalmaz segítségével.

    1. Töltsd be a tanító adatokat a korábbi feladathoz hasonlóan. Töltsd be az osztályneveket is, amit később a vizualizációhoz fogunk használni.

        ```python
        import pandas as pd
        import json

        train_path = 'data/train.csv'
        df = pd.read_csv(train_path, header=None)
        X = df.iloc[:, 0:4]
        y = df.iloc[:, 4]

        class_path = 'data/classes.json'
        with open(class_path, 'r') as f:
            class_names = json.load(f)
        ```

    2. Válaszd szét az adatokat tanító és validáló adatokra 75:25 arányban. A validáló adatokkal fogjuk kalibrálni a tanítási folyamatot.

        !!! tip "Miért 75:25 arányban osztunk?"
            Korábban a tanító és teszt adatokat 80:20 arányban osztottuk. Ha ezután a tanító adatokat tovább osztjuk 75:25-re, akkor a végeredmény, hogy a teljes adathalmazra nézve az adatok 60:20:20 arányban vannak felosztva tanító, validáló és teszt adatokra.

        ```python
        from sklearn.model_selection import train_test_split

        val_size = 0.25
        X_train, X_val, y_train, y_val = train_test_split(X, y, test_size=val_size)
        ```

    3. Hozd létre a modellt, legyen 3 rejtett rétege 10–10 neuronnal, és 100 epochra állítsd a tanítást.

        ```python
        from sklearn.neural_network import MLPClassifier

        model = MLPClassifier(
            hidden_layer_sizes=(10, 10, 10),
            max_iter=100,
        )
        ```

    4. Tanítsd be a modellt a tanító adatokon.

        ```python
        model.fit(X_train, y_train)
        ```

    5. Értékeld ki a modell pontosságát a validáló adatokon.

        1. A modell segítségével hozz létre predikciót a validáló adatokon.

            ```python
            y_pred = model.predict(X_val)
            ```

        2. Számold ki a pontosságot.

            ```python
            from sklearn.metrics import accuracy_score

            accuracy = accuracy_score(y_val, y_pred)
            print(f"Pontosság: {accuracy*100:.2f}%")
            ```

        3. Készíts az eredmény alapján konfúziós mátrixot, és jelenítsd meg azt. A megjelenítéshez a [`matplotlib`](https://matplotlib.org/) csomagot tudod használni.

            ```powershell
            PS> poetry add matplotlib
            ```

            ```python
            from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
            from matplotlib import pyplot as plt

            cm = confusion_matrix(y_val, y_pred)
            disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=class_names)
            disp.plot()
            plt.show()
            ```

3. Futtasd a modellt és elemezd az eredményeket.

    ```powershell
    PS> python main.py
    ```

    !!! warning "Változó eredmények"
        Minden egyes futtatáskor eltérő eredményeket kapunk. Ennek oka, hogy a tanítási folyamat során több véletlenszám-generátort is használunk a felszín alatt. Egyszer az adatok felosztásakor, egyszer pedig a tanítás során. Ha minden egyes futtatáskor konzekvens eredményeket szeretnénk kapni, a véletlenszám-generátoroknak át kell adni egy `seed` paramétert. Ha minden futtatáskor ugyanazt az értéket kapja, akkor mindig ugyanazokat a véletlen számokat fogja generálni, így a futtatások között nem lesz különbség. Jelen példában ezt a `random_state=<int>` paraméterrel tudjuk beállítani a `train_test_split` függvényen és az `MLPClassifier` osztályban. A paraméternek tetszőleges értéket választhatunk, csak legyen ugyanaz.

        ```python
        seed = 42

        # ...
        ... = train_test_split(..., random_state=seed)

        # ...
        ... = MLPClassifier(..., random_state=seed)
        ``` 

4. Commitold a változtatásokat git segítségével.

### Modell optimalizálása

Az előző feladatban kapott modell láthatóan még nem működik túl jól. Hozzávetőlegesen 60% körüli pontosságot kapunk. Ennek oka lehet a feladathoz nem megfelelő architektúra vagy kevés tanítás. Próbáljuk meg optimalizálni a modellt a feladathoz, ehhez valósítsunk meg hiperparaméter-optimalizálást. Hiperparaméter-optimalizálás során a legegyszerűbb eljárás a kimerítő keresés. Ilyenkor definiálunk egy sor paramétert, amelyek kombinációin végigmenve tanítunk egy modellt, és kiértékeljük azt a validációs adatokon. A végén azt a modellt választjuk, amelyik a legjobb eredményt adja.

1. Valósíts meg egy egyszerű hiperparaméter-optimalizálást, ehhez módosítsd a korábban elkészült kódot. Ne felejts el commitolni a feladat megkezdése előtt, hogy a megoldásodban látható legyen a folytonosság.

    1. Definiálj paramétereket, és ezeknek értékkészletet, amelyek segítségével elvégzed az optimalizálást. Paraméternek válaszd a rejtett rétegek számát és az epochok számát. A rejtett rétegek száma legyen 2–4 között, az epochok száma pedig 100–1000 között.

        ```python
        parameters = {
            'hidden_layers': [2, 3, 4],
            'epochs': [100, 250, 500, 750, 1000]
        }
        ```

    2. Iterálj végig az értékek kombinációin, és állíts össze egy modellt minden kombinációval. A kombinációk előállításához használd a `ParameterGrid` segédosztályt.

        ```python
        from sklearn.model_selection import ParameterGrid
        import numpy as np

        for params in ParameterGrid(parameters):

            model = MLPClassifier(
                hidden_layer_sizes=np.repeat(10, params['hidden_layers']),
                max_iter=params['epochs'],
            )
        ```

    3. Tanítsd be a modelleket, majd számold ki minden egyes modell veszteségét a validációs adatokon.

        ```python
        from sklearn.metrics import log_loss

        model.fit(X_train, y_train)

        y_prob = model.predict_proba(X_val)
        loss = log_loss(y_val, y_prob)
        ```

        !!! note "Predikciók és a veszteség"
            Osztályozás során a célunk, hogy egyes mintákat osztályokba soroljunk, viszont az osztályozó modellek ezt nem közvetlenül teszik meg, hanem először valószínűségeket prediktálnak. A végső osztályt az alapján tudjuk meghatározni, hogy melyik osztályhoz tartozik a legnagyobb valószínűség. Ebben a keretrendszerben, ha a konkrét osztályokat szeretnénk megkapni (amelyeknek legnagyobb a valószínűsége), akkor a `.predict(...)`, ha valószínűségeket, akkor pedig a `.predict_proba(...)` metódust használjuk.

            A modell veszteségének kiszámolásához számos különböző veszteségfüggvény elérhető. A legelterjedtebben osztályozási feladatokhoz a *log loss*-t (vagy más néven *cross-entropy loss*-nak nevezünk) használjuk. A veszteség értékek meghatározásához nem az osztályokat, hanem azok valószínűségét használjuk.

    4. Mentsd el az értékeket. Ebben a lépésben érdemes a veszteségértékek mellett a paramétereket és a betanított modelleket is kimenteni.

        ```python
        search_results = []
        for params in ParameterGrid(parameters):
            # ...
            search_results.append((loss, params, model))
        ```

    5. Válaszd ki a legjobban teljesítő modellt (amelyiknek a legkisebb a vesztesége).

        ```python
        min_loss, best_params, best_model = min(search_results, key=lambda x: x[0])
        print(f"A legjobb modell vesztesége {min_loss:.2f}, paraméterei {best_params}")
        ```

2. Értékeld a betanított modell pontosságát a teszt adathalmazon.

    1. Töltsd be a teszt adatokat.

        ```python
        test_path = 'data/test.csv'
        df = pd.read_csv(test_path, header=None)
        X_test = df.iloc[:, 0:4]
        y_test = df.iloc[:, 4]
        ```

    2. Értékeld ki a legjobban teljesítő modellt a korábbiakhoz hasonlóan.

        ```python
        from sklearn.metrics import accuracy_score, confusion_matrix, ConfusionMatrixDisplay
        from matplotlib import pyplot as plt

        y_pred = best_model.predict(X_test)
        accuracy = accuracy_score(y_test, y_pred)
        print(f"Teszt pontosság: {accuracy*100:.2f}%")

        cm = confusion_matrix(y_test, y_pred)
        disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=class_names)
        disp.plot()
        plt.show()
        ```

3. Futtasd a scriptet, és elemezd, mely paraméterekkel rendelkező modell teljesített a legjobban, és milyen eredményt ér el.

4. Commitold a változtatásokat.

### Modell exportálása

A betanított modellek újrahasznosításához érdemes az elkészült modellt kimenteni egy fájlba, aminek visszatöltésével később használni tudjuk. Számos különböző, keretrendszerfüggő és -független formátum elérhető modellek mentésére. Az egyik elterjedten használt formátum az [ONNX](https://onnx.ai/), ami számos különböző platformon használható.

1. Telepítsd a [`skl2onnx`](https://onnx.ai/sklearn-onnx/index.html) csomagot.

    ```powershell
    PS> poetry add skl2onnx
    ```

2. Exportáld a betanított modellt `.onnx` formátumban a `models` mappába. Az `initial_types` paraméter értékének a bemenet típusát és méretét kell megadni.

    ```python
    from skl2onnx import convert_sklearn
    from skl2onnx.common.data_types import FloatTensorType

    best_onnx = convert_sklearn(best_model, initial_types=[('input', FloatTensorType([None, X_train.shape[1]]))])
    model_path = "models/model.onnx"
    with open(model_path, "wb") as f:
        f.write(best_onnx.SerializeToString())
    ```

3. Elemezd az elkészült fájl tartalmát, ehhez keresd fel a [Netron](https://netron.app/) weboldalt.

4. Commitold a változtatásokat.

### Modell verziókezelése

Az exportált modell verzióit az adatokhoz hasonlóan akár a DVC eszközzel is verziózhatnánk, viszont egy specializált eszköz használatával sokkal részletesebb verziózást is meg tudunk valósítani. Ilyen eszköz például az [MLflow](https://mlflow.org/), aminek segítségével követni tudjuk a modellek tanítását és verziózhatjuk őket. Az eszköz alkalmas arra, hogy egy saját webes felületen részletes metrikákat mutasson a tanítási folyamatokról és a modellek pontosságáról, valamint a legjobban teljesítő modelleket egy központi tárba is tudjuk menteni, és így később könnyen visszakereshető lesz.

Az MLflow használatához lokálisan futtathatunk egy MLflow-példányt, vagy kapcsolódhatunk egy központi szerverhez. Ebben a példában egy köztes megoldást választunk: futtatunk egy MLflow szervert egy Docker-konténerben. Ehhez már minden elő van készítve a `docker-compose.yaml` fájlban.

!!! note "Docker konténerek és a Docker Compose"
    A **Docker-konténerek** arra jók, hogy egy alkalmazást minden szükséges függőségével együtt, elkülönített és hordozható környezetben futtassunk. Így a program bárhol ugyanúgy működik, miközben a konténer könnyebb és gyorsabb, mint egy virtuális gép, mert a host operációs rendszer kernelét használja. Több konténer együttes kezelését a **Docker Compose** teszi lehetővé. Egy *YAML* fájlban leírható a teljes rendszer felépítése, majd egyetlen paranccsal (`docker-compose up`) az egész környezet egyszerre indítható el. Windows alatt a **Docker Desktop** alkalmazás telepítése szükséges konténerek futtatásához.

1. Futtasd az MLflow UI-t Docker segítségével.

    1. Indítsd el a Docker Desktop alkalmazást, ez a háttérben fog futni. Első indításkor be kell állítani, hogy Linux-konténerek futtatására alkalmas legyen. Erről [ezen a linken](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/set-up-linux-containers) tudsz részletesen olvasni.
   
    2. Indítsd el a szükséges konténert, a UI-t a [localhost:5000](http://localhost:5000/) elérési úton találod.

        ```powershell
        PS> docker-compose up -d mlflow
        ```

2. Telepítsd az MLflow csomagot a `poetry add mlflow` parancs segítségével, hogy a scriptben is használni tudd.

3. Módosítsd a korábbi scriptet úgy, hogy logolja a kísérleteket.

    1. Először konfiguráld a csomagot a script elején úgy, hogy a megfelelő szerverre logolja az adatokat.

        ```python
        import mlflow

        mlflow.sklearn.autolog(silent=True)
        mlflow.set_tracking_uri('http://localhost:5000')
        mlflow.set_experiment('IrisDev')
        ```

    2. A tanítást burkold be egy futtatásba (`run`), ami itt a különböző kísérleteket jelenti.

        ```python
        from sklearn.model_selection import ParameterGrid

        for params in ParameterGrid(parameters):
            with mlflow.start_run() as run:
                model = ...
                # ...
        ```

    3. A futtatáson belül logolj minden egyes modellt ONNX formátumban is, így később hozzá fogunk férni.

        ```python
        from skl2onnx import convert_sklearn
        from skl2onnx.common.data_types import FloatTensorType

        onnx = convert_sklearn(model, initial_types=[('input', FloatTensorType([None, X_train.shape[1]]))])
        mlflow.onnx.log_model(onnx, name='onnx')
        ```

    4. Mentsd el az egyes futtatások azonosítóját. Ezzel fogunk tudni később hivatkozni a legjobb eredményűre.

        ```python
        search_results.append((..., run.info.run_id))
        ```

    5. Regisztráld (és így verziózd) a legjobban teljesítő modellt `IrisModel` néven. Preferencia szerint egyedi adatokat is el lehet menteni a `tags` paraméterben.

        ```python
        ..., best_run = min(search_results, ...)

        # ...

        tags = {
            'test_accuracy': accuracy,
            'epochs': best_params['epochs'],
            'hidden_layers': best_params['hidden_layers'],
        }
        mlflow.register_model(f'runs:/{best_run}/onnx', 'IrisModel', tags=tags)
        ```

4. Futtasd a scriptet, és elemezd az MLflow UI felületén, hogy milyen adatokat logol a rendszer.

5. Commitold a változtatásokat.

### Beadandó

!!! example "2. feladat beadandó"
    * Commitold a változtatásokat.
    * Készíts egy-egy képernyőképet az MLflow UI `Experiments / IrisDev` és `Models / IrisModel` felületéről, és mentsd el **`f2-1.png`** és **`f2-2.png`** néven a repository gyökerébe.

## 3. feladat: Modell üzembe helyezése és futtatása

A verziózott `ONNX` modellünket felhasználva készítsünk egy új webes szolgáltatást. A szolgáltatásnak lesz egy API-ja, amire ha elküldünk paramétereket (bemenő adatokat), válaszként a legvalószínűbb osztályt fogjuk visszakapni. A szolgáltatást [FastAPI](https://fastapi.tiangolo.com/) segítségével hozzuk létre, ami egy egyszerű Python-os keretrendszer. Ezt az API-t egy Docker konténerbe fogjuk beburkolni, és így fogjuk tudni futtatni, ezzel szimuláljuk az alkalmazás üzembe helyezését. A szükséges modellt közvetlenül az MLflow Model Registryből fogjuk letölteni, így új verzió esetén könnyű lesz frissíteni az alkalmazást.

### API készítése

A projekt gyökerében már találunk egy előre elkészített `api` mappát, ami tartalmaz egy `api.py` fájlt. Ebbe a fájlba készítsük el az API-hoz szükséges kódot.

!!! note "API script felépítése"
    Az `api.py` fájl felépítése kicsit más, mint amit korábban használtunk, annak érdekében, hogy kövessük a FastAPI konvencióit. Összefoglalva, a fájl az alábbi fontos elemeket tartalmazza:

    * `Settings` osztály és `settings` változó: Az alkalmazás beállításait tartalmazza, környezeti változók segítségével lehet a paramétereket módosítani.
    * `state` változó: Az alkalmazás globális állapotát tartalmazza. Az alkalmazás indulásakor az `init` függvényben van lehetőségünk inicializálni.
    * `app` változó: Az alkalmazás interface.
    * `predict()` függvény: Az alkalmazás `/predict` végpontját leíró függvény.
    * `PredictInput` és `PredictOutput` osztályok: A bemenő és a kimenő JSON formátumát leíró objektumok.

1. Telepítsd az API-hoz szükséges `fastapi[all]` és az ONNX futtatáshoz szükséges `onnxruntime` csomagokat.

    ```powershell
    PS> poetry add fastapi[all] onnxruntime
    ```

2. Futtasd az API-t és nézd meg a [http://localhost:8000/docs](http://localhost:8000/docs) oldalt.

    ```powershell
    PS> fastapi dev api/api.py
    ```

    !!! tip "Swagger használata"
        Az API `/docs` elérési útján találunk egy [Swagger](https://swagger.io/) interface-t. Ez az interface az API leírását tartalmazza interaktív módon, ahol ki is lehet próbálni az egyes végpontokat. Ez nagyon hasznos a hibakereséshez:

        1. Válaszd ki a tesztelni kívánt végpontot, és nyisd le.
        2. Menj a **Try it out** gombra, és szerkeszd a kérés törzsében található JSON-t.
        3. Menj az **Execute** gombra, és elemezd a válaszként kapott struktúrát.

3. Inicializáld a predikciók készítéséhez szükséges futtatókörnyezetet.

    1. Vegyél fel két új változót a beállítások közé. Egyet az MLflow, másikat az osztályokat leíró JSON fájl elérési útjának. Állíts be alapértelmezett értéket.

        ```python
        class Settings(BaseSettings):
            class_path: str = "data/classes.json"
            mlflow_uri: str = "http://localhost:5000"
        ```

        !!! note "Beállítások"
            Erre azért lesz szükség, mert fejlesztés közben, valamint majd később üzemeltetés közben ezeknek a paramétereknek akár különböző értékük is lehet.

    2. Írd meg az inicializáló utasításokat az `init()` függvényben. Ez kizárólag egyszer, az alkalmazás indulásakor fog lefutni. Az alkalmazás állapotait a `state` változóba írd.

        1. Állítsd be az MLflow elérési útját és töltsd le a legfrissebb osztályozó modellt.

            ```python
            import mlflow

            mlflow.set_tracking_uri(settings.mlflow_uri)

            model_name = "IrisModel"
            model_version = "latest"
            model_uri = f"models:/{model_name}/{model_version}"
            model = mlflow.onnx.load_model(model_uri)
            ```

            !!! note "MLflow modellek"
                A modelleket az MLflow `models:/<model-name>/<model-version>` formátumú URI-kkal azonosítja. Ebben az URI-ban megadhatjuk a modell konkrét verzióját, vagy akár mondhatjuk azt is a `latest` kulcsszóval, hogy a legfrissebbet töltse le. Ez utóbbi akkor lehet hasznos, ha kiadunk egy új modellt, és szeretnénk, hogy az API azonnal lekövesse a változásokat.

        2. Készíts egy ONNX futtatókörnyezetet és mentsd el az alkalmazás állapotai közé, hogy később használni tudjuk.

            ```python
            import onnxruntime as rt

            session = rt.InferenceSession(
                model.SerializeToString(),
                providers=["CPUExecutionProvider"]
            )
            state['session'] = session
            ```

        3. Töltsd be az osztályokat tartalmazó JSON fájlt, és ennek tartalmát is mentsd el az állapotok közé.

            ```python
            class_names = None
            if os.path.exists(settings.class_path):
                with open(settings.class_path, 'r') as f:
                    class_names = json.load(f)
            state['class_names'] = class_names
            ```

            !!! note "A `settings` változó"
                Figyeld meg, hogy a beállításokat a `settings` változóból vesszük ki.

4. Egészítsd ki a `predict()` függvényt, hogy használja az előző lépésekben inicializált környezetet.

    1. A függvény bemenetként kap 4 float típusú számot (objektumba csomagolva). Állítsd össze ez alapján a modell bemenő tenzorát.

        ```python
        input_tensor = np.array([[input.sepal_length, input.sepal_width, input.petal_length, input.petal_width]], dtype=np.float32)
        ```

    2. A tenzort és a korábban inicializált környezetet felhasználva végezz predikciót.

        ```python
        input_name = state['session'].get_inputs()[0].name
        output_class, _ = state['session'].run(None, {input_name: input_tensor})
        ```

    3. Határozd meg a kapott osztály nevét, és térj vissza vele.

        ```python
        class_name = f"Class {output_class[0]}"
        if state['class_names'] is not None:
            class_name = state['class_names'][output_class[0]]

        return PredictOutput(class_name=class_name)
        ```

5. Teszteld a kód helyes működését a Swagger interfészen keresztül.

    !!! tip "Tesztadatok"
        A teszt adathalmazból másolj ki néhány sor adatot különböző osztálynevekkel, és teszteld, hogy a modell helyesen felismeri-e azokat.

### Konténer létrehozása

Miután meggyőződtünk róla, hogy az API-nk helyesen működik, következhet annak üzembe helyezése production környezetbe. Ezt egy Docker konténer létrehozásával és futtatásával fogjuk szimulálni. A gyakorlatban, ha készen van a Docker image, azt már bármilyen környezetben futtathatjuk, akár éles szerveren is.

A Docker konténer létrehozásához szükségünk lesz egy Docker Image-re, amit egy [`Dockerfile`](https://docs.docker.com/reference/dockerfile/) leíróval, és annak build-elésével fogunk tudni létrehozni. Az image-re úgy kell gondolni, mint egy önálló, elszigetelt rendszer képfájlra. Ebben futó alkalmazás független lesz az aktuális környezetünktől, így ott is telepíteni kell minden függőséget.

1. Sorold fel a projekt függőségeit az `api` mappán belül található `requirements.txt` fájlban.

    ```text
    fastapi[all]
    onnx
    onnxruntime
    mlflow
    ```

    !!! note "Csomaglista fájlok"
        A `requirements.txt` egy olyan csomaglista fájl, ami alapján egyszerűen tudunk Python környezetet telepíteni. Az aktuális projektünkben ez a `pyproject.toml` fájlnak felel meg, mivel ebben vannak a függőségek. Ahhoz viszont, hogy ezt használjuk, telepíteni kéne a `poetry`-t is a Docker környezetbe, míg a `requirements.txt`-t natívan támogatja. Ezt a fájlt elő tudjuk állítani az aktuális környezet lemásolásával, például a `pip freeze > api/requirements.txt` paranccsal, vagy a `poetry export --without-hashes --output api/requirements.txt` segítségével is, de jelen példában törekedjünk az egyszerűségre, és állítsuk össze kézzel.

2. Hozz létre egy `Dockerfile`-t szintén az `api` mappán belül az alábbiak szerint (szöveges fájl, nincs kiterjesztése):

    1. Származz le a `python:3.12` verziójú képfájlból. Ez biztosítja azt, hogy a Python-t már ne kelljen telepíteni, ez alapértelmezetten használható lesz.

        ```Dockerfile
        FROM python:3.12
        ```

    2. Válaszd ki a `/code` mappát mint munkakönyvtár. Ezen belül fogunk dolgozni. Ez az utasítás után minden parancs ebben a mappában értve fog futni.

        ```Dockerfile
        WORKDIR /code
        ```

    3. Másold át a szükséges fájlokat ebbe a könyvtárba (ilyen lesz az `api.py`, `requirements.txt` és a `data/classes.json`).

        ```Dockerfile
        COPY data/classes.json ./classes.json
        COPY api/api.py ./api.py
        COPY api/requirements.txt ./requirements.txt
        ```

    4. Telepítsd a függőségeket a `requirements.txt` alapján, ehhez a `pip` eszközt tudod használni.

        ```Dockerfile
        RUN pip install --no-cache-dir -r ./requirements.txt
        ```

    5. A `classes.json` fájl helye fixen meghatározott (a projekt gyökerébe másoltuk), így ennek elérési útját egy környezeti változó létrehozásával állítsd be. Ezt később már nem fogjuk változtatni.

        ```Dockerfile
        ENV CLASS_PATH="/code/classes.json"
        ```

    6. Jelezd, hogy a konténer a `80`-as porton keresztül fog kommunikálni (ez alapvetően csak dokumentációs célokra szolgál, nincs konkrét hatása).

        ```Dockerfile
        EXPOSE 80
        ```

        !!! note "80-as port"
            A 80-as port az alapértelmezett HTTP-s webszolgáltatások esetén, a böngészők ezen a porton fognak próbálkozni abban az esetben, ha nem írunk ki explicit portszámot (pl.: csak a [http://localhost/-ot](http://localhost/-ot) írjuk).

    7. Állítsd be a konténer fő folyamatát, ez a production módban futó FastAPI-utasítás lesz.

        ```Dockerfile
        CMD ["fastapi", "run", "./api.py", "--port", "80"]
        ```

### Konténer futtatása

Miután elkészültünk a leíróval, a konténer indítását és paraméterezését a [`docker-compose.yaml`](https://docs.docker.com/reference/compose-file/services/) fájlban fogjuk leírni. Megtehetnénk, hogy a konténert manuálisan buildeljük és futtatjuk, de a compose fájlon keresztül ez sokkal egyszerűbb.

1. Vedd fel az új konténert a `docker-compose.yaml` fájlba a többi szolgáltatáshoz (pl.: mlflow) hasonlóan.

    1. A szolgáltatás neve legyen `api`.

    2. Az `image`-nek tetszőleges nevet adhatsz, ami még nem létezik. Ennek akkor van jelentősége, ha egy már létező, mások által definiált konténert akarunk használni.

        ```yaml
        services:
            ...

            api:
                image: api
        ```

    3. A build contextnek add meg a gyökérkönyvtárat, a dockerfile-nak válaszd ki az `api` mappán belül létrehozott `Dockerfile`-t.

        ```yaml
        build:
            context: .
            dockerfile: api/Dockerfile
        ```

    4. Rendeld a `80`-as portot a konténer `80`-as portjához. Ez fogja ténylegesen kinyitni a portot a konténeren, ezután fogunk tudni kommunikálni vele `localhost`-on keresztül.

        ```yaml
        ports:
            - "80:80"   
        ```

        !!! note "Portütközés"
            Előfordulhat olyan eset, ha már valamilyen más webszolgáltatás fut a számítógépeden, hogy a 80-as port már foglalt. Ebben az esetben csak egyszerűen írd át egy másik portra, pl. 8080-ra (`- 8080:80`). Ne felejtsd el, hogy ilyenkor ki kell írni a portszámot a `localhost` után.

    5. Állítsd be az `MLFLOW_URI` környezeti változót, mutasson a saját MLflow példányra.

        ```yaml
        environment:
            MLFLOW_URI: "http://mlflow:5000"
        ```

        !!! note "Kommunikáció konténerek között"
            Az egy Docker Compose-ban definiált konténerek kommunikálni tudnak egymással. Ez amiatt van, mert alapértelmezetten bekerülnek egy saját hálózatba, ahol elérik egymást. Ezen a hálózaton belül az egyes konténerekre a Compose fájlban megadott nevükkel lehet hivatkozni. Tehát ha a szolgáltatást `mlflow`-nak hívjuk (mivel ez szerepel a `services` felsorolásban), akkor a `http://mlflow` hoston keresztül érjük el a portok figyelembevételével.

    6. Végül állítsd be az `mlflow` konténert függőségnek. Emiatt az automatikusan el fog indulni az `api` konténer indításával.

        ```yaml
        depends_on:
            - mlflow
        ```

2. Futtassuk a szolgáltatást a `docker-compose up -d api` paranccsal, és teszteljük a [http://localhost/docs](http://localhost/docs) elérési utat.

### Beadandó

!!! example "3. feladat beadandó"
    * Commitold a változtatásokat.
    * Készíts egy képernyőképet az API Swagger interfészéről, és mentsd el **`f3-1.png`** néven a repository gyökerébe.
    * Készíts egy képernyőképet a Docker Desktop felületéről, vagy a `docker-compose ps -a` parancs kimenetéről, és mentsd el **`f3-2.png`** néven a repository gyökerébe.

## 4. feladat: API monitorozása

Egy szolgáltatás üzemeltetése közben fontos, hogy időben értesüljünk a hibákról vagy problémákról. Ebben a példában egy egyszerű monitorozást fogunk megvalósítani, viszont ehhez két új szolgáltatásra is szükségünk lesz. A metrikák gyűjtéséhez [Prometheus](https://prometheus.io/)-t, a megjelenítéshez pedig [Grafana](https://grafana.com/)-t fogunk használni.

A monitorozás push-pull elven fog működni, ami azt jelenti, hogy az API-nak egy végponton publikálnia kell a saját metrikáit. Nem tudunk tetszőleges metrikát monitorozni, csak azt, amit az API közzétesz magáról. Ez a végpont mindig csak az aktuális metrikákat mutatja, nem tartalmaz időbeliséget, ezért szükségünk van egy olyan eszközre, ami ezt gyűjteni és sorosítani fogja. Ez lesz a Prometheus. A szolgáltatás bizonyos időközönként lekéri az adatokat az API-tól, és időbélyeggel látja el az adatokat. Ezen felül lehetővé teszi az adatok lekérését a saját lekérdező nyelvével (PromQL), és meg is tudja jeleníteni őket. Egyetlen hátránya, hogy a megjelenítésben nem túl erős. Ezért fogjuk használni a Grafanát, amivel komplex megjelenítést és vizualizációkat lehet létrehozni. Egyedi dashboardokat tudunk benne létrehozni, ahol a saját metrikáinkat tudjuk folyamatosan nyomon követni.

### API metrikák publikálása

1. Egészítsd ki a meglévő API-t, hogy a saját metrikáit ki tudja írni:

    1. Telepítsd a [prometheus-fastapi-instrumentator](https://github.com/trallnag/prometheus-fastapi-instrumentator) csomagot.

        ```powershell
        PS> poetry add prometheus-fastapi-instrumentator
        ```

    2. Importáld az `Instrumentator` osztályt, és kapcsold be az `api.py` fájlban. Ez automatikusan publikálni fogja a `/metrics` végpontot, amin a belső állapotokat publikálja.

        ```python
        from prometheus_fastapi_instrumentator import Instrumentator

        app = ...
        Instrumentator().instrument(app).expose(app)
        ```

    3. Indítsd el az alkalmazást a `fastapi dev api/api.py` utasítással, és teszteld a [/metrics](http://localhost:8000/metrics) végpontot. Nézd meg, milyen formában publikálja a metrikákat és mik ezek.

2. Egészítsd ki a metrikákat egy saját, a modellhez köthető számlálóval. A számláló azt fogja számolni, hogy a modell hány alkalommal prediktálta az egyes osztályokat.

    1. Hozz létre egy új `predicted_class_name` függvényt az `api.py` elején (az `Instrumentator` előtt), ez fogja beburkolni a metrikát.

        ```python
        from typing import Callable
        from prometheus_client import Info

        def predicted_class_name() -> Callable[[Info], None]:
            # Ide jön a metrika kódja
        ```

    2. A függvényen belül hozz létre egy `Counter` objektumot, ez lesz maga a számláló. Töltsd ki alapinformációkkal. A számláló képes több mértéket is számolni, ezeket label-ekkel fogja azonosítani. Itt az osztálynevek lesznek a label-ek.

        ```python
        from prometheus_client import Counter

        METRIC = Counter(
            "predicted_class_name",
            "Number of times a certain class predicted",
            labelnames=("class_name",)
        )
        ```

    3. Hozz létre egy `instrumentation` függvényt a függvényen belül. Ez fogja a kérésre adott válaszból kinyerni, hogy a modell melyik osztályt prediktálta. Ez alapján pedig a megfelelő számlálót növeli.

        ```python
        def instrumentation(info: Info) -> None:
            if info.request.url.path == '/predict' and info.response.status_code == 200:
                class_name = json.loads(info.response.body)['class_name']
                METRIC.labels(class_name).inc()
        ```

    4. Térj vissza az `instrumentation` függvénnyel.

        ```python
        return instrumentation
        ```

    1. Regisztráld az új metrikát az `Instrumentator`-ban, ehhez módosítsd ezt a sort az alábbiak szerint.

        ```python
        (Instrumentator(body_handlers=[r".*"])
        .instrument(app)
        .add(predicted_class_name())
        .expose(app)
        )
        ```

   2. Indítsd újra az alkalmazást, és teszteld az API `/metrics` végpont kimenetét. Először a metrika még nem fog megjelenni a metrikák között, csak akkor, ha végzel néhány predikciót. Próbálj ki több különbözőt, hogy az összes osztálynév megjelenjen.

3. Frissítsd az éles alkalmazást is a Docker konténerben, hogy képes legyen a metrikákat publikálni.

    1. Vedd fel a függőségek közé `requirements.txt`-ben a `prometheus-fastapi-instrumentator` csomagot.

        ```text
        ...
        prometheus-fastapi-instrumentator
        ```

    2. Buildeld és indítsd újra a szolgáltatást.

        ```powershell
        PS> docker-compose up -d --build api
        ```

    3. Teszteld, hogy megjelentek a metrikák a [/metrics](http://localhost/metrics) végponton.

### Prometheus konfigurálása

A Prometheus fogja gyűjteni az adott metrikákat. A szolgáltatás képes akár több rendszer metrikáit is gyűjteni, és ezeket egységesen tárolni. A szolgáltatást a `prometheus.yml` fájl segítségével lehet konfigurálni. Itt kell majd felvenni a monitorozni kívánt szolgáltatások adatait. A szolgáltatás a többihez hasonlóan Docker konténerben fog futni. Ez már elő van készítve a `docker-compose.yaml` fájlban.

1. Adj hozzá egy új `scrape_configs` bejegyzést a `prometheus.yml` fájlba.

    1. Adj meg neki egy tetszőleges `job_name` nevet (pl.: api).
    2. A `metrics_path` legyen `/metrics`, ez fogja mutatni, hol találja a metrikákat.
    3. A `static_configs` alá vegyél fel egy `targets` bejegyzést, ami a szolgáltatás hostnevét tartalmazza. Itt is hasonlóan a docker-compose-ban található névre tudunk hivatkozni, ami szintén `api`. Mivel a 80-as porton kell hallgatózni, ezért a portot nem kell kiírni.

    ```yaml
    global:
        scrape_interval: 5s

    scrape_configs:
        - job_name: 'api'
            metrics_path: /metrics
            static_configs:
            - targets: [ 'api' ]
    ```

2. Indítsd el a `prometheus` Docker konténert a `docker-compose up -d prometheus` parancs segítségével.

3. Vizsgáld meg a Prometheus UI felületét a [http://localhost:9090](http://localhost:9090) linken.

4. Írj néhány [egyszerű lekérdezést](https://prometheus.io/docs/prometheus/latest/querying/examples/), és próbáld ki, hogy hogyan jelennek meg az API metrikái a Prometheus felületén. A lekérdezéseket az `Expression` mezőbe kell írni, és az `Execute` gombbal lehet futtatni.

    ```promql
    predicted_class_name_total{class_name="Iris-versicolor"}
    ```

### Grafana beállítása

Ahogy láttuk, a Prometheus is képes a gyűjtött metrikák vizualizálására, de nagyon limitáltak a lehetőségek. Komplexebb vizualizációkhoz a Grafanát szokták elterjedten használni.

1. Futtasd a Grafana UI felületét a `docker-compose up -d grafana` paranccsal.
2. Látogasd meg a [http://localhost:3000](http://localhost:3000) linket. Alapértelmezetten a felhasználónév és a jelszó is `admin`.
3. Add hozzá a UI-hoz a Prometheus adatforrást.

    1. A bal oldali `☰` menüben válaszd a `Connections > Add new connection` menüpontot.
    2. Válaszd ki a `Prometheus` adatforrást, és menj az `Add new data source` gombra a jobb felső sarokban.
    3. A `Connection` alatt add meg a Prometheus szolgáltatás elérési útját, ami a `http://prometheus:9090` cím lesz.
    4. Mentsd el a konfigurációt a `Save & test` gombbal az oldal alján.
   
4. Hozz létre egy tetszőleges vizualizációt a Prometheus által összegyűjtött adatokra, és mentsd el egy dashboardra.

    1. Menj a `+` ikonra a jobb felső sarokban, és válaszd a `New dashboard` opciót.
    2. Menj az `Add visualization` gombra, majd válaszd a `prometheus` adatforrást.
    3. Alul a metrikák közül a legördülő ablakban válaszd a `predicted_class_name_total` metrikát, a szűrőnél pedig a `class_name`-et.
    4. Alul az `Options` alatt a `Legend`-nél válaszd a `Custom` opciót, és írj `{{class_name}}`-et.
    5. A jobb oldali menüben a `Panel options` alatt add meg a `Predicted Classes` címet a `Title` mezőben.
    6. A grafikon felett jobb felső sarokban válaszd időtartománynak az utolsó 5 percet (`Last 5 minutes`), a `Refresh` alatt pedig a frissítési rátát 5 másodpercnek (`5s`).
    7. Mentsd el a dashboardot a `Save dashboard` opcióval `API` néven, majd menj a `Back to dashboard` gombra.
   
5. Csinálj néhány predikciót az API segítségével, és nézd meg, hogyan változnak az értékek a dashboardon.

### Beadandó

!!! example "4. feladat beadandó"
    * Commitold a változtatásokat.
    * Készíts egy képernyőképet az elkészült Grafana dashboardról, és mentsd el **`f4.png`** néven a repository gyökerébe.

## Opcionális feladat

MI modellek esetén előfordulhatnak hibák, és az is előfordulhat, hogy a felhasználó, aki a mi szolgáltatásunkat használja, pontosan tudja, hogy a modellünk hibázott. Ebben az esetben megkérhetjük rá, hogy segítsen a modell fejlesztésében, és küldje el az adatokat, amelyekben a modellünk hibázott, és ezt fel tudjuk használni a modellünk fejlesztéséhez.

!!! warning "Felhasználói inputok"
    Természetesen a felhasználótól származó inputokat minden esetben validálni kell. Ebben a példában ettől most eltekintünk, de éles környezetben csak olyan adatot használjunk, amiben biztosak vagyunk, hogy helyesek.

Automatizáljuk a tanítási és üzembe helyezési folyamatot. Adjunk lehetőséget a felhasználónak, hogy elküldje a szükséges 4 paramétert, és a várt osztálynevet. Erre reagálva az adatot tegyük be a tanító adatok közé, és finomhangoljuk a modellt. Tegyük meg mindezt automatizáltan. Az automatizáláshoz használjuk a Prefectet.

!!! danger "Fontos"
    Mielőtt ennek a feladatnak nekikezdesz, mindenképp commitolj! Így, ha módosítasz vagy törölsz valamit, akkor is lesz nyoma, hogyan csináltad korábban. De a legjobb, ha az opcionális feladatot újabb kódok hozzáadásával oldod meg, és nem a meglévő felülírásával.

!!! warning "Segédeszközök"
    Ez egy nehéz feladat, ha még korábban nem használtad ezeket a technológiákat. Emiatt ne tántorodj el, ha elsőre nem tudod, hogyan kellene megoldani a feladatot. A legfontosabb információkat itt összefoglaljuk neked, de ezen felül **bátran használj AI-eszközöket** (pl. Copilot, ChatGPT …stb.). Fontos ugyanakkor, hogy **kritikusan** próbáld megoldani a feladatot: **ne elégedj meg az első megoldással**, próbáld meg megérteni, miért úgy kell megoldani, és **csak azt fogadd el, amit értesz** is. Törekedj az egyszerű, átlátható megoldásra, amit te is **el tudnál magyarázni** egy csoporttársadnak. Ha élsz ezzel a lehetőséggel, **mindenképp csatolj promptnaplót** a megoldásod mellé, hogy ellenőrizni tudjuk a gondolatmenetet.

### Prefect

A [Prefect](https://docs.prefect.io/v3/get-started) egy Python-alapú gépi tanulási folyamatok automatizálására és ütemezésére szolgáló eszköz. Két fő részből áll:

* **Prefect Server**: központi vezérlő és UI, amely nyomon követi a futásokat.
* **Prefect Worker**: végrehajtó folyamat, amely a szervertől érkező feladatokat futtatja.

Automatizáláshoz a feladatokat `@flow` és `@task` dekorátorokkal definiáljuk Pythonban, majd a worker a szerver által kiadott runokat végrehajtja, így könnyen kezelhetők ütemezett tanítások vagy finetune folyamatok. A worker esetén két különböző logikával találkozhatunk:

* **Általános worker**: bármilyen flow-t futtat, több különböző feladatot is képes kezelni ugyanabból a work poolból.
* **Feladatspecifikus worker**: egy adott feladatra van optimalizálva, és csak az adott folyamat kódját tartalmazza, így izoláltabb működést biztosít.

A Prefect tipikusan konténerizált komponensekből áll, és mindegyik külön-külön Docker konténerként futtatható.

### Megoldás menete

1. A finomhangoláshoz szükségünk lesz az eredeti modellre is, nem csak az ONNX-re, mivel azt csak futtatásra használjuk. Ezért módosítsd az eredeti `main.py` logikát, hogy a legjobb modell esetén ne csak az ONNX-et, hanem az scikit-learn-ös modellt is regisztrálja, például `IrisDev` néven. A modell elérési útja hasonló, mint a korábbi esetben, csak a `/onnx` helyett `/model` néven érjük el.
2. Hozz létre egy `finetune` mappát a projekt gyökerébe, ebbe a mappába készítsd el a finomhangoláshoz szükséges logikát.
3. Hozd létre a modell finomhangolásához szükséges kódot egy `finetune.py` fájlba. A finomhangolás menete a következő:

    1. Töltsd be a korábbi tanító és tesztadatokat, továbbá a finomhangoláshoz használt adatokat (ez például lehet egy `finetune.csv` fájl, ami jelenleg még üres).
    2. Fűzd össze a tanító és finomhangoló adatokat, jegyezd meg, melyik sor mihez tartozik.
    3. Töltsd be a legfrissebb modellt (eredeti, nem ONNX) az MLflow Model Registryből.
    4. Növeld a modell tanításhoz használt iterációinak számát egy kis számmal (pl. 5-tel), csökkentsd a kiinduló tanulási rátát (learning rate) egy nagyságrenddel, és állíts be *warm startot*. Ez utóbbi fogja biztosítani, hogy a modellt tovább tanítja, és nem új tanítást kezd.
    5. Tanítsd a modellt az összefűzött adatokkal, de adj meg `0.7`-es súlyértéket az eredeti tanító adatok soraira, és `1.0`-ás súlyértéket a finomhangoló sorokra. Ezt a `fit()` függvény `sample_weight` paraméterével lehet szabályozni. Minden sorhoz 1 értéket kell megadni.
    6. A korábbiakhoz hasonlóan a tanítást kövesd nyomon az MLflow segítségével, és a folyamat végén konvertáld ONNX formátumba, majd logold a modellt.
    7. A kapott modell működését ellenőrizd a tesztadatok segítségével. Ha a tesztadatokon továbbra is 90% feletti pontosságot ér el a modell, regisztráld az eredeti és az ONNX-változatot, mint új verziót.
   
4. Futtass egy [Prefect Server Docker konténert](https://hub.docker.com/r/prefecthq/prefect). Ehhez vedd fel a `docker-compose.yaml` fájlba ennek konfigurációját.
5. Készíts Prefect pipeline-t a finomhangoló logikából.

    1. Telepítsd a `prefect-client` csomagot.
    2. A fő utasítást `@flow`, az esetleges alfüggvényeket `@task` dekorátorral kell ellátni.
    3. A pipeline paraméterül kapja meg a 4 inputot és az elvárt osztálynevet.
    4. A paramétereket fűzze hozzá a `finetune.csv` végére. Amennyiben a finomhangoló adatok száma elér egy bizonyos mennyiséget (pl. 5 darab), indítsuk el a finomhangoló logikát. Ezután megint csak akkor finomhangolunk, ha újabb 5 adat érkezik (ekkor már 10 adattal).
    5. Teszteld a kód működését. A pipeline-t a Prefect UI felületéről lehet manuálisan elindítani.
   
6. Csomagold be a finomhangoló kódot egy [feladatspecifikus worker konténerbe](https://docs.prefect.io/v3/how-to-guides/deployment_infra/serve-flows-docker).

    1. Hozd létre a szükséges `requirements.txt` és `Dockerfile` fájlokat. Itt nagyon hasonlóan kell eljárnod, mint az API esetén.
    2. Vedd fel a `finetune` szolgáltatást is a `docker-compose.yaml`-be, hasonlóan a korábbiakhoz.
    3. Ne felejtsd el beállítani a környezeti változókat a többi szolgáltatásra. Szükség lesz egy `PREFECT_API_URL`-re és egy `MLFLOW_URL`-re. Az utóbbit neked kell majd kézzel feldolgozni és beállítani az `mlflow.set_tracking_uri(...)` utasításban. A környezeti változót az `os.environ.get(...)` utasítással lehet lekérni.
    4. A szolgáltatásnak fel kell majd venned egy `volume`-ot is. A volume lényegében egy olyan mappa, amit elérhetővé teszel a konténernek a konténeren kívülről. Ilyen lehet például a `data` mappa, ahol tárolod majd a `finetune.csv` fájlt. Így a futtatások között ennek értéke állandó lesz.
    5. Teszteld a konténer működését.
   
7. Hozz létre egy új végpontot az API-n, ahol 4 *double* és 1 *string* paramétert várunk.

    1. Készítsd el a végponthoz szükséges modell osztályokat (pl. `SuggestInput`).
    2. A végpontban a kapott érték alapján futtasd a Prefect pipeline-t egy trigger eseménnyel. A Prefect API leírását megtalálod a [https://docs.prefect.io/v3/api-ref](https://docs.prefect.io/v3/api-ref) címen.
    3. A trigger elkészítéséhez vedd fel a Prefect Server elérési útját a `Settings`-be, és add meg az értékét a `docker-compose.yaml`-ben.
    4. Derítsd ki a pipeline azonosítóját (ezt ugyan lehet manuálisan is, de célszerű automatizáltan csinálni). Ehhez használd a [`/api/deployments/name/{flow-name}`](https://docs.prefect.io/v3/api-ref/rest-api/server/deployments/read-deployment-by-name) végpontot.
    5. Indítsd el a folyamatot a megfelelő paraméterek átadásával, a [`/api/deployments/{deployment_id}/create_flow_run`](https://docs.prefect.io/v3/api-ref/rest-api/server/deployments/create-flow-run-from-deployment) végpont segítségével.
   
8. Módosítsd az API kódját úgy, hogy új modellverzió esetén automatikusan az új modellt használja.

    1. A modellek aktuális verzióit az `MLFlowClient` osztály `search_model_versions` metódusával lehet lekérni.
    2. Az API `init` függvényében a modell mellett mentsük el annak verzióját is.
    3. A prediktálás előtt hasonlítsuk össze az elérhető legújabb verziót és az aktuális verziót. Ha új verzió érhető el, mentsük le azt a modellt. (Az adott predikció emiatt lehet, hogy kicsit tovább fog tartani, de nem jelentősen. Éles környezetben erre lehetne egy újabb automatizmust definiálni, ami jelzi az új modell jelenlétét, és azt automatikusan a háttérben letölti.)
    4. Frissítsd az API kódját a Docker konténerben, és teszteld a működését.

### Beadandó

!!! example "Opcionális feladat beadandó (10 pont)"
    * Commitold a változtatásokat.  
    * Készíts egy képernyőképet a Prefect UI felületéről, ahol látszik, hogy lefutott egy finomhangoló pipeline, és mentsd el **`fo.png`** néven a repository gyökerébe.

    Ezen felül, amennyiben használtál valamilyen AI-eszközt a feladat megoldásához, mellékelj **promptnaplót**:

    * A napló tartalmazza szöveges formátumban a feltett kérdéseket és az arra kapott válaszokat,
    * Szerepeljen benne az is, hogy melyik modellt kérdezted, és mikor (dátum elég, időpont nem kell),
    * A promptnaplót mentsd el a projekt gyökerébe `prompts.log` néven, és **commitold**.
