---
name: country-map-game
description: Generate a new "place cities, rivers and landmarks on the map" game for a country of the fun-games app. Input is only the country name and, optionally, the language of the legend (default French). Use when the user asks to add a country, a new map game, or "the same game as France for <country>".
argument-hint: "<country> [legend language, default French]"
---

# Country map game

Creates the drag-and-drop map game (game 3, built for France) for another country: a map of the country,
about 30 cities, 50 landmarks, 30 culinary specialties and up to 30 rivers, each with a fun fact, all in the legend language.

**Input:** `$ARGUMENTS` = the country name, optionally followed by the legend language
(e.g. `Italy`, `Italie`, `Japan Spanish`, `Portugal --lang pt`). No language means **French** (`fr`).
If the country is missing, ask for it. Nothing else is needed from the user.

The generic game is already in the repo (`src/routes/CountryGame.svelte`, `src/lib/games/places.ts`).
A country is only data: `src/lib/countries/<id>/` (`map.ts` generated, `content.ts` written, `meta.ts`, `index.ts`)
plus one line in `src/lib/countries/registry.ts`. France (`src/lib/countries/france/`) is the reference example.
Read `content-guide.md` (next to this file) before writing any content.

**Counts are targets, not requirements.** The goals (30 cities, 50 landmarks, 30 specialties, 30 rivers) are not always reachable: the
map data may have few rivers, a small country may have few notable landmarks, a web fetch may fail, a fact may not be
verifiable. **Never stop or ask the user because a target is missed: carry on and build the country with what you have.**
Do not pad to hit a number with doubtful or invented items; fewer verified ones are better. The only hard minimum is
what makes the game playable (at least 8 cities, enforced by the tests). Say what fell short in the final report.

## Steps

1. **Language code.** Turn the language into an ISO 639-1 code (`fr`, `en`, `es`, `de`, `it`, `pt`…). Default `fr`.
   `fr` and `en` have ready-made interface texts; any other language needs them translated (step 5).

2. **Generate the map.** Run (from the repo root; the first run downloads ~25 MB of Natural Earth data into `.cache/`):
   ```sh
   node scripts/country/build-map.mjs --country "<country>" --lang <code>
   ```
   **Not a country?** (an island, region or territory such as Tahiti, Corsica, Hawaii): never stop to ask. Find the Natural Earth
   country it belongs to (Tahiti → French Polynesia, Corsica → France) and zoom in on it: `--country "<parent>" --id <slug>
   --keep-radius <degrees>` (about 0.4 for an island and its closest neighbours; raise it until the outline looks right), then
   in step 5 write only places that are on that area, set `name` and the menu name in `meta.ts` to the area (e.g. "Tahiti et Moorea"),
   and say in the report that the area is a part of a larger country.
   Exit codes: `2` = country not found or ambiguous (retry with the suggested English name or the ISO 3-letter code, or
   ask the user); `3` = too big or stretched for a flat map (try `--keep-radius 4`, otherwise tell the user this country
   is not supported and why). Note the printed `id`, the name in the legend language, and the number of landmasses kept.

3. **Check and curate the rivers.** The script lists the longest rivers it found inside the country, and other candidates.
   The free data is imperfect: names are in the local language with lost accents (`Rhne` for the Rhône), the same river can
   appear under two names (`Rhein`, `Rhin`), border rivers can be reduced to a sliver, and some countries have very few
   rivers (Japan, Chile: fine, carry on; the game then offers cities and landmarks only; say so in the report).
   - Pick up to 30 well-known rivers (famous ones first, not just the longest). Re-run with
     `--include "A,B,C"` (exact list) and `--alias "Rhne=Rhône,Rhein=Rhin"` to merge duplicates or fix names,
     `--rivers N` for the default count.
   - Look at the result: render `.cache/countries/<id>/preview.svg` with headless Chrome
     (`"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --screenshot=<scratchpad>/preview.png --window-size=900,880 file://$PWD/.cache/countries/<id>/preview.svg`)
     and read the PNG. Check the outline looks like the country, and the rivers are where they should be.
   - Never edit `map.ts` by hand: change the options and re-run.

4. **Scaffold.**
   ```sh
   node scripts/country/scaffold.mjs --id <id> --lang <code>
   ```
   This writes `content.ts` (a skeleton, with every river listed), `meta.ts`, `index.ts` and registers the country.

5. **Write the content** in `src/lib/countries/<id>/content.ts`, following `content-guide.md`:
   - `cities`: up to 30 largest cities, `{ name, lat, lon, fact }`.
   - `sites`: up to 50 famous landmarks, **at most one per city/commune** (field `city`), `{ name, city, lat, lon, fact }`.
   - `foods`: up to 30 culinary specialties, **one per place** (field `place`), `{ name, place, lat, lon, fact }`: coordinates of
     the town of origin, not the region. Regional dishes, cheeses, charcuterie, pastries, protected products (AOP/IGP or the local
     equivalent) tied to one town or village; not dishes eaten everywhere. Origins are often disputed: say so in the fact.
   - `rivers`: for every river id of the map, `{ name, fact }` with the name in the legend language.
   - `name`: the country's name in the legend language.
   - For a language other than `fr`/`en`: translate the whole `strings` object (keep the `{placeholders}` and `{link}`; `lead` is one short line that fits on a phone, and each `legend` entry is a symbol ● ◆ ▬ ■ plus one short word), and the
     `name`/`description` in `meta.ts`. That includes `originLabel` (with `{place}`) and `siteOriginLabel` (with `{place}`).
   - **Write `meta.ts` last, from the final counts**, once verification has dropped or kept items. A kind (cities, landmarks,
     specialties, rivers) appears in the menu card `name` and `description` **only if its count is above 0**: no "rivières"
     with 0 rivers, no "spécialités" with 0 foods, no "monuments" with 0 sites, no "villes" with 0 cities. Use the plural
     kind words of `strings.kindNames` (the test looks for them). Order: cities, landmarks, rivers, specialties, joined in the
     legend language ("A, B et C"; a single kind has no "et"). Example: 12 cities, 9 sites, 0 rivers, 4 foods gives
     `Japon : villes, monuments et spécialités`. The in-game title and legend already adapt by themselves.
   - **Verify with sources** (WebSearch / WebFetch on Wikipedia or Wikidata): the population ranking, every coordinate,
     and every number, date or superlative in a fact. Fix or drop what you cannot confirm. Do not keep a fact you are unsure of.
   - **If a search or fetch fails** (site unreachable, nothing found), try another source once; if it still fails, carry on:
     keep only what you are confident of, replace a doubtful fact with a safer, simpler one (or drop the item), and list
     what stayed unverified in the report. If you run out of good items, stop at the count you reached.

6. **Verify the build.** Run `npm run check`, `npm test` and `npm run build`. The generic tests (`tests/countries.test.ts`)
   check the minimum to play, one site per city, one specialty per place, unique names, complete texts, no leftover `TODO`, and that every city and landmark lies
   on the country (this catches swapped latitude/longitude). Fix what they report; do not weaken the tests.
   Every kind named in the `meta.ts` `name`/`description` must have a count above 0, and every kind with a count above 0 must be named
   (a test checks it).
   Then look at the real game: `npx vite preview --port 4199`, and screenshot `http://localhost:4199/#/map/<id>` with headless
   Chrome at ~1100 px and ~500 px wide, a few times (rounds are random). The tests print a warning for each count below its
   target: that is expected, not an error. Check markers sit on the right places, labels are
   readable and not cut off, rivers highlight correctly. Stop the preview server afterwards.

7. **Report** briefly: the country, id, language, and the counts against the targets (cities / sites / specialties / rivers) with the reason
   for every shortfall (e.g. only 7 rivers in the map data, 38 landmarks found with reliable sources, a failed fetch), and which kinds were left out of the menu title because their count is 0;
   which facts and coordinates were verified against sources and which could not be; limitations seen (stretched map, few rivers,
   overseas territories left out); that nothing is committed. Offer to commit and open a PR.

## Rules

- Titles list only the kinds present: a type goes in the menu title and description only when its count is above 0.
- Work on the current branch; do not commit or push unless asked.
- Coordinates are decimal degrees, `lat` then `lon`, 4 decimals. Latitude/longitude swaps are the most common mistake.
- One short, factual sentence per fact, in the legend language. Hedge when unsure ("environ", "souvent présenté comme").
  No marketing language, no unverifiable "best/oldest/biggest" claims.
- City and landmark names as people say them in the legend language (French: « La tour Eiffel », « Le Mont-Saint-Michel »).
- Specialties: name as people say it (« Le camembert »), a precise town of origin, at most one per place; a specialty may share its town with a city or landmark (never in the same round).
- A site may stand in a city that is also in `cities` (the game never shows both in the same round), but no two sites share a city.
- Disputed or contested territories (e.g. a border region that the free data draws as part of the country): the map follows the free data; add no cities,
  landmarks or specialties there, and mention it in the report so the user can decide.
- Tiny islands: the free outline is coarse (up to ~1 km off), so markers can sit slightly outside the coast; that is expected.
- Keep the data disclaimer (`aboutText`): facts are written in good faith, not systematically verified.
- Scripts live in `scripts/country/` (`build-map.mjs`, `scaffold.mjs`, `lib.mjs`); read their header comments for all options.
