# GeDa Pràctica 2 — CSV, pandas i matplotlib

---

**Assignatura**: GeDa - Gestió de Dades: Comunicacions, Programació i Simulació  
**Titulació**: Ciències i Tecnologies del Mar  
**Autor**: Enoc Martínez  
**Departament**: Departament d'Enginyeria Electrònica (EEL)  
**Contacte**: enoc.martinez@upc.edu  

<p align="center">
  <img height="100" src="https://github.com/EnocMartinez/citm-geda-2026/blob/main/resources/banner.png?raw=true" alt="banner">
</p>

---

## Introducció

Tractarem **17 anys de mesures** de dos sensors diferents de l'OBSEA — un CTD al fons del mar i una estació meteorològica a la boia de superfície — i els hauràs de llegir, filtrar, agregar, dibuixar i combinar per respondre una pregunta que amb un sol fitxer no es pot respondre:

> **Quant triga el mar a assabentar-se que ha arribat l'estiu?**

### Com funciona aquesta pràctica

El LAB1 tenia espais `____` per omplir. **Aquest no.** Cada tasca té dues parts:

> **Com funciona** — un exemple complet i executable sobre un DataFrame de 6 files, on veus l'entrada i la sortida senceres.
>
> **Ara et toca** — què has de fer sobre el fitxer real. **Sense codi.** Te'l dono tot excepte la línia que has d'escriure tu.

Perquè puguis saber si ho has encertat sense preguntar-ho, cada tasca acaba amb la **sortida esperada**. Si el teu número no coincideix, encara no ho tens.

El símbol 📝 marca una pregunta que has de respondre a l'informe.

### Objectius

* Carregar un CSV real amb pandas, i fer front a un fitxer que no es llegeix bé al primer intent
* Inspeccionar un conjunt de dades i saber dir què hi tens
* Seleccionar, filtrar i agregar amb pandas
* Escriure una funció de neteja reutilitzable i cobrar-ne els dividends
* **Dibuixar amb matplotlib**: sèries temporals, histogrames, panells, bandes de dispersió i núvols de punts
* Combinar dos conjunts de dades de procedència diferent en un de sol

### Què cal lliurar

Lliura a l'Atenea els fitxers següents:

* Un **informe en format PDF** amb captures del codi **que has escrit tu**, explicant cada tasca, i les respostes a totes les preguntes marcades amb 📝.
* L'script final de Python amb totes les tasques en un únic fitxer anomenat **`LAB2.py`**, que es pugui executar de dalt a baix sense errors.
* El fitxer `obsea_seabed_net.csv` que generaràs a la Tasca 6.
* **Totes les figures** de les tasques 7, 8 i 9, en PNG.
* L'informe ha d'incloure una secció `Ús d'IA` que descrigui quines eines has fet servir i per què.

---

## Preparació

### 1. Les llibreries

Al LAB1 només vas fer servir Python pelat. Avui necessites dues llibreries externes. Amb l'entorn virtual del teu projecte activat:

```bash
pip install pandas matplotlib
```

Comprova que han entrat:

```python
import pandas as pd
import matplotlib.pyplot as plt

print(pd.__version__)
```

### 2. Els fitxers de dades

Necessites `obsea_seabed_TS.csv` i `obsea_buoy_AIRT.csv` d'aquesta carpeta. Pots clonar tot el repositori:

```bash
git clone https://github.com/EnocMartinez/citm-geda-2026.git
cd citm-geda-2026/LAB2
```

o descarregar els dos fitxers des de la pàgina web de GitHub (obre'ls i clica **Download raw file**).

> ⚠️ Mantén `LAB2.py` i els dos CSV **a la mateixa carpeta**, o Python no trobarà les dades.

### 3. El DataFrame de prova

Tots els exemples d'aquest guió es fan sobre aquest DataFrame de 6 files. Copia'l al principi del teu `LAB2.py`: el faràs servir per entendre cada operació abans d'aplicar-la a 236.085 files.

```python
import pandas as pd

exemple = pd.DataFrame({
    "time": pd.to_datetime([
        "2024-07-01 00:00",
        "2024-07-01 06:00",
        "2024-07-01 12:00",
        "2024-07-01 18:00",
        "2024-07-02 00:00",
        "2024-07-02 06:00",
    ]),
    "TEMP":    [21.4, 21.9, 23.1, 22.0, float("nan"), 21.7],
    "TEMP_QC": [1,    1,    1,    4,    9,            1],
}).set_index("time")

print(exemple)
```

```
                     TEMP  TEMP_QC
time
2024-07-01 00:00:00  21.4        1
2024-07-01 06:00:00  21.9        1
2024-07-01 12:00:00  23.1        1
2024-07-01 18:00:00  22.0        4
2024-07-02 00:00:00   NaN        9
2024-07-02 06:00:00  21.7        1
```

Sis files que contenen, en miniatura, els defectes del fitxer real: una mesura marcada com a dolenta i un valor absent.

---

## Abans de començar: tres idees noves

### Què és un DataFrame

Al LAB1 vas guardar un perfil CTD en **tres llistes paral·leles**: `depths`, `temperatures`, `salinities`. Funcionava, però tenia un problema que potser no vas notar: si ordenaves una llista, les altres dues deixaven de correspondre-hi. Les tres llistes sabien els números, però cap d'elles sabia que anaven juntes.

Un **DataFrame** és una taula on les columnes van lligades. Si mous una fila, es mou sencera.

```
            ┌─────────┬─────────┬──────────┐
            │  TEMP   │  PSAL   │ TEMP_QC  │   ← columnes, cadascuna d'un tipus
┌───────────┼─────────┼─────────┼──────────┤
│ 10:00:00  │  21.4   │  37.9   │    1     │
│ 10:30:00  │  21.9   │  37.9   │    1     │   ← files
│ 11:00:00  │  23.1   │  38.0   │    4     │
└───────────┴─────────┴─────────┴──────────┘
      ↑
    l'índex
```

Tres paraules que faràs servir tot el dia:

* **DataFrame** — la taula sencera.
* **Series** — una sola columna. `df["TEMP"]` és una Series.
* **Índex** — l'etiqueta que identifica cada fila. Aquí és el temps, i això és el que fa que `df.loc["2024-07"]` tingui sentit.

### Els mètodes

`df` és un **objecte** i `df.mean()` és un **mètode** seu: una funció que viu dins de l'objecte i actua sobre les seves dades.

Al LAB1, per obtenir la mitjana havies d'escriure un bucle. Avui és `df["TEMP"].mean()`. El bucle no ha desaparegut: l'ha escrit algú altre, millor que tu, en C.

### Els *flags* de qualitat

Les dades reals no són totes bones. Cada mesura d'aquests fitxers porta al costat una columna amb un ***flag* de qualitat** (`TEMP_QC`) — un número que resumeix el veredicte del control de qualitat sobre aquella mesura concreta:

| *Flag* | Significat |
|---|---|
| **1** | correcta |
| 2 | no avaluada |
| 3 | sospitosa |
| 4 | errònia |
| 9 | absent |

Fixa't que les dolentes **no s'han esborrat del fitxer**: s'han marcat. Qui publica les dades no sap què en faràs tu, i esborrar és irreversible.

---

## Les dades

Dues plataformes del mateix observatori, a 4 km de la costa de Vilanova i la Geltrú.

| Fitxer | Què és | Variable | Període | Files |
|---|---|---|---|---|
| `obsea_seabed_TS.csv` | CTD a l'estació del fons, **20 m de fondària** | `TEMP` (°C), `PSAL` (PSU) | 2009-05 → 2026-10 | 236.085 |
| `obsea_buoy_AIRT.csv` | Estació meteorològica de la **boia de superfície** | `AIRT` (°C) | 2011-10 → 2026-05 | 121.652 |

Són descàrregues directes del servidor ERDDAP de l'OBSEA, sense tocar. Tots els defectes que trobaràs avui són reals.

---

# Enunciat de la pràctica

**Objectiu:** en acabar tindràs un únic script (`LAB2.py`) que llegeix els dos fitxers crus, els filtra amb una funció que hauràs escrit tu, els combina, i produeix una col·lecció de figures que responen la pregunta del dia.

---

## Tasca 1 — El fitxer que no es llegeix

**Objectiu:** descobrir que carregar un CSV real no és una línia, sinó tres passos, i veure què fa cadascun.  
**Informe**: captura del primer intent fallit, i dels `dtypes` després de cada pas.

### Com funciona

Carregar un CSV amb pandas és, en principi, una sola funció:

```python
df = pd.read_csv("un_fitxer.csv")
```

### Ara et toca

**1a.** Carrega `obsea_seabed_TS.csv` així, sense cap argument, i imprimeix dues coses: `df.dtypes` i `df.head(2)`.

Hauries de veure això:

```
time        object
TEMP        object
PSAL        object
TEMP_QC    float64
PSAL_QC    float64
dtype: object

                   time  TEMP     PSAL  TEMP_QC  PSAL_QC
0                   UTC  degC  Dmnless      NaN      NaN
1  2009-05-29T18:30:00Z   NaN  37.6358      NaN      1.0
```

`object` vol dir *text*. La temperatura ha entrat com a text, i per tant **no hi pots fer cap operació aritmètica**.

**1b.** Obre `obsea_seabed_TS.csv` amb el teu IDE i mira les tres primeres línies. Ja saps què ha passat: és la mateixa trampa que al LAB1 et va fer petar el `float()`. La segona línia del fitxer no és una mesura, són les **unitats**, i una columna que barreja `degC` amb números només pot ser text.

A partir d'aquí ho farem **en tres passos separats**, imprimint el resultat de cadascun. Podries fer-ho tot dins del `read_csv`, però així veuràs exactament què canvia a cada pas.

**Pas 1 — salta la fila de les unitats.** Busca l'argument `skiprows` a la [documentació de `read_csv`](https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html) i torna a carregar el fitxer. Imprimeix `df.dtypes`.

```
time        object
TEMP       float64
PSAL       float64
TEMP_QC    float64
PSAL_QC      int64
```

`TEMP` ja és un número. `time` encara és text.

**Pas 2 — converteix `time` en una data de debò.** La funció és `pd.to_datetime()`, i es fa sobre la columna:

```python
df["time"] = pd.to_datetime(df["time"])
```

Afegeix-hi després aquesta línia, que treu la zona horària per simplificar la resta de la pràctica:

```python
df["time"] = df["time"].dt.tz_localize(None)
```

Imprimeix `df["time"].dtype`. Ha de dir `datetime64[ns]`.

**Pas 3 — fes que `time` sigui l'índex**, no una columna qualsevol:

```python
df = df.set_index("time")
```

Imprimeix `df.head(2)` i fixa't que `time` ha baixat una línia: ja no és una columna, és l'etiqueta de cada fila.

```
                     TEMP     PSAL  TEMP_QC  PSAL_QC
time
2009-05-29 18:30:00   NaN  37.6358      NaN        1
2009-05-29 19:00:00   NaN  37.6341      NaN        1
```

Finalment, imprimeix quantes files tens, el primer i l'últim dia, i els tipus de les columnes.

**Sortida esperada:**

```
236085 files, de 2009-05-29 a 2026-10-01
TEMP: float64 · PSAL: float64 · TEMP_QC: float64
L'índex és de tipus: datetime64[ns]
```

> 📝 Quina fila concreta del fitxer feia que `TEMP` fos `object`? Per què `TEMP_QC` sí que era `float64` des del primer moment, si estava al mateix fitxer?

> 📝 Al LAB1 vas llegir un CSV amb `open()`, un bucle i `split(",")`. Aquí han estat tres línies. Què t'ha estalviat exactament pandas?

> 📝 Què guanyes fent que el temps sigui l'índex en comptes de deixar-lo com una columna més? Pensa-hi ara i torna-hi a la Tasca 3.

---

## Tasca 2 — Què tinc a les mans?

**Objectiu:** saber descriure un conjunt de dades que no has vist mai.  
**Informe**: captura de la sortida de `describe()` i les respostes.

### Com funciona

Quatre mètodes contesten quatre preguntes diferents. Prova'ls sobre el DataFrame de prova i mira què dona cadascun:

```python
print(exemple.shape)       # (6, 2)        quantes files i columnes
print(exemple.head(3))     #               les primeres files
print(exemple.dtypes)      #               de quin tipus és cada columna
print(exemple.describe())  #               resum estadístic de les numèriques
```

Fixa't en una cosa de `describe()` sobre l'exemple: diu `count = 5` per a `TEMP`, però el DataFrame té **6** files. El `count` només compta els valors que hi són.

I aquests dos no són el mateix:

```python
print(type(exemple["TEMP"]))     # <class 'pandas.core.series.Series'>
print(type(exemple[["TEMP"]]))   # <class 'pandas.core.frame.DataFrame'>
```

### Ara et toca

Aplica els quatre mètodes al fitxer real i imprimeix-ne la sortida.

**Sortida esperada** (`describe()`, arrodonit a 3 decimals):

```
             TEMP        PSAL     TEMP_QC     PSAL_QC
count  227727.000  235595.000  227920.000  236085.000
mean       17.382      37.743       1.012       1.061
std         3.865       0.615       0.235       0.435
min        11.642      30.666       1.000       1.000
25%        14.010      37.673       1.000       1.000
50%        16.163      37.875       1.000       1.000
75%        20.541      38.061       1.000       1.000
max        28.180      38.444       9.000       9.000
```

> 📝 `shape` diu que hi ha **236.085** files, però `describe()` diu `count = 227.727` per a `TEMP`. On són les altres? Escriu el codi que ho demostri amb un número.

> 📝 La temperatura mínima del fitxer és 11,64 °C i la màxima 28,18 °C. Mira la columna `PSAL`: el mínim és 30,67 PSU, quan l'aigua mediterrània és d'uns 38. Et sembla una mesura creïble? Què faries per comprovar-ho? *(no cal que ho facis encara — ho resoldràs a la Tasca 3)*

---

## Tasca 3 — Seleccionar, filtrar, i aplicar el control de qualitat

**Objectiu:** quedar-te amb el tros de dades que t'interessa, i saber sempre quant has llençat.  
**Informe**: captura del codi de les seleccions i dels recomptes.

### Com funciona

Hi ha dues maneres de triar files, i les faràs servir totes dues.

**Per etiqueta de l'índex**, amb `.loc`. Com que l'índex és temps, pandas entén els trossos de data escrits com a text:

```python
print(exemple.loc["2024-07-01"])     # tot el dia 1 de juliol
```

**Per condició**, amb una màscara booleana. Això és el pas que costa més d'entendre, així que mira'l en dos temps:

```python
mascara = exemple["TEMP"] > 22        # primer, una Series de True/False
print(mascara)

print(exemple[mascara])               # després, les files on era True
```

```
time
2024-07-01 00:00:00    False
2024-07-01 06:00:00    False
2024-07-01 12:00:00     True
2024-07-01 18:00:00    False
2024-07-02 00:00:00    False
2024-07-02 06:00:00    False
Name: TEMP, dtype: bool

                     TEMP  TEMP_QC
time
2024-07-01 12:00:00  23.1        1
```

Dues condicions es combinen amb `&` (i) i `|` (o). **Cada condició ha d'anar entre parèntesis**, o Python s'equivoca d'ordre:

```python
print(exemple[(exemple["TEMP"] > 22) & (exemple["TEMP_QC"] == 1)])
```

I per saber quants valors diferents té una columna, `value_counts()`:

```python
print(exemple["TEMP_QC"].value_counts())
```

### Ara et toca

Sobre el fitxer real, fes aquestes quatre coses i imprimeix el resultat de cadascuna:

**3a.** Quantes mesures hi ha del **juliol de 2024**, i quina n'és la temperatura mitjana.

**3b.** Quantes mesures superen els **24 °C**, i quin percentatge del total són.

**3c.** D'aquestes, quantes són **del mes d'agost**. *(pista: `df.index.month` et dona el número de mes de cada fila)*

**3d.** El recompte dels *flags* `TEMP_QC`. Després, queda't **només amb els de *flag* 1** i imprimeix quantes mesures has descartat i quin percentatge representen.

**Sortida esperada:**

```
Juliol de 2024: 283 mesures, mitjana 21.09 °C
Mesures per sobre de 24 °C: 15544 (6.6 % del total)
...i d'aquestes, a l'agost: 6122

TEMP_QC
1.0    226955
2.0        44
3.0       772
9.0       149

Ens quedem amb 226955 mesures bones
Descartades: 9130 (3.87 %)
```

> 📝 Has descartat el 3,87 % de les mesures. Imagina que publiques una figura feta amb el 96,13 % restant i no ho dius enlloc. És un frau, una omissió, o és irrellevant? Justifica-ho.

> 📝 Torna a la salinitat de 30,67 PSU de la Tasca 2. Quin *flag* de qualitat té aquella mesura? Comprova-ho amb codi *(pista: `df["PSAL"].idxmin()` et dona l'instant on passa)* i digues si el sistema de qualitat l'havia detectada.

---

## Tasca 4 — Valors absents, i la funció de neteja

**Objectiu:** saber què falta al fitxer, i **empaquetar la decisió de què fer-ne en una funció**.  
**Informe**: captura del recompte d'absents, de la funció sencera, i les respostes.

### 4a. Els valors absents

#### Com funciona

```python
print(exemple.isna().sum())          # quants absents per columna
print(exemple.dropna())              # treu les files que en tenen
print(exemple["TEMP"].interpolate()) # els omple amb una recta entre veïns
```

#### Ara et toca

Compta els valors absents de cada columna del fitxer real.

**Sortida esperada:**

```
TEMP       8358
PSAL        490
TEMP_QC    8165
PSAL_QC       0
```

Ara mira **com de llargs** són els buits. Aquest codi te'l dono fet, perquè la tècnica (agrupar per sèries consecutives) encara no la coneixes:

```python
diaria = df["TEMP"].resample("D").mean()
buit_max = diaria.isna().astype(int).groupby(diaria.notna().cumsum()).sum().max()
print(f"Buit més llarg sense cap dada: {buit_max} dies")
```

```
Buit més llarg sense cap dada: 273 dies
```

> 📝 `interpolate()` ompliria aquest buit amb una línia recta de 273 dies. Dibuixa mentalment què hi posaria entre un desembre i el setembre següent. Per què seria pitjor que deixar-hi el forat?

### 4b. La funció

Ja tens les dues decisions preses: **només *flag* 1**, i **fora els absents**. Ara empaqueta-les.

#### Ara et toca

Escriu una funció:

```python
def neteja_serie(df, variable):
    ...
```

que rebi un DataFrame i el **nom** d'una variable (`"TEMP"`, per exemple), i retorni un DataFrame d'una sola columna que:

1. es quedi només amb les files de *flag* 1 — recorda que la columna del *flag* es diu `variable + "_QC"`;
2. tregui els valors absents.

Comprova-la primer sobre el DataFrame de prova. Ha de deixar quatre files, sense la mesura de *flag* 4 ni la de *flag* 9:

```
                     TEMP
time
2024-07-01 00:00:00  21.4
2024-07-01 06:00:00  21.9
2024-07-01 12:00:00  23.1
2024-07-02 06:00:00  21.7
```

I després sobre el fitxer real:

```
neteja_serie(df, 'TEMP'): 236085 -> 226955 mesures
Descartades: 9130 (3.87 %)
```

> ⚠️ Si no obtens exactament **226955**, revisa-la ara. Totes les tasques restants en depenen.

> 📝 Per què la funció rep el nom de la variable com a argument, en comptes de tenir `"TEMP"` escrit a dins? Ho sabràs valorar de debò a la Tasca 9.

> 📝 La funció retorna `df[[variable]]` (amb dobles claudàtors) i no `df[variable]`. Mira la Tasca 2 i digues quina diferència hi ha, i per què aquí ens convé la primera.

---

## Tasca 5 — Columna calculada, i agregar en el temps

**Objectiu:** operar sobre columnes senceres, i entendre les dues maneres d'agregar en el temps, que semblen la mateixa i no ho són.  
**Informe**: captura de la columna calculada, de les tres agregacions, i les respostes.

### Com funciona

Una operació sobre una columna sencera s'escriu com si fos sobre un sol número:

```python
exemple["TEMP_F"] = exemple["TEMP"] * 9 / 5 + 32
print(exemple)
```

pandas l'aplica a totes les files alhora, sense que hi hagis d'escriure cap bucle. Això es diu **vectorització**, i és la manera normal de treballar amb pandas.

Per agregar en el temps tens dues eines que fan coses diferents. `resample` recorre el temps i talla trossos consecutius:

```python
net = neteja_serie(exemple, "TEMP")
print(net["TEMP"].resample("D").mean())
```

```
time
2024-07-01    22.133333
2024-07-02    21.700000
```

`groupby` **plega** el temps i ajunta tot el que comparteix una propietat, encara que estigui a anys de distància:

```python
print(net.groupby(net.index.hour)["TEMP"].mean())
```

```
time
0     21.4
6     21.8
12    23.1
```

Les freqüències de `resample` que necessites: `"D"` (diària), `"ME"` (final de mes), `"YE"` (final d'any).

### Ara et toca

Sobre la sèrie neta de la Tasca 4:

**5a.** Afegeix-hi una columna `TEMP_F` amb la temperatura en graus Fahrenheit (`F = C × 9/5 + 32`), vectoritzada, i imprimeix les tres primeres files.

```
                       TEMP  TEMP_F
time
2010-02-26 23:00:00  12.430  54.373
2010-02-26 23:30:00  12.431  54.377
2010-02-27 00:00:00  12.432  54.378
```

**5b.** Calcula la mitjana **diària** i la mitjana **mensual** amb `resample`, i la **climatologia** amb `groupby`: la temperatura mitjana de cada mes de l'any, agafant tots els anys alhora. *(pista: `df.index.month`)*

Imprimeix quants valors té cadascuna, i els 12 valors de la climatologia.

**Sortida esperada:**

```
resample('D').mean()  -> 4858 valors
resample('ME').mean() -> 180 valors
groupby(mes).mean()   -> 12 valors

1     13.95
2     13.23
3     13.32
4     14.25
5     15.85
6     17.82
7     21.18
8     23.11
9     23.38
10    20.92
11    17.75
12    15.39
```

> ⚠️ Guarda't la sèrie de **mitjanes diàries** en una variable. La faràs servir a les tasques 6, 7, 8 i 9.

> 📝 Les dues últimes operacions agrupen «per mes», i una dona **180** valors i l'altra **12**. Explica la diferència, i digues quina pregunta respon cadascuna.

> 📝 La climatologia diu que el mes més càlid al fons del mar és el **setembre**, no l'agost. Guarda't aquesta observació: a la Tasca 9 l'hauràs d'explicar.

---

## Tasca 6 — Desar i tornar a llegir

**Objectiu:** tancar el cicle, i descobrir que un número que passa per un fitxer de text no torna exactament igual.  
**Informe**: captura de les primeres línies del fitxer generat i de la comprovació.

### Com funciona

```python
exemple.to_csv("prova.csv")
tornat = pd.read_csv("prova.csv")
```

### Ara et toca

**6a.** Desa la sèrie de **mitjanes diàries** de la Tasca 5 a `obsea_seabed_net.csv`. Obre'l amb el teu IDE i comprova que té capçalera i 4.858 files de dades.

**6b.** Torna'l a llegir **sense arguments** i imprimeix el tipus de la columna `time`. Et tornarà a sortir `object`: el fitxer és text, i el temps ha tornat com a text.

**6c.** Torna'l a llegir **bé**, amb `parse_dates` i `index_col`.

**6d.** Verifica que el que has rellegit és el que havies escrit. Compta quants valors són **exactament iguals** amb `==`, i quina és la diferència màxima real entre les dues sèries.

**Sortida esperada:**

```
Escrites 4858 files a obsea_seabed_net.csv
Sense arguments, 'time' torna com a: object
Amb parse_dates i index_col, l'índex és: datetime64[ns]

Valors idèntics bit a bit: 3586 de 4858
Valors que NO coincideixen amb '==': 1272
Màxima diferència real: 3.552713678800501e-15
```

> 📝 **1.272 valors de 4.858 no tornen idèntics**, però la diferència màxima és de 3,5 × 10⁻¹⁵ °C. Què ha passat exactament quan el número ha passat per un fitxer de text? *(pista: quantes xifres decimals cabien al CSV?)*

> 📝 Al LAB1, la Tasca 6 comparava un valor rellegit amb `==` i deia `True`. Mira per què allà funcionava i aquí no. Com s'han de comparar dos nombres decimals?

---

## Tasca 7 — Les primeres gràfiques

**Objectiu:** aprendre a dibuixar amb matplotlib, i veure que el tipus de gràfica que tries és una decisió, no un detall.  
**Informe**: les quatre figures, amb el codi de cadascuna.

### Com funciona

Una figura de matplotlib té **dues peces**, i confondre-les és l'error més comú del principi:

* la **Figure** (`fig`) és el full de paper: la mida, el títol general, i és qui sap desar-se a un fitxer;
* els **Axes** (`ax`) són el sistema d'eixos dibuixat damunt del full: les dades, les etiquetes dels eixos, la llegenda, la graella.

Un full pot tenir un sol sistema d'eixos o deu. Es demanen tots dos alhora:

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(9, 4))     # un full, un sistema d'eixos
ax.plot(exemple.index, exemple["TEMP"])    # les dades van a l'ax
ax.set_xlabel("Data")                      # les etiquetes, també
ax.set_ylabel("Temperatura (°C)")
ax.set_title("Un títol que digui alguna cosa")
ax.grid(alpha=0.3)
fig.tight_layout()                         # el full s'encarrega de l'espaiat
fig.savefig("prova.png", dpi=200)          # i de desar-se
```

Mètodes de l'`ax` que faràs servir avui:

| Mètode | Què fa |
|---|---|
| `ax.plot(x, y)` | línia (i punts, si hi poses `marker="o"`) |
| `ax.scatter(x, y)` | núvol de punts |
| `ax.hist(valors, bins=40)` | histograma |
| `ax.fill_between(x, baix, dalt)` | omple una banda entre dues corbes |
| `ax.errorbar(x, y, yerr=...)` | punts amb barres d'error |
| `ax.axvline(x)` / `ax.axhline(y)` | una línia vertical / horitzontal de referència |
| `ax.set_xlim(a, b)` / `ax.set_ylim(a, b)` | retallar els eixos |
| `ax.set_xticks(...)` / `ax.set_xticklabels(...)` | decidir què es llegeix a l'eix |
| `ax.legend()` | la llegenda, a partir dels `label=` de cada corba |

> ⚠️ Si fas servir el teu IDE, afegeix `plt.show()` al final per veure la figura. Si no, desa-la amb `fig.savefig()` i obre el PNG.

### Ara et toca

**7a. La sèrie sencera.** Dibuixa la mitjana diària de temperatura dels 17 anys, amb una línia fina (`linewidth=0.6`), els dos eixos etiquetats **amb unitats**, títol i graella. Desa-la com `fig_serie_completa.png`.

Mira-la bé abans de passar endavant: hi veus el cicle anual? Hi veus els forats?

**7b. Un any, amb dues sèries al mateix eix.** Queda't amb el **2024** (`diaria.loc["2024"]`) i dibuixa-hi dues coses alhora:

* la mitjana diària, amb línia fina i color clar,
* la mitjana mensual de la Tasca 5, també del 2024, amb `marker="o"` i color fosc.

Posa-hi `label=` a cada `ax.plot()` i crida `ax.legend()`. Desa-la com `fig_2024.png`.

```
fig_2024.png — 298 dies el 2024
```

**7c. Un histograma.** Dibuixa la distribució de **totes** les temperatures netes (no les diàries) amb `ax.hist(..., bins=40)`, i afegeix-hi una línia vertical vermella a trossos (`linestyle="--"`) a la mitjana, amb el valor escrit a la llegenda. Desa-la com `fig_histograma.png`.

```
mitjana 17.36 °C, mediana 16.15 °C
```

**7d. Dos panells.** Neteja també la salinitat — **amb la mateixa funció de la Tasca 4**, passant-li `"PSAL"` — i fes-ne la mitjana diària. Després dibuixa les dues variables en **dos panells, un sobre l'altre, compartint l'eix del temps**:

```python
fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(10, 6), sharex=True)
```

Cada panell amb la seva etiqueta de l'eix Y (`TEMP (°C)` i `PSAL (PSU)`), l'etiqueta `Data` només a baix, i un títol general amb `fig.suptitle()`. Desa-la com `fig_dos_panells.png`.

```
4959 dies de salinitat
```

> 📝 A la figura 7a, la línia que uneix els dos extrems d'un forat de 273 dies sembla una mesura, i no ho és. Com ho dibuixaries perquè es vegi que allà no hi ha dades?

> 📝 L'histograma de 7c no té forma de campana: hi ha un pic molt alt cap als 13–14 °C, una cua llarga cap als 25 °C, i la mitjana (17,36 °C) cau en una zona on hi ha relativament poques mesures. Mira la figura 7b i explica d'on surt aquesta forma. Què representa millor aquestes dades, la mitjana o la mediana (16,15 °C)?

> 📝 Als dos panells de 7d, la salinitat té caigudes brusques i estretes que la temperatura no té. Mira si coincideixen amb res de l'altre panell, i proposa una hipòtesi.

---

## Tasca 8 — Mitjana i dispersió: la banda ±1σ

**Objectiu:** una mitjana sola menteix. Aprendre a dibuixar la mitjana **i** la variabilitat al mateix temps.  
**Informe**: les dues figures, la taula, i les respostes.

### Com funciona

Un `groupby` pot calcular més d'una cosa. Si en vols la mitjana i la desviació estàndard:

```python
mitjana = diaria.groupby(diaria.index.month).mean()
desviacio = diaria.groupby(diaria.index.month).std()
```

I per dibuixar la banda entre `mitjana - desviacio` i `mitjana + desviacio`:

```python
ax.plot(mitjana.index, mitjana, marker="o", label="Mitjana mensual")
ax.fill_between(mitjana.index,
                mitjana - desviacio,
                mitjana + desviacio,
                alpha=0.2, label="±1 desviació estàndard")
```

`alpha` és la transparència: 0 és invisible, 1 és opac. Una banda opaca taparia la corba.

### Ara et toca

**8a.** A partir de les **mitjanes diàries**, calcula la mitjana i la desviació estàndard de cada mes de l'any. Posa-les juntes en un DataFrame i imprimeix-lo. Digues també quin mes té més variabilitat i quin en té menys.

**Sortida esperada:**

```
      mitjana  desviacio
time
1       13.95       0.58
2       13.22       0.47
3       13.32       0.59
4       14.27       0.85
5       15.87       1.23
6       17.88       1.57
7       21.20       2.10
8       23.13       2.08
9       23.39       1.84
10      20.95       1.65
11      17.77       1.44
12      15.40       0.93

Mes amb més variabilitat: 7 (σ = 2.10 °C)
Mes amb menys variabilitat: 2 (σ = 0.47 °C)
```

**8b.** Dibuixa-ho: la corba de la mitjana amb `marker="o"`, la banda ±1σ amb `fill_between`, i **noms de mes en comptes de números** a l'eix X:

```python
mesos = ["Gen", "Feb", "Mar", "Abr", "Mai", "Jun",
         "Jul", "Ago", "Set", "Oct", "Nov", "Des"]
ax.set_xticks(range(1, 13))
ax.set_xticklabels(mesos)
```

Amb llegenda, graella, eixos etiquetats amb unitats i títol. Desa-la com `fig_banda_sigma.png`.

**8c.** El mateix, però **per any** i amb una altra eina: agrupa les mitjanes diàries per `diaria.index.year` i dibuixa la mitjana anual amb barres d'error:

```python
ax.errorbar(mitjana_anual.index, mitjana_anual, yerr=desviacio_anual,
            fmt="o", capsize=4)
```

Desa-la com `fig_anual.png`.

```
2010    15.25
2011    19.47
2012    15.54
2013    16.58
2014    17.52
2015    17.20
2016    15.55
2017    17.77
2018    17.19
2019    17.98
2020    17.90
2021    18.67
2022    17.54
2023    17.73
2024    17.25
2025    17.93
2026    19.39
```

> 📝 El juliol té σ = 2,10 °C i el febrer σ = 0,47 °C: al juliol, el mateix dia de l'any pot ser 4 °C més càlid o més fred segons l'any. Per què l'estiu és tan més variable que l'hivern a 20 m de fondària?

> 📝 A la figura 8c, el 2010 i el 2011 tenen mitjanes anuals molt diferents de la resta, i el 2026 també. Abans de parlar de cap tendència climàtica, mira la figura 7a i digues què té d'especial aquests anys. Quina conclusió **no** pots treure d'aquesta figura?

> 📝 La climatologia de la Tasca 5 (13,95 · 13,23 · 13,32 …) i la de la 8a (13,95 · 13,22 · 13,32 …) no donen exactament el mateix, tot i sortir de les mateixes dades. Explica per què. *(pista: una fa la mitjana de totes les mesures, i l'altra la mitjana de les mitjanes diàries)*

---

## Tasca 9 — Les dues plataformes, i la resposta

**Objectiu:** respondre la pregunta del dia combinant dos conjunts de dades que ningú no havia ajuntat per tu.  
**Informe**: les dues figures, el codi, i les respostes. Aquesta és la tasca que es valora més.

### Com funciona

Per ajuntar dues sèries que comparteixen l'índex de temps, `pd.concat` amb `axis=1` les posa una al costat de l'altra i **les alinea per l'índex**, no per l'ordre:

```python
a = pd.Series([1.0, 2.0, 3.0], index=pd.to_datetime(["2024-07-01", "2024-07-02", "2024-07-03"]))
b = pd.Series([9.0, 8.0],      index=pd.to_datetime(["2024-07-02", "2024-07-03"]))

totes = pd.concat({"a": a, "b": b}, axis=1)
print(totes)
print(totes.dropna())
```

```
              a    b
2024-07-01  1.0  NaN      ← el dia 1 no té 'b'
2024-07-02  2.0  9.0
2024-07-03  3.0  8.0

              a    b
2024-07-02  2.0  9.0      ← dropna() es queda només amb els dies complets
2024-07-03  3.0  8.0
```

Guardar-ho tot (`concat`) o quedar-se només amb el que és comú (`concat` + `dropna`) **no és una decisió tècnica, és una decisió científica**.

### Ara et toca

**9a.** Carrega `obsea_buoy_AIRT.csv`. És el mateix format que el del fons: els tres passos de la Tasca 1 valen igual.

**9b.** Neteja'l. **Amb la funció de la Tasca 4**, que ara rep `"AIRT"` en comptes de `"TEMP"`.

> Si vas escriure bé la funció, aquest pas són **dues línies**. Si vas anar fent les coses a pèl, ara les has de tornar a fer totes. Això és el que volia dir el LAB1 quan et va fer refactoritzar la fórmula de Mackenzie.

**9c.** Passa la sèrie de l'aire a **mitjanes diàries**, ajunta-la amb la del mar amb `pd.concat(axis=1)`, i compta els dies que tens de cada manera: amb tots els dies, i només amb els dies que tenen les dues mesures.

**Sortida esperada:**

```
Boia: 121652 files, de 2011-10-06 a 2026-05-12
Netejada amb neteja_serie(): 121652 -> 121054 mesures

Dies de mar:  4858
Dies d'aire:  2941

Unió completa (outer): 5369 dies
Només dies amb totes dues (inner): 2430 dies
```

**9d. El núvol de punts.** Dibuixa un `scatter` amb la temperatura de l'aire a l'eix X i la del mar a l'eix Y, **només amb els dies comuns**. Fes-hi tres coses més:

* **acoloreix cada punt segons el mes**, amb `c=comunes.index.month, cmap="twilight_shifted"`, i afegeix-hi la barra de color:
  ```python
  punts = ax.scatter(comunes["AIRT"], comunes["TEMP"], c=comunes.index.month,
                     cmap="twilight_shifted", s=8, alpha=0.6)
  barra = fig.colorbar(punts, ax=ax, ticks=range(1, 13))
  barra.set_label("Mes")
  barra.ax.set_yticklabels(mesos)
  ```
* dibuixa-hi la recta `y = x` (`ax.plot([5, 30], [5, 30], "k--")`), que marca on l'aire i el mar estarien a la mateixa temperatura;
* posa els dos eixos amb els **mateixos límits** (`set_xlim`, `set_ylim`) i `ax.set_aspect("equal")`, perquè la recta surti realment a 45°.

Calcula també la correlació entre les dues sèries amb `comunes["AIRT"].corr(comunes["TEMP"])` i posa-la al títol. Desa-la com `fig_scatter.png`.

```
Correlació AIRT–TEMP (dies comuns): 0.821
```

**9e. La resposta.** Calcula la **climatologia mensual de totes dues** sobre els dies comuns, i imprimeix en quin mes té el pic cadascuna i quina amplitud anual té (màxim menys mínim).

**Sortida esperada:**

```
       TEMP   AIRT
1     13.85  10.10
2     13.09   9.78
3     13.51  12.26
4     14.60  14.10
5     15.85  16.87
6     17.85  21.29
7     20.57  24.13
8     23.13  24.83
9     23.40  22.53
10    21.01  19.42
11    17.35  14.89
12    15.43  12.53

Pic de l'aire: mes 8 (24.8 °C)
Pic del mar:   mes 9 (23.4 °C)
Amplitud de l'aire: 15.1 °C
Amplitud del mar:   10.3 °C
```

**9f. La figura final.** Les dues climatologies al mateix eix, cadascuna amb la seva **banda ±1σ** (com a la Tasca 8), colors diferents, `marker` diferent, noms de mes a l'eix X, etiquetes amb unitats, títol, llegenda i graella. Desa-la amb `dpi=200` com `cicle_aire_mar.png`.

Afegeix-hi un peu de figura amb la font i el nombre de dies:

```python
fig.text(0.5, -0.01, f"Font: OBSEA. {len(comunes)} dies amb mesura simultània.",
         ha="center", fontsize=8, color="#555555")
fig.savefig("cicle_aire_mar.png", dpi=200, bbox_inches="tight")
```

> 📝 Inclou la figura a l'informe. L'aire fa el pic a l'**agost** i el mar al **setembre**, i el mar oscil·la **5 °C menys** al llarg de l'any. Explica totes dues coses amb el mateix argument físic.

> 📝 A la teva figura les dues corbes **es creuen dues vegades**. Digues en quins mesos, i què vol dir cada encreuament: qui està més calent que qui, i per què canvia.

> 📝 Mira el núvol de punts de 9d. Els punts **no formen una recta, formen un bucle**: per a una mateixa temperatura de l'aire (posem 20 °C) el mar pot estar a 16 °C o a 23 °C segons el color del punt. Quins mesos hi ha a cada branca del bucle? Quina informació dona aquesta figura que la figura 9f no dona?

> 📝 La correlació surt 0,82, prou alta. Si només haguessis vist aquest número i no la figura, quina conclusió equivocada podries haver tret?

> 📝 Tenies 4.858 dies de mar i 2.941 d'aire, i quedar-te només amb els comuns n'ha deixat **2.430**. Explica a què correspon cada número. Si haguessis fet servir els 5.369 dies de la unió completa, què hauria passat amb la climatologia de l'aire?

> 📝 Aquesta pregunta no es podia respondre amb cap dels dos fitxers per separat. Repassa les nou tasques i digues quines eren **imprescindibles** per arribar al resultat, i quines eren només pràctica.

---

## Resum

Avui has après a:

- carregar un **CSV real** que no es llegeix bé al primer intent, pas a pas, i saber per què no es llegia
- **inspeccionar** un conjunt de dades desconegut i dir quantes dades hi falten
- **seleccionar i filtrar** amb `.loc` i amb màscares booleanes, i comptar sempre què llences
- aplicar els ***flags* de qualitat**, i escriure una **funció de neteja** reutilitzable de la qual cobres els dividends cinc tasques més tard
- **agregar** en el temps de dues maneres diferents, i saber quina respon quina pregunta
- **desar i rellegir**, i que un decimal que passa per un fitxer de text no torna exactament igual
- **dibuixar amb matplotlib**: sèries temporals, histogrames, panells compartits, bandes de dispersió, barres d'error i núvols de punts acolorits — i triar quin tipus respon cada pregunta
- **combinar dues fonts** de dades i respondre una pregunta que cap de les dues podia respondre sola

El pipeline que acabes d'escriure — llegir, validar, filtrar, agregar, visualitzar, combinar, desar — és literalment el que fa un servei de dades oceanogràfiques. La diferència és la mida del fitxer i qui et paga per fer-ho.

**Al LAB3** aquestes dades deixaran d'arribar d'un fitxer que algú t'ha deixat a GitHub: les anirà a buscar el teu propi codi, i un bot t'avisarà quan passi alguna cosa interessant.
