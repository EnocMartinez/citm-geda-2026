# GeDa Pràctica 1 — Variables, llistes, funcions i fitxers

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

Benvingut a la teva primera sessió pràctica de GeDa. En acabar aquesta pràctica hauràs escrit un programa que llegeix un perfil CTD real d'un fitxer, calcula la velocitat del so a cada profunditat, desa el resultat i el representa gràficament — i podràs assenyalar el **canal SOFAR** a la teva pròpia figura.

Allà on vegis `____`, hi ha un espai en blanc que has d'omplir. El símbol 📝 marca una pregunta que has de respondre a l'informe.

### Objectius

* Variables, tipus bàsics i aritmètica en Python 3
* Llistes, bucles `for` i condicionals `if`
* Escriure i reutilitzar les teves pròpies funcions
* Llegir i escriure fitxers de text
* Entendre per què la velocitat del so a l'oceà no és constant

### Què cal lliurar

Lliura a l'Atenea els fitxers següents:

* Un informe en format PDF amb captures del teu codi explicant cada tasca.
* En algunes tasques hi ha preguntes, marcades amb 📝, que cal respondre a l'informe.
* L'informe ha d'incloure una secció `Ús d'IA` que descrigui quines eines has fet servir i per què.
* L'script final de Python amb totes les tasques en un únic fitxer anomenat `LAB1.py`.

### Com obtenir els fitxers

Necessites `ctd_profile.csv` d'aquesta carpeta. Pots clonar tot el repositori:

```bash
git clone https://github.com/EnocMartinez/citm-geda-2026.git
cd citm-geda-2026/LAB1
```

o descarregar només aquest fitxer des de la pàgina web de GitHub (obre'l i clica **Download raw file**).

> ⚠️ Mantén `LAB1.py` i `ctd_profile.csv` **a la mateixa carpeta**, o Python no trobarà el fitxer de dades.

---

## Preparació — abans de la primera tasca

### 1. Instal·la Python 3 i un IDE

Necessites dues coses diferents, i val la pena entendre'n la diferència:

* **Python 3** és l'*intèrpret* — el programa que realment executa el teu codi.
* **Un IDE** és l'*editor* — on escrius el codi còmodament. No executa res per si mateix; li demana a Python que ho faci.

Instal·la Python 3 des de [python.org](https://www.python.org/downloads/) (marca **"Add Python to PATH"** a Windows) i després [PyCharm Community](https://www.jetbrains.com/pycharm/download/) o [Visual Studio Code](https://code.visualstudio.com/).

### 2. El teu primer script

Crea un fitxer anomenat `hello_world.py` amb una sola línia:

```python
print("Hola, oceà!")
```

## Abans de començar: cinc idees de programació que faràs servir avui

### Variables i tipus

Una variable és un nom que guarda un valor perquè el puguis reutilitzar sense tornar-lo a escriure — una capsa etiquetada.

```python
depth = 25.4          # float   -> un número amb decimals
n_samples = 12        # int     -> un número enter
station = "OBSEA"     # str     -> text, sempre entre cometes
is_valid = True       # bool    -> True o False
```

Python dedueix el tipus tot sol. Li ho pots preguntar:

```python
print(type(depth))        # <class 'float'>
print(type(n_samples))    # <class 'int'>
print(type(station))      # <class 'str'>
```

Els tipus importen, perquè Python es nega a barrejar-los:

```python
print(n_samples + depth)      # 37.4   -> correcte, tots dos són números
print(station + " és una estació")   # OBSEA és una estació  -> correcte, tots dos són text
print(station + n_samples)    # TypeError!     -> text més número no té sentit
```

Per barrejar-los expressament, cal convertir primer:

```python
print(station + " té " + str(n_samples) + " mostres")   # str() converteix un número en text
print(float("25.4") + 1)                                # float() converteix text en número -> 26.4
```

### Llistes

La majoria de vegades no tenim un sol valor sinó tota una seqüència — una mesura per cada profunditat, per exemple. Una **llista** guarda una seqüència ordenada, i s'escriu amb claudàtors:

```python
temperatures = [24.1, 20.4, 15.6, 12.1]

print(temperatures[0])     # primer element  -> 24.1   (Python compta des de 0!)
print(temperatures[1])     # segon element   -> 20.4
print(temperatures[-1])    # últim element   -> 12.1
print(len(temperatures))   # quants n'hi ha  -> 4
```

Les llistes comencen buides i creixen amb `append`:

```python
speeds = []                # una llista buida
speeds.append(1533.8)      # afegeix un valor al final
speeds.append(1525.2)
print(speeds)              # [1533.8, 1525.2]
```

### Bucles `for`

Un bucle `for` repeteix el mateix bloc de codi un cop per cada element d'una llista. Fixa't en els dos punts i en la indentació — Python fa servir la indentació en comptes de claus, i no és opcional.

```python
temperatures = [24.1, 20.4, 15.6]

for t in temperatures:
    print("Temperatura:", t)
```

```
Temperatura: 24.1
Temperatura: 20.4
Temperatura: 15.6
```

Molt sovint necessites la *posició* a més del valor, per poder consultar el mateix índex en una segona llista. `range(len(...))` et dona 0, 1, 2, ...:

```python
depths       = [0, 50, 100]
temperatures = [24.1, 20.4, 15.6]

for i in range(len(depths)):
    print("A", depths[i], "m la temperatura és", temperatures[i], "C")
```

```
A 0 m la temperatura és 24.1 C
A 50 m la temperatura és 20.4 C
A 100 m la temperatura és 15.6 C
```

### Condicionals `if`

Un `if` executa un bloc només quan una condició és certa:

```python
temperature = 1.8

if temperature < 2:
    print("Avís: aquesta aigua és molt freda")
elif temperature > 30:
    print("Avís: aquesta aigua és sospitosament càlida")
else:
    print("La temperatura sembla normal")
```

```
Avís: aquesta aigua és molt freda
```

Les condicions es poden combinar amb `or` i `and`:

```python
if temperature < 2 or temperature > 30:
    print("Fora de l'interval vàlid")
```

### Funcions

Una funció és una recepta reutilitzable amb nom: li dones unes entrades, fa una feina i retorna un resultat amb `return`. Escrius el càlcul un sol cop i el fas servir tantes vegades com vulguis.

```python
def add_five(x):
    return x + 5

print(add_five(10))    # -> 15
print(add_five(2.5))   # -> 7.5
```

Amb tants arguments com necessitis:

```python
def rectangle_area(width, height):
    return width * height

print(rectangle_area(3, 4))     # -> 12
print(rectangle_area(10, 2.5))  # -> 25.0
```

### Llegir i escriure fitxers

Per llegir un fitxer de text, obre'l i recorre les seves línies amb un bucle. `with` garanteix que el fitxer es tanca encara que alguna cosa falli:

```python
with open("ctd_profile.csv", "r") as f:      # "r" = read (llegir)
    for line in f:
        print(line)
```

Cada `line` arriba com a **text**, incloent-hi el caràcter de salt de línia invisible del final. Dues eines ho netegen:

```python
line = "0,24.10,36.52\n"

clean = line.strip()          # elimina el salt de línia -> "0,24.10,36.52"
parts = clean.split(",")      # separa per les comes     -> ['0', '24.10', '36.52']

print(parts[0])               # '0'      <- encara és text!
print(float(parts[0]))        # 0.0      <- ara és un número
```

Escriure funciona igual, però amb `"w"` en comptes de `"r"`. `\n` és el caràcter de salt de línia — sense ell tot acaba en una sola línia:

```python
with open("results.csv", "w") as f:          # "w" = write (escriure, sobreescriu el fitxer!)
    f.write("depth_m,sound_speed_ms\n")
    f.write("0,1533.78\n")
```

---

## La part científica: per què la velocitat del so no és constant

A l'aire, el so viatja a uns 340 m/s. A l'aigua de mar és d'uns **1500 m/s** — però no exactament, i aquest "no exactament" és el que fa interessant l'acústica submarina.

La velocitat del so a l'aigua de mar augmenta amb totes tres:

| Propietat | Efecte |
|---|---|
| **Temperatura** | l'efecte més fort a prop de la superfície — l'aigua càlida és més ràpida |
| **Salinitat** | l'efecte més feble — l'aigua més salada és lleugerament més ràpida |
| **Pressió (profunditat)** | dominant a l'oceà profund — l'aigua més fonda és més ràpida |

Aquests efectes tiben en direccions oposades a mesura que baixes. La temperatura cau ràpidament a través de la termoclina i frena el so; però la pressió no para de créixer i l'accelera. En algun punt intermedi hi ha un **mínim**, i el so que entra en aquesta capa hi queda refractat i atrapat. Aquesta guia d'ones és el **canal SOFAR**, i és per això que el cant d'una balena pot recórrer milers de quilòmetres.

Avui el trobaràs en dades reals.

### L'equació de Mackenzie

Mackenzie (1981) va ajustar un polinomi de nou termes a mesures experimentals:

**c = 1448.96 + 4.591·T − 5.304×10⁻²·T² + 2.374×10⁻⁴·T³ + 1.340·(S−35) + 1.630×10⁻²·D + 1.675×10⁻⁷·D² − 1.025×10⁻²·T·(S−35) − 7.139×10⁻¹³·T·D³**

on **T** és la temperatura en °C, **S** és la salinitat en PSU, **D** és la profunditat en metres, i **c** en surt en m/s.

Només és vàlida per a **2 ≤ T ≤ 30 °C**, **25 ≤ S ≤ 40 PSU** i **0 ≤ D ≤ 8000 m**. Recorda-ho — és important a la Tasca 5.

En Python, `10⁻²` s'escriu `1e-2`, i `T²` s'escriu `t**2`.

---

# Enunciat de la pràctica

**Objectiu:** en acabar tindràs un únic script (`LAB1.py`) que llegeix `ctd_profile.csv`, calcula la velocitat del so a cada profunditat, escriu el resultat en un fitxer nou i representa el perfil.

---

## Tasca 1 — Hola, oceà

**Objectiu:** agafar confiança amb variables, tipus i `print()` abans que hi entri gens d'oceanografia.  
**Informe**: afegeix una captura del codi i de la seva sortida.

```python
name = "____"                       #  posa-hi el teu nom (o el de la teva parella)
print("Hola, em dic:", name)

temperature = 24.1                  # graus Celsius
salinity = 36.5                     # PSU
depth = 0                           # metres

print("Temperatura:", temperature, "C")
print("Salinitat:", salinity, "PSU")
print("Profunditat:", depth, "m")

print("Tipus de temperature:", ____(temperature))   #  quina funció informa del tipus?
print("Tipus de depth:", ____(depth))               #  aquí igual
```

> 📝 `temperature` i `depth` són tots dos números, però Python n'informa de dos tipus diferents. Quins són, i quina diferència hi ha?

Ara prova aquesta línia i després **esborra-la**, un cop hagis vist què passa:

```python
print("La profunditat és " + depth)     # això peta expressament
```

> 📝 Copia el missatge d'error a l'informe. De què es queixa Python, i què canviaries perquè funcionés?

---

## Tasca 2 — La velocitat del so, per les braves

**Objectiu:** calcular la velocitat del so en tres masses d'aigua diferents.  
**Informe**: afegeix una captura del codi i dels tres resultats.

Aquí tens l'equació de Mackenzie per a la primera massa d'aigua. Omple els dos espais en blanc:

```python
# 1. Aigua mediterrània superficial a l'estiu
t = 24.1
s = 36.5
d = 0

c = (1448.96
     + 4.591 * t
     - 5.304e-2 * t**2
     + 2.374e-4 * ____            #  aquest terme necessita T al cub
     + 1.340 * (s - 35)
     + 1.630e-2 * d
     + 1.675e-7 * d**2
     - 1.025e-2 * t * (s - 35)
     - 7.139e-13 * t * ____)      #  aquest terme necessita D al cub

print("Mediterrània superficial:", round(c, 2), "m/s")
```

Sortida esperada:

```
Mediterrània superficial: 1533.76 m/s
```

Ara fes el mateix per a dues masses d'aigua més. **Copia i enganxa** tot el bloc dues vegades i canvia només els tres valors d'entrada:

```python
# 2. Aigua Intermèdia Llevantina
t = 13.5
s = 38.7
d = 400

# 3. Aigua atlàntica profunda
t = 2.5
s = 34.9
d = 3000
```

Hauries d'obtenir:

```
Intermèdia Llevantina:  1512.85 m/s
Atlàntica profunda:     1510.34 m/s
```

> 📝 L'Aigua Intermèdia Llevantina és **11 °C més càlida** que l'aigua atlàntica profunda, i tot i així les dues velocitats del so són gairebé idèntiques. Explica per què, fent servir la taula d'efectes de més amunt.

> 📝 Acabes d'escriure la mateixa fórmula de nou termes tres vegades. Imagina't que hi trobes una errata en un dels termes. En quants llocs l'hauries de corregir, i com n'estàs de segur que els detectaries tots?

---

## Tasca 3 — Escriu la funció

**Objectiu:** escriure la fórmula **una sola vegada**, i mai més.  
**Informe**: afegeix una captura del codi i de la sortida de la comprovació.

Aquesta última pregunta és tot el sentit de les funcions. Empaqueta la fórmula, dona-li un nom i deixa que rebi els tres valors com a arguments:

```python
def sound_speed(t, s, d):
    """Velocitat del so a l'aigua de mar (Mackenzie 1981), en m/s."""
    c = (1448.96
         + 4.591 * t
         - 5.304e-2 * t**2
         + 2.374e-4 * t**3
         + 1.340 * (s - 35)
         + 1.630e-2 * d
         + 1.675e-7 * d**2
         - 1.025e-2 * t * (s - 35)
         - 7.139e-13 * t * d**3)
    return ____                    #  què hauria de retornar la funció?
```

Comprova que reprodueix la Tasca 2, en tres línies en comptes de trenta:

```python
print(round(sound_speed(24.1, 36.5, 0), 2))       # -> 1533.76
print(round(sound_speed(13.5, 38.7, 400), 2))     # -> 1512.85
print(round(sound_speed(____, ____, ____), 2))    #  els valors de l'atlàntica profunda -> 1510.34
```

**Autocomprovació.** El valor que tothom fa servir per verificar una implementació de Mackenzie és T = 25 °C, S = 35 PSU, D = 1000 m:

```python
print(round(sound_speed(25, 35, 1000), 3))    # ha d'imprimir exactament 1550.744
```

> ⚠️ Si no obtens `1550.744`, tens una errata a la fórmula. Corregeix-la ara — totes les tasques restants depenen que aquesta funció sigui correcta.

---

## Tasca 4 — Llegeix el perfil CTD d'un fitxer

**Objectiu:** carregar 25 registres reals de profunditat / temperatura / salinitat en tres llistes.  
**Informe**: afegeix una captura del codi i del resum que s'imprimeix.

`ctd_profile.csv` és un perfil CTD d'oceà obert amb forma realista. Les seves tres primeres línies són així:

```
depth_m,temperature_c,salinity_psu
0,24.10,36.52
10,24.05,36.52
```

La primera línia és una **capçalera** — dona nom a les columnes, no són dades, i l'hem d'ometre. `f.readline()` llegeix exactament una línia i avança, que és justament el que necessitem:

```python
depths = []
temperatures = []
salinities = []

with open("ctd_profile.csv", "r") as f:
    header = f.readline()               # llegeix la línia de capçalera i deixa-la de banda
    for line in f:                      # ara el bucle comença a la primera línia de dades
        parts = line.strip().split(____)      #  quin caràcter separa les columnes?
        depths.append(float(parts[0]))
        temperatures.append(float(parts[____]))    #  quina columna és la temperatura?
        salinities.append(float(parts[____]))      #  quina columna és la salinitat?
```

Comprova què has carregat:

```python
print("La capçalera era:", header.strip())
print("Nombre de registres:", ____(depths))       #  com se saben els elements d'una llista?
print("Menys profund:", depths[0], "m ->", temperatures[0], "C,", salinities[0], "PSU")
print("Més profund:", depths[____], "m ->", temperatures[____], "C,", salinities[____], "PSU")
```

Sortida esperada:

```
La capçalera era: depth_m,temperature_c,salinity_psu
Nombre de registres: 25
Menys profund: 0.0 m -> 24.1 C, 36.52 PSU
Més profund: 4000.0 m -> 1.8 C, 34.92 PSU
```

> 📝 Per què necessitem `float()` aquí? Què donaria `depths[0] + depths[1]` si deixéssim els valors com a text? Prova-ho i explica què passa.

---

## Tasca 5 — Calcula tot el perfil i valida les entrades

**Objectiu:** cridar la teva funció un cop per cada profunditat, i no refiar-te de valors fora de l'interval de validesa de l'equació.  
**Informe**: afegeix una captura del codi i dels avisos que imprimeix.

Ja tens una funció i tres llistes. Ara ajunta-ho tot. Com que hem de llegir la mateixa posició `i` de les tres llistes alhora, iterem sobre `range(len(depths))`:

```python
speeds = []

for i in range(len(depths)):
    t = temperatures[i]
    s = salinities[i]
    d = depths[i]

    speeds.append(sound_speed(t, s, d))

print("Calculades", len(speeds), "velocitats del so")
print("A la superfície:", round(speeds[0], 2), "m/s")
```

Ara afegeix-hi la validació. Recorda que Mackenzie només és vàlida per a 2 ≤ T ≤ 30 °C. Insereix això **dins del bucle**, just abans de l'`append`:

```python
    if t < ____ or t > ____:            #  l'interval vàlid de temperatura
        print("AVÍS: la temperatura", t, "C a", d, "m és fora de l'interval vàlid!")
```

Sortida esperada:

```
AVÍS: la temperatura 1.95 C a 3500.0 m és fora de l'interval vàlid (2-30 C)
AVÍS: la temperatura 1.8 C a 4000.0 m és fora de l'interval vàlid (2-30 C)
Calculades 25 velocitats del so
A la superfície: 1533.78 m/s
```

> 📝 Dos registres han activat l'avís. Són errors de mesura, o és aigua real? Busca la temperatura típica de l'Aigua de Fons Antàrtica abans de respondre.

> 📝 Hem imprès un avís però igualment hem calculat un valor per a aquestes dues profunditats. Va ser la decisió correcta? Què més hauríem pogut fer, i què perdríem en cada cas?

---

## Tasca 6 — Desa els resultats i torna'ls a llegir

**Objectiu:** escriure un fitxer CSV nou, i demostrar que ha funcionat tornant-lo a carregar.  
**Informe**: afegeix una captura del codi i de les primeres línies del fitxer de sortida.

Calcular una cosa no serveix de res si desapareix quan acaba l'script. Escriu els resultats:

```python
with open("sound_speed_profile.csv", "____") as f:      #  "r" per llegir, "w" per escriure?
    f.write("depth_m,sound_speed_ms\n")
    for i in range(len(depths)):
        f.write(str(depths[i]) + "," + str(round(speeds[i], 2)) + "____")   #  acaba la línia
```

Obre `sound_speed_profile.csv` al teu IDE. Hauria de començar així:

```
depth_m,sound_speed_ms
0.0,1533.78
10.0,1533.82
20.0,1533.63
```

i acabar així:

```
4000.0,1524.75
```

Ara torna'l a llegir — no et refiïs mai d'un fitxer que no has tornat a obrir:

```python
check_depths = []
check_speeds = []

with open("sound_speed_profile.csv", "r") as f:
    f.readline()                        # omet la capçalera
    for line in f:
        parts = line.strip().split(",")
        check_depths.append(float(parts[0]))
        check_speeds.append(float(parts[1]))

print("Rellegits", len(check_speeds), "valors")
print("El primer valor coincideix:", round(check_speeds[0], 2) == round(speeds[0], 2))
```

> 📝 Què passa si executes l'script dues vegades? El fitxer creix, o es queda igual de gran? Explica què fa `"w"` amb un fitxer que ja existeix.

---

## Tasca 7 — Representa el perfil i troba el canal SOFAR

**Objectiu:** veure la física a les teves pròpies dades.  
**Informe**: inclou la figura i respon les preguntes.

Avui no explicarem com funcionen els gràfics — això és el T03. Simplement copia això i executa-ho.

Dues convencions que val la pena notar: els oceanògrafs posen la **profunditat a l'eix vertical**, i el **giren** perquè el fons quedi a sota, tal com és l'oceà de debò.

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(5, 7))

ax.plot(speeds, depths, marker="o")
ax.invert_yaxis()                       # el fons a baix, com a l'oceà real
ax.set_xlabel("velocitat del so (m/s)")
ax.set_ylabel("profunditat (m)")
ax.set_title("Perfil de velocitat del so")
ax.grid(True)

plt.show()
```

Ara troba el mínim. Python té una funció integrada per fer-ho, i `.index()` et diu *on* es troba un valor dins d'una llista:

```python
c_min = min(speeds)
i_min = speeds.index(c_min)

print("Velocitat del so mínima:", round(c_min, 2), "m/s")
print("Trobada a la profunditat:", depths[i_min], "m")
```

Sortida esperada:

```
Velocitat del so mínima: 1490.46 m/s
Trobada a la profunditat: 1200.0 m
```

> 📝 Inclou la figura a l'informe i marca-hi l'eix del canal SOFAR. A quina profunditat es troba?

> 📝 La velocitat del so a la superfície és 1533.78 m/s i a 4000 m és 1524.75 m/s — pràcticament el mateix valor. Però el perfil entremig no s'assembla gens a una línia recta. Explica'n la forma en termes dels tres efectes que competeixen.

### Bonus

Representa la temperatura i la salinitat al costat de la velocitat del so, i mira quina de les dues en governa la forma:

```python
fig, axes = plt.subplots(1, 3, figsize=(12, 6), sharey=True)

axes[0].plot(temperatures, depths, color="red")
axes[0].set_xlabel("temperatura (C)")
axes[0].set_ylabel("profunditat (m)")

axes[1].plot(salinities, depths, color="green")
axes[1].set_xlabel("salinitat (PSU)")

axes[2].plot(speeds, depths, color="blue")
axes[2].set_xlabel("velocitat del so (m/s)")

axes[0].invert_yaxis()
plt.show()
```

> 📝 Per sobre dels 1000 m, quina de les dues — temperatura o salinitat — explica la corba de velocitat del so? Per sota dels 2000 m, cap de les dues ho fa. Què la governa allà?

---

## Resum

Avui has après a:

- guardar valors en **variables**, i per què a Python li importa el seu **tipus**
- mantenir seqüències de valors en **llistes**, i recórrer-les amb bucles **`for`**
- prendre decisions amb **`if`**, i fer-lo servir per validar dades abans de refiar-te'n
- escriure una **funció** un sol cop i reutilitzar-la, en comptes de copiar i enganxar una fórmula de nou termes
- **llegir** un fitxer de dades real, separar cada línia en columnes i convertir text en números
- **escriure** els teus resultats en un fitxer nou, i verificar-los tornant-los a llegir
- convertir una taula de números en una figura que mostra el **canal SOFAR**

L'script que acabes d'escriure és una peça de programari oceanogràfic completa, encara que petita: ingesta dades, les valida, les processa, desa el resultat i el visualitza. Totes les pràctiques d'ara endavant són una variació d'aquests mateixos cinc passos.
