# Szolga Tanács — Fejlesztői dokumentáció

*Műszaki áttekintés a működésről, az architektúráról és a kód felépítéséről.*

Tervezés és koncepció: Rajczi András. Implementáció: AI-asszisztenssel.

---

## Áttekintés

A Szolga Tanács egy **egyetlen, önálló HTML-fájl** (`szolga_tanacs.html`), amely minden logikát, stílust és sablont tartalmaz. Nincs build-lépés, nincs csomagkezelő, nincs szerveroldal — egy statikus fájl, amely közvetlenül megnyitható vagy bárhonnan (pl. GitHub Pages) kiszolgálható.

- **Backend:** nincs. Minden a böngészőben fut.
- **API:** [OpenRouter](https://openrouter.ai) (`/api/v1/chat/completions` és `/api/v1/models`).
- **Tárolás:** kizárólag `localStorage` (kulcs, beállítások, mentések, árcache, nyelv).
- **Külső függőségek:** két CDN-könyvtár, kizárólag dokumentum-feldolgozáshoz (lásd lentebb).

---

## Technológiai stack

- Vanilla JavaScript (nincs keretrendszer), HTML, CSS — minden egy fájlban.
- **pdf.js** `3.11.174` (CDN) — PDF szövegkinyerés.
- **mammoth.js** `1.6.0` (CDN) — Word (.docx) szövegkinyerés.

A CDN-könyvtárak csak akkor terhelődnek, ha ténylegesen dokumentumot/PDF-et dolgozol fel; az alapfunkciók internet nélkül is betöltődnek (de az AI-hívások és az árlista értelemszerűen online-ok).

---

## Magas szintű működés

### Két üzemmód

**Párhuzamos mód** (`legacySend` → `streamOne`)
Minden kártya egymástól függetlenül, párhuzamosan kap egy hívást. Kártyánként megmarad a többfordulós beszélgetési előzmény (`conversations[idx]`), így a párhuzamos kártyák önálló, folytatható szálak. *Ebben a módban szándékosan nincs fallback* — modellt váltani menet közben összekeverné a kártya előzményét.

**Tanács mód** (`runCouncil`)
Több fordulós, megvitatott folyamat:

1. `initMemory(query)` — létrehoz egy munkamenet-állapotot.
2. Fordulónként `buildRound`:
   - 1. forduló: minden Szolga megkapja a kérdést (`buildAgentContext` → `buildSystemPrompt`).
   - N. forduló: minden Szolga megkapja az addigi átiratot (`buildRoundNPrompt` → `buildTranscript`), hogy reagálhasson a többiekre.
   - A hívások párhuzamosan futnak (`streamOneCouncil`), hibatűrően (egy Szolga hibája nem dönti meg a többit).
3. Forduló végén `detectConvergence` — ha a Szolgák már egyetértenek, a tanács korábban leállhat.
4. Végül `generateConsensus` — a kijelölt konszenzus-modell összefoglalót készít (`buildConsensusPrompt`).
5. A konklúzió bekerül a **tanács emlékezetébe** (`councilHistory`), hogy a következő kérdésnél kontextus legyen.

---

## Kód-architektúra (logikai blokkok)

A `<script>` blokk nagyjából a következő szekciókra tagolódik:

| Szekció | Felelősség | Fő függvények |
|---|---|---|
| **I18N** | Nyelvi szótár és fordítás | `I18N`, `t()`, `setLang()`, `applyLang()` |
| **CONSTANTS** | Presetek, szerepek, identitás | `PRESETS`, `ROLES`, `DEFAULT_ROLES`, `baseCouncilIdentity` |
| **CONFIG** | Beállítások betöltése/mentése | `loadCfg`, `saveCfg`, `migrate`, `defaultModels` |
| **STATE** | Globális futásidejű állapot | `cfg`, `conversations`, `rtexts`, `councilMemory`, `councilHistory`, `modelStatus` |
| **GRID / CARDS** | Kártyák, állapotok renderelése | `buildGrid`, `mkCard`, `setCardState`, `mkSummaryCard`, `renderMarkdown` |
| **PROMPTS** | A modelleknek szóló szövegek | `buildSystemPrompt`, `buildAgentContext`, `buildRoundNPrompt`, `buildConsensusPrompt`, `buildCouncilMemoryBlock`, `buildDocBlock` |
| **COST** | Költség- és token-mérés | `CostController`, `loadPricing`, `priceLabel` |
| **MODEL PICKER** | Modell-böngésző | `openModelPicker`, `renderModelPickerList`, `priceColor`, `sortedModels`, `chooseModel` |
| **AVAILABILITY** | Elérhetőség-teszt, hibaosztály | `pingModel`, `testConfiguredModels`, `classifyError`, `modelStatus`, `runPool` |
| **SESSIONS** | Mentés/betöltés | `saveSession`, `restoreSession`, `buildSnapshot`, `loadSessions` |
| **PRESETS** | Gyors összeállítás | `presetFree`, `presetCheap`, `presetCustomApply`, `applyModelIds`, `refreshPresetState` |
| **EXECUTOR** | API-transport | `rawStream` (SSE), `streamOne`, `streamOneCouncil`, `modelChain` |
| **ORCHESTRATOR** | Tanács-vezérlés | `runCouncil`, `buildRound`, `detectConvergence`, `generateConsensus` |
| **DOCUMENTS** | Dokumentum-feldolgozás | `ingestFiles`, `ingestFile`, `extractPdf`, `extractDocx`, `visionTranscribe`, `buildDocBlock` |
| **SETTINGS PANEL** | Beállítások UI | `openPanel`, `renderMCList`, `renderConsensusOptions` |
| **PAID CONFIRM** | Fizetős-figyelmeztetés | `paidModelsInUse`, `confirmPaid`, `resolvePaid` |

---

## Adatmodell

### `cfg` (a fő beállítás-objektum)

```js
{
  key: "sk-or-...",          // OpenRouter API-kulcs
  count: 3,                  // aktív Szolgák száma (1–8)
  models: [                  // mindig 8 elem (a count határozza meg, hány aktív)
    { name, id, role, roleCustom, fallbacks: [] }
  ],
  readerModel: "google/gemini-2.0-flash-001",
  council: {
    maxRounds: 3,
    maxTotalTokens: 50000,
    maxTokensPerCall: 1500,
    convergenceEnabled: true,
    convergenceThreshold: 0.70,
    councilMode: false,
    callTimeoutMs: 90000,
    consensusModel: "auto"   // "auto" vagy egy index (0–7)
  }
}
```

### Szerepek (`ROLES`)

Általános, Architekt, Kritikus, Biztonsági, Rendszergazda, Ördög Ügyvédje, Költség-Optimalizáló, Összefoglaló, Dokumentáció. Mindegyikhez tartozik egy `label` és egy `guidance` (a rendszerpromptba kerülő útmutató). A szerep felülírható egyéni szöveggel (`roleCustom`).

### localStorage-kulcsok

| Kulcs | Tartalom |
|---|---|
| `szolgatanacs-cfg-v4` | A teljes `cfg` objektum |
| `szolgatanacs-pricing` | Az OpenRouter árlista 24 órás cache-e (`{ts, map}`) |
| `szolgatanacs-sessions` | Mentett tanácskozások (max. 30) |
| `szolgatanacs-custom-preset` | Az egyéni preset modelljei |
| `szolgatanacs-lang` | `"hu"` vagy `"en"` |

---

## Az API-transport (`rawStream`)

Egyetlen, közös függvény kezeli az OpenRouter `/chat/completions` hívást, **Server-Sent Events** streamként:

- `stream: true`, `stream_options: { include_usage: true }`.
- A delták érkezésekor `onDelta` callback frissíti a kártyát (élő streamelés).
- A `usage` mezőből pontos token-számot vesz; ha hiányzik, becsül (`content.length / 4`).
- Hiba esetén a `throw`-olt `Error`-ra ráteszi a `httpStatus`-t és `apiCode`-ot — ezek alapján osztályoz a `classifyError`.

### Hibaosztályozás (`classifyError`)

| Visszatérés | Mikor | Viselkedés |
|---|---|---|
| `'credit'` | 402, vagy „insufficient credits" stb. | **Megáll**, nem vált fallbackre (ne költsön akaratlanul) |
| `'avail'` | 404, 429, 5xx, „no endpoints", „rate limit", timeout stb. | **Fallback** a láncban |
| `'other'` | minden más | Nem fallback-elhető, jelez |

### Fallback-lánc (`modelChain`)

Szolgánként: `[elsődleges, ...fallbacks]` (deduplikálva). A `streamOneCouncil` és a `generateConsensus` végigpróbálja a láncot: `'avail'` hibánál a következőre lép (a kártyán „↪ tartalék: X" jelzéssel), `'credit'` hibánál megáll. A párhuzamos mód (`streamOne`) **nem** használ fallbacket (előzmény-integritás).

---

## Konvergencia-érzékelés (`detectConvergence`)

A 2. fordulótól számol egy 0–1 közötti pontszámot három jelből:

1. **Egyetértési nyelv aránya (40%)** — hány válasz tartalmaz egyetértést jelző kifejezést (pl. „egyetértek", „agree", „as noted").
2. **Válaszhossz-csökkenés (30%)** — ha a válaszok rövidülnek az előző fordulóhoz képest (a leggyengébb jel — lásd „Ismert korlátok").
3. **Explicit konszenzus-deklaráció (30%)** — van-e olyan válasz, ami kimondja, hogy megállapodás született (pl. „consensus", „unanimous").

Ha a pontszám eléri a küszöböt (`convergenceThreshold`), a tanács a forduló után leáll.

---

## Költség-mérés (`CostController`)

- A `loadPricing` letölti az OpenRouter `/models` árlistáját, és 24 órára `localStorage`-ba cache-eli. Friss cache esetén **nem** kér újra hálózatról.
- Minden befejezett hívás `CostController.record(tokens, modelId)`-dal könyvel.
- Ingyenes (`$0`) modellnél a költség 0; ismert árnál pontos „$", ismeretlennél „≈" becslés.
- A `isExhausted` / `canAffordRound` őrzi a token-keretet: új forduló csak akkor indul, ha belefér.

---

## Elérhetőség-teszt (`testConfiguredModels`)

- Lepingeli a beállított modelleket **és** az összes ingyenes modellt (1 tokenes hívás), korlátozott párhuzamossággal (`runPool`), hogy ne ütközzön a rate limitbe.
- Az eredmény `modelStatus`-ba kerül (`ok` / `down` / `limited` / `testing`), és státusz-pöttyként jelenik meg a kártyákon és a böngészőben.
- A tényleges futások is frissítik a státuszt (siker → `ok`, provider-hiba → `down`).

---

## Dokumentum-feldolgozás

- `ingestFiles` → `ingestFile` fájltípusonként: szöveg (közvetlen), `.docx` (mammoth), PDF (pdf.js), kép/szkennelt PDF (vízió-modell, `visionTranscribe`).
- Limitek: `DOC_CHAR_BUDGET = 12000` / dokumentum, `DOC_TOTAL_BUDGET = 30000` összesen, `MAX_DOCS = 8`.
- A `buildDocBlock` egyetlen, sorszámozott blokkba fűzi a kész dokumentumokat, amit a promptokba injektál; több fájlnál összehasonlításra ösztönzi a Szolgákat.

---

## Internacionalizáció (i18n)

- `I18N` egy kulcsolt szótár; minden bejegyzés `{hu, en}` (string vagy interpoláló függvény).
- `t(key, ...args)` az aktuális `lang` szerint old fel.
- A statikus HTML a `data-i18n`, `data-i18n-ph` (placeholder), `data-i18n-title`, `data-i18n-aria`, `data-i18n-optg` attribútumokkal van megjelölve; az `applyLang()` ezeket bejárja és a dinamikus régiókat is újrarendereli.
- A nyelv `setLang()`-gal vált, `localStorage`-ba ment, és beállítja a `<html lang>`-et.
- **Szándékosan nem fordított:** az AI-nak szóló promptok (tanács-memória blokk, konszenzus-utasítás, `X-Title` fejléc) — ezek a modelleknek szóló instrukciók, nem UI.

---

## Mentés/betöltés (`buildSnapshot` / `restoreSession`)

A pillanatkép tartalmazza a módot, a modelleket (szereppel és fallbackkel), a kártyák végső szövegét, a kérdést, a konszenzust és a tanács-emlékezetet. A visszatöltés tisztán lokális: újraépíti a rácsot, visszafesti a szövegeket és az összefoglalót — **API-hívás nélkül** (0 token).

---

## A fájl szerkesztése / fejlesztés

Mivel minden egyetlen fájlban van, a fejlesztés egyszerű, de figyelni kell pár dologra:

- **Validáció:** a `<script>` blokk kiemelhető és `node --check`-kel ellenőrizhető; a `<div>` nyitó/záró egyensúly gyors épségvizsgálatot ad.
- **Tesztelés:** headless böngészővel (pl. Playwright) érdemes — a `window.fetch` mockolható az API-válaszokhoz, mivel a sandbox jellemzően nem éri el az OpenRoutert élesben.
- **Új UI-szöveg:** mindig vegyél fel `I18N`-bejegyzést és használj `t()`-t vagy `data-i18n`-t — soha ne égess be magyar/angol stringet.
- **Új modell-funkció:** a `cfg.models[i]` objektumot bővítsd, és gondoskodj a `migrate`, `buildSnapshot`/`restoreSession` és a `renderMCList` összhangjáról.

---

## Ismert korlátok / tudatos kompromisszumok

- A **konvergencia hossz-jele** (2. jel) a leggyengébb heurisztika: ha a Szolgák a konvergencia felé nem rövidülnek (néha épp bővülnek), félrevihet. Tudatos kompromisszum; finomítás esetén ez az első jelölt.
- A **párhuzamos módban nincs fallback** (előzmény-integritás miatt) — ez szándékos.
- A **böngésző-tárolás nem titkosított**: az API-kulcs a `localStorage`-ban olvasható, ha valaki hozzáfér a böngészőprofilhoz. Érzékenyebb környezetben érdemes `sessionStorage`-ra váltani.
- A presetek és a böngésző az **élő árlistára** támaszkodnak — offline állapotban korlátozottak.

---

## Biztonsági megjegyzések

- Az API-kulcs sosem hagyja el a böngészőt a hívások fejlécén kívül.
- A `:free` modellek a fizetős fiókon is ingyenesek, de napi/perces limittel és változó elérhetőséggel.
- A fizetős-figyelmeztetés (`confirmPaid`) az élő árlista alapján dönt; ismeretlen árú modellt nem nyilvánít fizetősnek (nem blokkol feleslegesen).
