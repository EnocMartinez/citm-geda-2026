# GeDa Pràctica 3 — Automatització i APIs amb un bot de Telegram

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

Al T04 vas demanar dades a un servidor amb una petició **GET**. Avui ho faràs
al revés: el teu script **enviarà** coses — text i imatges — a una aplicació
de missatgeria real, i després **llegirà** què li respon una persona i actuarà
en conseqüència.

Al final de la sessió tindràs un bot de Telegram que funciona: respon
`turtle`, `shark` o `octopus` amb la foto corresponent, i diu alguna cosa
sensata quan escrius qualsevol altra cosa.

### Com funciona aquesta pràctica

Quatre tasques, cadascuna construïda sobre l'anterior. **El mateix fitxer va
creixent tota la sessió** — no comencis un script nou per cada tasca.

Cada tasca té dues parts:

> **Com funciona** — la mecànica, amb un exemple executable. No sempre és la
> resposta sencera.
>
> **Ara et toca** — què has d'escriure tu.

El símbol 📝 marca una pregunta que has de respondre a l'informe. L'informe és
curt expressament: avui el lliurament és **el codi**, no la prosa.

### Objectius

* Parlar amb una API REST (Telegram) amb la llibreria `requests`
* Enviar peticions **POST** amb text i amb fitxers
* Llegir una resposta en JSON i treure'n un valor d'una estructura imbricada
* Mantenir una credencial fora del codi font
* Escriure un programa que **no s'atura** — un bucle `while` que escolta i reacciona

### Què cal lliurar

Lliura a l'Atenea:

* **`LAB3.py`** — l'script final, les quatre tasques en un únic fitxer, que es
  pugui executar de dalt a baix.
* Un informe curt en PDF: una captura del teu xat de Telegram amb el bot
  funcionant, les respostes a les preguntes 📝, i una secció `Ús d'IA` dient
  quines eines d'IA has fet servir i per a què.

> ⚠️ **No lliuris el `secrets.json`, i no enganxis el teu token a l'informe.**
> Qui tingui el teu token controla el teu bot. Si se't filtra, obre el
> BotFather i fes servir `/revoke`.

---

## Preparació

### 1. Les llibreries

Amb l'entorn virtual del teu projecte activat:

```bash
pip install requests rich
```

`requests` parla amb l'API. La `rich` hi és per un sol motiu: avui miraràs
diccionaris molt imbricats, i el `print()` pelat te'ls aboca en una única
línia illegible. `rich.print()` imprimeix el mateix diccionari indentat i
acolorit, i així en veus la forma:

```python
import rich

rich.print({"ok": True, "result": [{"id": 1, "who": {"name": "Enoc"}}]})
```

En aquesta pràctica fes servir `rich.print()` en lloc de `print()` cada vegada
que estiguis inspeccionant una resposta.

### 2. Crea el teu bot

Obre el Telegram i parla amb el **[@BotFather](https://t.me/BotFather)**:

1. Envia-li `/newbot`
2. Posa-li un nom (el que vulguis) i un nom d'usuari (ha d'acabar en `bot`)
3. El BotFather et respon amb un **token** amb aquesta pinta:
   `8123456789:AAFk0qR_exampleTokenDoNotUseThisOne`

### 3. Aconsegueix el teu chat id

Un bot no pot escriure a algú que no li hagi escrit abans, i necessita saber
**on** ha d'escriure. Parla amb el **[@myidbot](https://t.me/myidbot)**:
envia-li `/start` i després `/getid`. Et respon amb un número — aquest és el
teu `chat_id`.

Ara obre el xat amb **el teu propi bot** i envia-li `/start`. Sense això, el
bot no té permís per enviar-te missatges.

### 4. El fitxer de credencials

Mai escriguis un token dins del codi. Crea un fitxer `secrets.json` **al
costat del teu script**:

```json
{
  "token": "8123456789:AAFk0qR_exampleTokenDoNotUseThisOne",
  "chat_id": 123456789
}
```

I llegeix-lo al principi del `LAB3.py` — ja saps llegir JSON des del LAB1:

```python
import json
import requests
import rich

with open("secrets.json") as f:
    secrets = json.load(f)

TOKEN = secrets["token"]
CHAT_ID = secrets["chat_id"]
URL = f"https://api.telegram.org/bot{TOKEN}"
```

`URL` és l'adreça base del **teu** bot. Tots els mètodes de Telegram en
pengen: `{URL}/sendMessage`, `{URL}/sendPhoto`, `{URL}/getUpdates`.

> ⚠️ **El `python3` no és un mag.** `open("secrets.json")` busca aquest fitxer
> a la carpeta des d'on executes l'script, i en cap altre lloc. No et rastreja
> el disc per trobar-lo. Si el `secrets.json` encara és a la carpeta de
> baixades i el teu script és a `GeDa/LAB3`, obtindràs
>
> ```
> FileNotFoundError: [Errno 2] No such file or directory: 'secrets.json'
> ```
>
> El fitxer existeix — el que passa és que Python no té ni idea d'on l'has
> posat. La mateixa regla te la tornaràs a trobar a la Tasca 2 amb el
> `turtle.jpg`. **Mantén l'script i tots els fitxers que obre en una mateixa
> carpeta, i executa l'script des d'aquella carpeta.**

> 📝 Acabes de posar un secret en un fitxer a part en comptes de dins del
> codi, i l'informe tampoc no l'ha de contenir. Explica què aniria malament si
> pugessis el `secrets.json` a un repositori públic de GitHub, i digues què
> hauries de fer a continuació.

> **No tens compte de Telegram?** Demana-ho — hi ha un bot compartit de classe
> perquè puguis fer tota la pràctica sense haver-te'n de crear un.

---

## Tasca 1 — Hola món

### Com funciona

Una petició té tres parts: **on** l'envies, **què** envies, i **què torna**.

```python
response = requests.post(
    f"{URL}/sendMessage",                              # on: l'endpoint
    data={"chat_id": CHAT_ID, "text": "Hello world"}   # què: els paràmetres
)

print(response.status_code)   # què torna: 200 vol dir que ha anat bé
rich.print(response.json())   # la resposta sencera, com a diccionari
```

`data` és un diccionari de Python normal. La `requests` te'l converteix en el
cos del POST — és la mateixa traducció de diccionari a *query string* que vas
veure al T04, però ara viatja dins d'un POST en lloc d'anar a la URL.

Executa-ho. El missatge apareix al teu xat, el terminal imprimeix `200`, i el
`rich.print` et mostra què ha respost Telegram: un diccionari que descriu el
missatge que acabes d'enviar. Llegeix-te'l bé abans de continuar — és una
versió petita de l'estructura on hauràs d'excavar a la Tasca 3.

### Ara et toca

**1a.** Envia't un `Hello world` a tu mateix.

**1b.** Embolcalla-ho en una funció per deixar de repetir-te:

```python
def send_message(text):
    ...
```

Rep el text, l'envia, i retorna `True` si la petició ha funcionat i `False` si
no. **Decideix-ho a partir del codi d'estat** — no donis per fet que ha anat
bé.

---

## Tasca 2 — Enviar una foto

### Com funciona

El text viatja dins de `data`. Un fitxer, no: viatja dins de `files`, i s'ha
d'obrir en mode **binari** (`"rb"`), perquè un JPEG no és text:

```python
with open("turtle.jpg", "rb") as f:
    response = requests.post(
        f"{URL}/sendPhoto",
        data={"chat_id": CHAT_ID},     # els paràmetres normals
        files={"photo": f}             # el fitxer en si
    )

print(response.status_code)
```

La clau de dins de `files` no te la pots inventar: el mètode `sendPhoto` de
Telegram espera que es digui exactament `photo`.

### Ara et toca

**2a.** Descarrega tres fotos — una tortuga, un tauró i un pop — i desa-les
**a la mateixa carpeta que l'script**, amb aquests noms exactes:

```
turtle.jpg    shark.jpg    octopus.jpg
```

(Wikimedia Commons és una bona font. Qualsevol JPEG serveix.) El navegador te
les haurà deixat a la carpeta de baixades — **mou-les al costat de
l'script**, o l'`open()` no les trobarà. El Python continua sense ser un mag.

**2b.** Envia `turtle.jpg` al teu xat.

**2c.** Escriu una segona funció:

```python
def send_photo(filename):
    ...
```

Amb el mateix contracte que `send_message`: retorna `True` o `False` segons el
codi d'estat.

**2d.** Crida-la tres vegades seguides, una per animal.

---

## Tasca 3 — Llegir l'últim missatge

Fins ara només has parlat. Ara toca escoltar.

### Com funciona

`getUpdates` és un **GET** — estàs demanant, no enviant — i retorna tot el que
la gent ha escrit al teu bot darrerament:

```python
response = requests.get(f"{URL}/getUpdates")
data = response.json()
rich.print(data)
```

Envia primer **tres missatges diferents** al teu bot des del Telegram, i
després executa-ho.

El que torna és la part interessant d'aquesta pràctica, i no té res a veure
amb Telegram: és una **estructura que no has dissenyat tu**. Diccionaris dins
de llistes dins de diccionaris, uns quants nivells avall, i la major part no
t'interessa. En algun lloc d'allà dins hi ha el text que acabes d'escriure, i
la teva feina és trobar el camí que hi porta.

Ningú no et donarà aquest camí. Deriva'l de la teva pròpia sortida:

* Comença pel nivell més extern. Quines claus té? Quina conté la part que
  t'interessa, i quines són només paperassa?
* Baixa un nivell i torna't a fer la mateixa pregunta.
* A cada pas, sàpigues **sobre què estàs trepitjant**: un diccionari, al qual
  entres amb una clau, o una llista, a la qual entres amb un índex.
* `type(x)` et diu quin dels dos és, i `x.keys()` et diu què conté un
  diccionari. Fes-los servir.
* Atura't quan tinguis a les mans el text que havies escrit.

Enviar tres missatges *diferents* abans d'executar això fa que la forma es vegi
molt més clara: la part que es repeteix tres vegades és la que conté un
missatge, i tot el que l'envolta és l'embolcall.

### Ara et toca

**3a.** Fes `rich.print` de la resposta i troba-hi els teus tres missatges.
Apunta el camí fins al text d'un d'ells.

**3b.** Escriu una funció que retorni el text del missatge **més recent**:

```python
def get_last_message():
    ...
```

Dues coses que has de clavar:

* Telegram et dona uns quants missatges recents, del més antic al més nou. Tu
  vols el més nou.
* Si ningú no ha escrit al teu bot, aquella col·lecció és **buida**, i entrar-hi
  llança un `IndexError`. Comprova-ho abans d'indexar, i retorna `None`.

**3c.** Envia un missatge al teu bot des del Telegram, executa la teva funció i
imprimeix què retorna.

> 📝 Escriu el camí sencer des del diccionari més extern fins al text del
> missatge, i digues què és cada pas: diccionari o llista, i què significa
> aquella clau o aquell índex. És el camí que has derivat tu a 3a — no un que
> t'hagin donat.

---

## Tasca 4 — El bot sencer

Ara el bot s'executa sol i et respon.

### Com funciona

Un bucle `while True` no s'acaba mai per si sol. A dins, comproves, actues, i
**esperes** abans de tornar a comprovar — sense el `sleep` bombardejaries els
servidors de Telegram centenars de vegades per segon i et bloquejarien:

```python
import time

while True:
    text = get_last_message()
    print("last message:", text)
    time.sleep(2)          # espera 2 segons abans de tornar a preguntar
```

Atura'l amb **Ctrl+C**.

### Ara et toca

**4a.** Executa exactament el bucle de dalt i mira el terminal mentre envies un
missatge des del Telegram.

**4b.** Tens un problema. `get_last_message()` retorna l'últim missatge cada
vegada, així que el mateix missatge es processa un cop i un altre — el teu bot
t'enviaria la tortuga fins a la fi dels temps.

La solució: cada *update* porta un `update_id`, que és un **número que només
creix**. Recorda l'últim que has processat, i actua només quan n'arribi un
de diferent.

Canvia la funció perquè retorni **tots dos** valors:

```python
def get_last_message():
    """Returns (update_id, text) of the latest message, or (None, None)."""
    ...
```

i fes-ne el seguiment dins del bucle:

```python
last_id = None

while True:
    update_id, text = get_last_message()

    if update_id is not None and update_id != last_id:
        last_id = update_id
        print("new message:", text)

    time.sleep(2)
```

**4c.** Ara respon. Dins de l'`if`, decideix què fer amb `text` amb una cadena
`if` / `elif` / `else`:

* `turtle` → envia `turtle.jpg`
* `shark` → envia `shark.jpg`
* `octopus` → envia `octopus.jpg`
* qualsevol altra cosa → envia el missatge de text `Picture not available`

Fes servir les dues funcions que ja has escrit a les tasques 1 i 2. Si aquí
estàs tornant a escriure `requests.post`, torna enrere i fes servir
`send_photo`.

**4d.** Dos detalls que el fan usable:

* `Turtle`, `TURTLE` i ` turtle ` han de funcionar tots tres. Una línia ho
  arregla: `text = text.strip().lower()`.
* **No tots els *updates* tenen text.** Un adhesiu és un missatge sense clau
  `text`; i si *edites* un missatge que ja havies enviat, Telegram no envia ni
  tan sols un `message` — envia un `edited_message`. Qualsevol dels dos casos
  fa petar `update["message"]["text"]` amb un `KeyError`. El `.get()` retorna
  `None` en comptes de llançar l'excepció, i així queden coberts tots dos:

  ```python
  text = update.get("message", {}).get("text")
  ```

  Després, ignora l'*update* quan `text` sigui `None`.

---

## Resum

Avui has après a:

- enviar una petició **POST** a una API REST real, amb paràmetres i amb un fitxer
- llegir un **codi d'estat** i fer-lo servir per decidir si alguna cosa ha anat bé
- mantenir una **credencial fora del codi font**, i fora del control de versions
- navegar per una **estructura JSON imbricada** que no has dissenyat tu, fins a
  arribar a un valor concret
- escriure un programa que **no s'atura**, que recorda què ja ha processat, i
  que es ramifica segons el que digui l'usuari
