# Szolga Tanács — Felhasználói kézikönyv

*Egy AI-modellekből álló „tanács", amely több szempontból megvizsgál egy kérdést, megvitatja, majd összefoglalót ad.*

Tervezés és koncepció: Rajczi András. Implementáció: AI-asszisztenssel.

---

## Mi ez?

A Szolga Tanács egyetlen HTML-fájlból álló webalkalmazás, amely az [OpenRouter](https://openrouter.ai) API-n keresztül egyszerre több AI-modellt (a „Szolgákat") kérdez meg. Két üzemmódban dolgozhat:

- **Párhuzamos mód:** minden modell egymástól függetlenül, egyszerre válaszol ugyanarra a kérdésre. Gyors, jó több vélemény gyors összevetésére.
- **Tanács mód:** a Szolgák több fordulóban *megvitatják* a kérdést — látják egymás válaszait, reagálnak rá —, végül egy kijelölt modell **összefoglalót** készít a megállapodásokról, a nézeteltérésekről és egy ajánlott irányról.

Az egészet te irányítod „Szolga Hajcsárként": felteszed a kérdést, a tanács dolgozik, te pedig átnézed és döntesz.

---

## Mire jó?

Bármilyen kérdésre, aminek **több, egymással versengő szempontja** van: IT-üzemeltetési dilemmák, „on-premise vagy felhő", beszerzési döntések, fejlesztési irányok, kockázatok feltárása. Minél több oldala van egy kérdésnek, annál többet ad a tanács.

Egyszerű ténykérdésre (pl. „mi egy város fővárosa") felesleges — ott a sok modell csak ugyanazt mondaná.

---

## Első lépések

### 1. OpenRouter API-kulcs

Az alkalmazás a saját OpenRouter-fiókod kulcsával működik:

1. Hozz létre fiókot az [openrouter.ai](https://openrouter.ai) oldalon.
2. Generálj egy API-kulcsot (Keys menü).
3. Indítsd el a Szolga Tanácsot, nyisd meg a **Beállításokat** (fogaskerék ikon), és illeszd be a kulcsot az „OpenRouter API Kulcs" mezőbe.

A kulcs **csak a saját böngésződben tárolódik** (localStorage), nem kerül sehova máshova.

### 2. Ingyenes vagy fizetős?

Fontos tudnivaló az OpenRouterről:

- A **`:free` végződésű** modellek ingyenesek (pl. `deepseek/deepseek-chat-v3-0324:free`). Ezeknek napi korlátja van, és időnként kieshetnek vagy túlterhelődhetnek.
- Minden más modell **fizetős** — köztük az összes OpenAI (GPT), Claude, Grok, Mistral Large stb. Ezekhez egyenleg kell a fiókodon.
- Egy kis egyenleg feltöltése (pl. néhány dollár) feloldja a magasabb napi ingyenes keretet **és** stabilabb, fizetős modelleket is elérhetővé tesz. Egy teljes tanácskozás jellemzően néhány *cent*.

Az alkalmazás **mindig figyelmeztet**, mielőtt fizetős modellt futtatnál (lásd lentebb).

### 3. Nyelv

A felület magyar és angol nyelvű. A Beállítások tetején válthatsz a **Magyar / English** között — azonnal, és a választás megmarad.

---

## A felület

### Felső sáv (fejléc)

- **Könyvjelző ikon** — Mentett tanácskozások (mentés / betöltés).
- **Kuka ikon** — Az aktuális munkamenet törlése (új lap).
- **Fogaskerék ikon** — Beállítások.
- **Forduló-jelvény** — tanács módban mutatja, hányadik fordulónál tart (pl. „Forduló 2/3").

### Lekérdezés-mező

Ide írod a kérdést. Alatta:

- **Gemkapocs** — dokumentum csatolása (lásd lentebb).
- **Párhuzamos / Tanács** kapcsoló — üzemmód.
- **Elküldés** (vagy `Ctrl+Enter`).
- Futás közben: **Leállítás** és **Forduló után megáll** gombok.

### Válaszok

Minden Szolga külön kártyán jelenik meg: a modell neve, az ID-ja, a szerepe (tanács módban), a streamelő válasz, és lent a karakterszám + állapot. Tanács módban a fordulók egymás alatt jelennek meg, a végén pedig külön **Összefoglaló** kártya.

### Alsó állapotsáv

- **Állapot** (Készen áll / Fut / Kész).
- **Token és költség** — valós idejű becsült/pontos költség és a felhasznált token a keretből.
- **Konvergencia** — tanács módban, mennyire értenek egyet a Szolgák (százalék).
- **Tanács emlékezet** — hány korábbi témakört tart fejben a tanács (kattintásra üríthető).

---

## Beállítások részletesen

### Modellek száma

Csúszkával 1–8 Szolga állítható be.

### Gyors összeállítás (presetek)

Három gombbal egy kattintással feltöltheted a modelleket:

- **Ingyenes** — csak `$0` modellekkel tölt fel. Zöld csíkot kap, ha van elég ingyenes modell a beállított Szolga-számhoz; ambert, ha nincs elég (és megírja, mennyi hiányzik).
- **Költséghatékony** — a legolcsóbb fizetős modellekkel tölt fel.
- **Egyéni** — a saját, korábban elmentett összeállításodat tölti be. A „Jelenlegi mentése egyéniként" gombbal rögzítheted az aktuális modelleket.

### Modell konfiguráció

Minden Szolgához:

- **Megjelenítési név** — szabadon átírható (pl. „Architekt GPT").
- **Model ID** — a pontos OpenRouter-azonosító. Beírhatod kézzel, vagy a **Modellek böngészése** gombbal kiválaszthatod egy kereshető listából.
- **Szerep a tanácsban** — pl. Architekt, Kritikus, Biztonsági, Költség-Optimalizáló, Ördög Ügyvédje, Összefoglaló, Dokumentáció. A szerep határozza meg, milyen szemszögből szól hozzá az adott Szolga.
- **Egyéni szerep** — saját szerep-leírás, ami felülírja a fentit.
- **Fallback (tartalék) modellek** — vesszővel elválasztva. Ha az elsődleges modell kiesik, az alkalmazás automatikusan ezekre vált (lásd lentebb).

### Modell-böngésző

A „Modellek böngészése" gomb megnyit egy kereshető ablakot a teljes OpenRouter-kínálattal:

- Legolcsóbbtól a legdrágábbig rendezve.
- Bal oldalt egy **színsáv** jelzi az árszintet (zöld = olcsó/ingyenes, piros = drága).
- **Mind / Ingyenes / Fizetős** szűrők.
- Egy státusz-pötty jelzi, hogy a modell elérhető-e (lásd „Modellek tesztelése").
- Egy sorra kattintva beíródik a modell a kártyába.

### Modellek tesztelése

A „Modellek tesztelése" gomb egy aprócska (1 tokenes) hívással leellenőrzi a beállított modelleket, valamint az összes ingyenes modellt. Az eredményt **státusz-pöttyök** mutatják:

- 🟢 zöld — elérhető
- 🔴 piros — nem elérhető (pl. megszűnt modell, hiba)
- 🟡 amber — rate limit (átmeneti korlát, később próbáld)

A pöttyök a tényleges futásokból is frissülnek: ami lefutott, zöld lesz; ami hibázott, piros.

### Dokumentum-olvasás

Az „Olvasó modell" a képek és szkennelt PDF-ek átírására szolgál (ehhez egy vízió-képes modell kell, alapból a `google/gemini-2.0-flash-001`).

### Tanács beállítások

- **Tanács mód** — ki/be (ugyanaz, mint a lekérdezés-mezőnél).
- **Maximum fordulók** — hány körben vitázzanak (1–5).
- **Token limit (összesen)** — a teljes tanácskozás token-kerete (alap: 50 000).
- **Max token / hívás** — egy-egy válasz hossza (alap: 1500).
- **Konvergencia érzékelés** — ha bekapcsolt, a tanács korábban leáll, ha a Szolgák már egyetértenek.
- **Konvergencia küszöb** — milyen egyetértésnél álljon le (alap: 0.70).
- **Konszenzus modell** — melyik modell készítse az összefoglalót. „Automatikus" esetén az Összefoglaló szerepű (vagy az első) Szolga.

---

## Dokumentumok csatolása

A gemkapocs ikonnal **több fájlt** is csatolhatsz (max. 8): PDF, Word (.docx), szövegfájlok, és képek. A kép és a szkennelt PDF tartalmát az olvasó modell írja át szöveggé. Minden dokumentum külön „chipen" jelenik meg, és egyenként eltávolítható.

A csatolt dokumentumok bekerülnek a Szolgák elé referenciaként — több fájl esetén az alkalmazás kéri őket, hogy hasonlítsák is össze (pl. három árajánlat). Hosszúság-korlátok: dokumentumonként kb. 12 000 karakter, összesen kb. 30 000 — ha túllépnéd, a chip „vágva" jelzést mutat.

---

## Fizetős futás — figyelmeztetés

Mielőtt elindítanál egy futást, ha **bármelyik résztvevő modell fizetős**, felugrik egy ablak („Vigyázz, báttya!"), felsorolja a fizetős modelleket az áraikkal, és csak a **Persze!** gombra indul el. A **Dehogy!** megszakítja. Ha minden modell ingyenes, nincs ilyen ablak — egyből indul.

Ez azért fontos, hogy soha ne költs akaratlanul: könnyű azt hinni, hogy ingyenes modelleket használsz, miközben egy fizetős van köztük.

---

## Tartalék (fallback) modellek

Az ingyenes modellek időnként kiesnek. Ha minden Szolgának adsz 1–2 tartalék modellt (a kártyán, vesszővel elválasztva), akkor:

- Ha az elsődleges modell **kiesik** (provider-hiba, „nincs endpoint", rate limit, időtúllépés), a tanács automatikusan a tartalékra vált — a kártyán egy „↪ tartalék: …" jelzéssel. Mivel a tartalékok Szolgánként eltérőek lehetnek, a tanács sokszínűsége megmarad.
- Ha a hiba **egyenleg-probléma** (fizetős modell, nulla egyenleg), a tanács **megáll és jelzi** — nem vált automatikusan másik (esetleg szintén fizetős) modellre, hogy ne költsön akaratlanul.

Ez érvényes a Szolgákra és a konszenzus-modellre is.

---

## Mentés és betöltés (0 token)

A könyvjelző ikonnal megnyithatod a **Mentett tanácskozások** ablakot:

- A „Jelenlegi tanácskozás mentése" elmenti a teljes állapotot: a modelleket, a válaszokat, a kérdést és az összefoglalót.
- A „Betöltés" gombbal bármelyik mentést **visszahozhatod** — kitölti a modelleket, visszarakja a válaszokat és az összefoglalót —, **API-hívás és token-költés nélkül**. Így nem kell újra lefuttatni ugyanazt.
- Egy mentés a böngésződben tárolódik (max. 30), és egyenként törölhető.

A betöltött állapot egy *pillanatkép*: a régi válaszokat hozza vissza. Ha ebből új kérdést küldesz, az már friss lekérdezés lesz.

---

## Tipikus munkafolyamat

1. Beállítod a modelleket (vagy egy presettel egy kattintással), és bekapcsolod a tanács módot.
2. Felteszed a kérdést, szükség esetén dokumentumot csatolsz.
3. A tanács több fordulóban megvitatja, majd összefoglalót ad.
4. Átnézed az összefoglalót, és aszerint döntesz vagy cselekszel — a végső ítélet mindig a tiéd.

---

## Gyakori kérdések

**Miért kapok „Insufficient credits" hibát ingyenes modelleknél is?**
Ha a fiók egyenlege nullára vagy mínuszba kerül, az OpenRouter az ingyenes modelleket is visszautasíthatja. Tölts fel egy kis egyenleget.

**Miért írja egy modell, hogy „No endpoints found"?**
Az adott model ID-t az OpenRouter már nem szolgálja ki (megszűnt vagy átnevezték). Cseréld le egy létezőre a böngészővel.

**Miért lett egyszerre az összes Szolga piros?**
Jellemzően egyenleg-probléma (mínuszba ment a fiók), vagy az összes modell fizetős és nincs fedezet. Nézd meg az OpenRouter Credits oldaladat.

**Kell-e internet?**
Igen: az AI-hívásokhoz, az árlista letöltéséhez és (kép/PDF csatolásnál) a feldolgozó könyvtárakhoz.

---

## Adatvédelem

Az API-kulcs és a beállítások **csak a böngésződ helyi tárolójában** vannak. A kérdéseid és a csatolt dokumentumok az OpenRouteren keresztül jutnak el a választott modellekhez — érzékeny adatoknál ezt vedd figyelembe.
