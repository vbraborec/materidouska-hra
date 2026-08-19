# Hra Mateřídouška — prototyp v0

Prototyp podle specifikace v0. Cíl: zjistit, jestli se to chce hrát podruhé.
Barevné tvary místo grafiky, žádné menu, žádný zvuk.

**Hrát:** otevřít `index.html` v prohlížeči, nebo publikovanou verzi
(GitHub Pages). Celá hra je jeden soubor bez závislostí.

## Ovládání

| PC | |
|---|---|
| ← → / A D | pohyb |
| Mezerník | aktivace uložené posily |
| P / Esc | pauza |
| Enter | restart po konci hry |

| Mobil | |
|---|---|
| tažení prstem | pohyb (přímé pozicování) |
| krátký tap | aktivace posily / restart |
| tap na ikonu vpravo nahoře | pauza |

Střelba je automatická u obou vstupů.

## Co je ve hře

| Objekt | Tvar / pohyb | Efekt |
|---|---|---|
| dokument | obdélník, rotuje | při srážce **zpomalí střelbu na 5 s** |
| riziko | trojúhelník s vykřičníkem, rotuje | při srážce **bere život** |
| káva | kruh, snáší se vlnivě, pulzuje | uloží se do slotu; aktivace = zpomalení času na 5 s |
| srdíčko | malý kruh, jiskra | okamžitě +100 bodů |

Hrozby jdou sestřelit lístkem, posily a drobnosti ne — těmi projektil prolétne.

### Odchylka od specifikace

§1 uvádí v prototypu jedinou hrozbu (dokument), §5.1 jí ale přiděluje debuff
místo ztráty života. S jedinou hrozbou by hráč neměl jak přijít o život a běh
by nikdy neskončil (proti §6.1). Prototyp proto obsahuje **dvě** hrozby —
dokument (debuff) a riziko (život). Odstupňování zásahů tak jde otestovat.

## Technický základ

- Vanilla JS + Canvas 2D, jeden HTML soubor, žádné závislosti
- Logické rozlišení 600 × 800, pevné, škáluje se na okno (DPR-aware)
- Pevný krok 60 Hz s akumulátorem, render oddělený od logiky
- Seedovaný PRNG (mulberry32). `Math.random()` se v kódu nevyskytuje —
  jediné nedeterministické místo je volba seedu na začátku běhu.
- Jeden stavový objekt `game`
- Vstup abstrahovaný do objektu `input` (`targetX`, `targetY`,
  `activatePressed`, …). Herní logika nesahá na události.
- Skóre se mění výhradně přes `addScore(amount, reason)`

Ověřeno: dva běhy se stejným seedem a stejnými vstupy dávají identický
výsledek (skóre i délku).

### Výsledek běhu

Po konci hry se do konzole vypíše objekt podle §7:

```js
{ score, mode, inputMethod, durationFrames, seed, rulesVersion, inputLog, timestamp }
```

`inputLog` je pole `[frameIndex, actionCode, hodnota?]`.

Historie běhů (bez `inputLog`) se ukládá do `localStorage` — kvůli kritériu
§10/4, tedy porovnání skóre mezi dotykem a klávesnicí. V konzoli:

```js
mdRuns()    // všechny běhy
mdReset()   // vymazat
```

## Přístupnost (§13 — tvrdé omezení)

- Žádné blikání nad 3 cykly/s, žádné záblesky, žádný screen shake
- Nezranitelnost = plynulé prolínání průhlednosti 2 cykly/s
- Pulz posil 1,6 Hz, amplituda jasu do ±20 %
- Zánik objektu = zmenšení a vyblednutí přes ~200 ms, ne exploze
- Pozadí statické
- Barva není nikde jediným nosičem informace: hrozby jsou hranaté a rotují,
  posily kulaté a vlní se, riziko má navíc vykřičník

Před sdílením je potřeba projít §13.4 — nahrát 60 s hraní v nejtěžší fázi
a projít záznam po snímcích.

## Připraveno, ale vypnuté

V kódu jsou už teď datové struktury pro odložené funkce, aby se nemusely
doplňovat zpětně:

- `MODES` — klidný / běžný / náročný (aktivní jen běžný)
- `THREAT_TYPES` — všech 6 kategorií hrozeb z §8, neaktivní mají `enabled:false`
- `POWERUP_TYPES` — všech 5 posil z §8, aktivní je jen káva
- `audio.play(name)` — prázdná abstrakce, volá se na všech 9 místech
- `rulesVersion`, `inputMethod`, `seed`, `inputLog` v záznamu běhu

## Ladění balancování

Všechny číselné hodnoty jsou v objektu `CONFIG` na začátku skriptu.
Křivka obtížnosti (§6.1) je v `CONFIG.difficulty`; za hranicí 4 minut
pokračuje `tail` fáze, aby každý běh skončil.

Naměřeno v aktuálním nastavení: hráč, který stojí uprostřed a nehýbe se,
vydrží ~165 s a nasbírá ~6 700 bodů. První extra život je na 25 000.
Po prvních hraních bude nejspíš potřeba doladit — pak inkrementovat
`CONFIG.rulesVersion`.

## Otázky k ověření na prototypu (§10)

1. Zahrál si někdo podruhé, aniž bys ho o to požádal?
2. Byla nějaká smrt vnímaná jako nefér?
3. Vydrží pozornost 90 sekund?
4. Jak velký je rozdíl ve skóre mezi dotykem a klávesnicí?
