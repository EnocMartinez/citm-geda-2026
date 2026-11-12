# GeDa Pràctica 4 — Dades de Copernicus Marine: descàrrega, exploració i vídeo

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

En aquest pràctica descarregarem dades de manera automatitzada des de copernicus, crearem un mapa per cada instant de 
temps (`pcolormesh`) i finalment crearem un video ajuntant tots els mapes. En aquesta pràctica es combinen els 
coneixements i recursos de totes les altres sessions, en concret:

| Funció | Exemples a  |
|---|----|
| Obrir un NetCDF amb `xarray` i mirar què conté | T06a |
| `.to_dataframe()` — i el seu avís sobre quantes files genera | T06a |
| `pcolormesh` amb barra de colors, `vmin`/`vmax` | T06b |
| `fig.savefig()` per desar una figura | LAB2, Tasca 7   |
| Un bucle `for` sobre els instants d'una sèrie temporal | LAB1, Tasca 5 / T06a |
| Seleccionar un instant concret (`.isel()`, `.sel()`)   | T06a  |


### Objectius

* Automatitzar la descàrrega d'un conjunt de dades real des d'una API d'autenticació (Copernicus Marine)
* Triar una regió i un interval de temps *abans* de descarregar
* Reconstruir el flux de treball NetCDF → DataFrame → mapa
* Generar una seqüència d'imatges numerades correctament i ajuntar-les en un vídeo



### Què cal lliurar

Lliura a l'Atenea:

* Un fitxer **`zip`** amb el teu script (`LAB4.py`) i el vídeo final (`.mp4`).
* Un informe curt en PDF i una secció `Ús d'IA`.

---

## Preparació

### 1. Llibreries

Per a aquesta sessió et faran falta les següents llibreries:

```bash
pip install copernicusmarine xarray pandas matplotlib
```

### 2. Compte de Copernicus Marine

Crea't un compte gratuït a **[data.marine.copernicus.eu](https://data.marine.copernicus.eu/register)**. Un cop tinguis compte, configura l'accés **un sol cop des del terminal**, amb el teu usuari i contrasenya:

```bash
# Windows
py -c "import copernicusmarine; copernicusmarine.login()"

# Linux / Mac
python -c "import copernicusmarine; copernicusmarine.login()"
```

A Windows, si `python` tampoc funciona, prova amb el *launcher* `py`:

```bash
py -c "import copernicusmarine; copernicusmarine.login()"
```

Et demanarà l'usuari i la contrasenya per terminal. Qualsevol dels dos mètodes guarda les credencials al teu ordinador; no cal tornar-ho a fer als scripts següents.

### 3. ffmpeg

Necessites el binari `ffmpeg` instal·lat i accessible des del terminal (no és una llibreria de Python):

* **macOS**: `brew install ffmpeg`
* **Windows**: `winget install ffmpeg` (o descarrega'l de [ffmpeg.org](https://ffmpeg.org/download.html) i afegeix-lo al PATH)
* **Linux**: `sudo apt install ffmpeg`

Comprova que funciona:

```bash
ffmpeg -version
```


---

## Les dades

**Producte**: [MEDSEA_ANALYSISFORECAST_PHY_006_013](https://data.marine.copernicus.eu/product/MEDSEA_ANALYSISFORECAST_PHY_006_013/description) — model físic del Mediterrani, Centro Euro-Mediterraneo sui Cambiamenti Climatici.

| | |
|---|---|
| **Dataset** | `cmems_mod_med_phy-tem_anfc_4.2km-2D_PT1H-m` — SST horària, 2D |
| **Variable** | `thetao` (°C) |
| **Resolució nativa** | ~1/24° (uns 4,2 km) |
| **Regió suggerida** | costa catalana → Còrsega: `lon 0°–9,5° E`, `lat 38°–43,5° N` |

> ⚠️ Aquest producte té una **finestra mòbil d'uns 27 mesos**.

---

## Exemple 1 — Baixar dades de Copernicus Marine

```python
import copernicusmarine

lon_min = 0
lon_max = 9.5
lat_min = 38
lat_max = 43.5

bbox = [lon_min, lon_max, lat_min, lat_max]  # útil per fixar l'extensió del mapa amb cartopy

copernicusmarine.subset(
    dataset_id="cmems_mod_med_phy-tem_anfc_4.2km-2D_PT1H-m",
    variables=["thetao"],
    minimum_longitude=lon_min, maximum_longitude=lon_max,
    minimum_latitude=lat_min, maximum_latitude=lat_max,
    start_datetime="2025-01-01T00:00:00",
    end_datetime="2025-01-31T23:00:00",
    output_filename="medsea_sst.nc",
    skip_existing=True,   # si ja l'has baixat, no el tornis a descarregar
)

```

Una sola crida per a tot l'interval que et calgui — **no facis un bucle que el cridi un cop per dia**. Cada crida a `subset()` té un cost fix important; cridar-la 31 vegades per baixar un mes és molt més lent i consumeix molta més xarxa que cridar-la un cop amb tot l'interval.

## Exemple 2 — Crear un vídeo a partir d'una carpeta d'imatges

Per a crear un video a partir d'un conjunt d'imatges podem utilitzar l'eina ffmpeg. `synthetic_frames.zip` conté exemples amb frames individuals. Fixa't que el seu nom és per ordre alfabètic ascendent estricte. Perquè puguis provar aquest pas, descarrega't el fitxer zip i descomprimeix-lo **dins d'una carpeta pròpia**. Després, crea un script de python amb el següent contingut:

```python
import subprocess
import os

subprocess.run([
    "ffmpeg", "-y",
    "-framerate", "24",
    "-i", os.path.join("synthetic_frames", "frame_%04d.png"),
    "-c:v", "libx264",
    "-pix_fmt", "yuv420p",
    "video_prova.mp4",
], check=True)
```

`%04d` li diu a `ffmpeg` que busqui `frame_0000.png`, `frame_0001.png`, ... en ordre. Si els teus noms de fitxer no tenen el mateix nombre de dígits, `ffmpeg` (i l'ordre alfabètic que l'enunciat et demana) es confon: `frame_10.png` queda alfabèticament abans que `frame_2.png`. Fes servir sempre el mateix nombre de zeros.

### Conversió de figures matplotlib a imatges

Per a guardar una figura com a imatge utilitzant la llibreria matplotlib, pots utilitzar la funció **`fig.savefig()`**. Recorda que l'hauràs d'executar **un cop per cada instant de temps, dins d'un bucle**.

```python
fig.savefig(f"frames/frame_{i:04d}.png", dpi=90)
```

**`os.makedirs(..., exist_ok=True)`** — la primera vegada que el teu script intenti desar un PNG dins d'una carpeta `frames/` que encara no existeix, petarà amb `FileNotFoundError`. Crea-la abans del bucle, un cop:

```python
import os
os.makedirs("frames", exist_ok=True)
```

`exist_ok=True` evita que peti si la carpeta ja hi és — per exemple, si tornes a executar l'script.

### Obtenir tots els valors de temps

Per a obtenir cadascun dels valors de temps, podem utilitzar la funció `np.unique()` de la llibreria numpy. Aquesta 
funció retorna un array amb valors únics.

```python
import numpy as np

time_slices = np.unique(ds.time)
```

### Evitar regenerar les imatges que ja s'han fet
Per a evitar tornar a generar imatges ja processades, es pot mirar si el fitxer ja existeix al disc dur. Ho podem
aconseguir amb la funció `os.path.exists()`, que retorna `True` si el fitxer existeix o `False` si no.

```python
import os
filename = "frame_0000.png"
if not os.path.exists(filename):
    # generate pccolormesh
else:
    print("File already exists! skipping")

```

### Fixar els mapa de color
Per a evitar que els valors del mapa de color canviïn a cada frame, podem utilitzar els valors màxims i mínim de la
variable quan generem el `pcolormesh`:

```python
ax.pcolormesh(
    data.longitude,
    data.latitude,
    data.thetao,
    vmin=sst_min,  # <== definim el valor mínim de sst aquí
    vmax=sst_max,  # <== definim el valor màxim de sst aquí
    cmap="RdYlBu_r",
)
```


---

# Enunciat de la pràctica

**Objectiu**: un únic script (`LAB4.py`) que baixa SST horària del Mediterrani occidental, en genera un mapa per instant, i ajunta els mapes en un vídeo.


1. Baixa un **mes** (gener de 2025) de dades amb l'Exemple 1.
2. Explora el NetCDF: quantes dimensions té, quins instants, quin rang de temperatura.
3. Guarda els valors de temperatura mínima i màxima.
4. Guarda tots els valors de temps en un array.
5. Per a cada instant, dibuixa un `pcolormesh` amb barra de colors, i desa'l a una carpeta `frames/` amb `fig.savefig()`, numerat i amb zeros davant.
6. Ajunta els frames de gener en un vídeo, amb la comanda de l'Exemple 2.




