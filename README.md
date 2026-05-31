# Szolga Tanács

> Egy AI-modellekből álló „tanács", amely több szempontból megvizsgál egy kérdést, megvitatja, majd összefoglalót ad.

A **Szolga Tanács** egyetlen, önálló HTML-fájlból álló webalkalmazás, amely az [OpenRouter](https://openrouter.ai) API-n keresztül egyszerre több AI-modellt — a „Szolgákat" — kérdez meg. A modellek vagy **párhuzamosan** (egymástól függetlenül), vagy **tanács módban** (több fordulóban megvitatva, majd összefoglalva) válaszolnak. Az egészet a felhasználó irányítja „Szolga Hajcsárként": felteszi a kérdést, a tanács dolgozik, ő pedig átnézi az eredményt és dönt.

## Főbb funkciók

- **Párhuzamos és tanács mód** — független válaszok vagy többfordulós, megvitatott konszenzus.
- **Szerepek** — minden Szolga más szemszögből szól hozzá (Architekt, Kritikus, Biztonsági, Költség-Optimalizáló, Ördög Ügyvédje, Összefoglaló stb.).
- **Tanács-emlékezet** — a tanács fejben tartja a korábbi konklúziókat a munkamenetben.
- **Modell-böngésző** — kereshető, ár szerint színezett lista a teljes OpenRouter-kínálattal.
- **Elérhetőség-teszt** — státusz-pöttyök mutatják, mely modellek érhetők el most.
- **Fallback modellek** — Szolgánkénti tartalék-lánc, ha egy modell kiesik.
- **Valós költségmérés** — élő token- és költségkijelzés, fizetős-futás előtti figyelmeztetés.
- **Gyors összeállítás (presetek)** — ingyenes / költséghatékony / egyéni.
- **Dokumentum-csatolás** — több PDF, Word, szöveg és kép referenciaként.
- **Mentés / betöltés** — teljes tanácskozások visszahozhatók, API-hívás nélkül (0 token).
- **Kétnyelvű felület** — magyar és angol, élő váltással.

## Gyors kezdés

1. Nyisd meg a `szolga_tanacs.html` fájlt böngészőben (vagy szolgáld ki statikusan, pl. GitHub Pages).
2. A Beállításoknál add meg az [OpenRouter](https://openrouter.ai) API-kulcsodat.
3. Állítsd be a modelleket (vagy egy kattintással egy presettel), tedd fel a kérdést, és indítsd el.

Részletekért lásd a [felhasználói kézikönyvet](HASZNALAT.md).

## Dokumentáció

- **[HASZNALAT.md](HASZNALAT.md)** — Felhasználói kézikönyv: beállítás, kezelőfelület, funkciók, GYIK.
- **[FEJLESZTOI.md](FEJLESZTOI.md)** — Fejlesztői dokumentáció: architektúra, kódfelépítés, adatmodell, működés.

## Technológia

Egyetlen HTML-fájl, vanilla JavaScript, backend nélkül. Minden a böngészőben fut; a beállítások a `localStorage`-ban tárolódnak. Az AI-hívások az OpenRouter API-n keresztül történnek. A dokumentum-feldolgozáshoz két CDN-könyvtárat használ (pdf.js, mammoth.js), amelyek csak fájl csatolásakor töltődnek be.

## Készítette

Tervezés és koncepció: **Rajczi András**. Implementáció: AI-asszisztenssel.

## Licenc

Minden jog fenntartva. A felhasználási feltételeket lásd a [LICENSE](LICENSE) fájlban.
