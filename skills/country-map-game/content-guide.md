# Writing the content of a country

Reference: `src/lib/countries/france/content.ts` (30 cities, 50 sites, 30 specialties, 30 rivers, in French).

## Shape

```ts
export const content: CountryContent = {
  id: 'france',          // = folder name, set by the scaffold
  name: 'France',        // country name in the legend language
  lang: 'fr',
  cities: [
    { name: 'Paris', lat: 48.8566, lon: 2.3522, fact: "…" },
  ],
  sites: [
    { name: 'La tour Eiffel', city: 'Paris', lat: 48.8584, lon: 2.2945, fact: "…" },
  ],
  foods: [
    { name: 'Le roquefort', place: 'Roquefort-sur-Soulzon', lat: 43.9767, lon: 2.9889, fact: "…" },
  ],
  rivers: {
    loire: { name: 'La Loire', fact: "…" },   // keys = the river ids printed by build-map.mjs / listed by the scaffold
  },
  strings: makeStrings('fr', 'France'),         // 'fr' and 'en' are built in; otherwise write the whole object
}
```

## Cities (about 30)

- The largest by population (city proper, current figures; check Wikipedia/INSEE-like national statistics).
- Skip suburbs of a bigger city that would sit on top of it on the map (e.g. Villeurbanne next to Lyon); markers closer than
  ~15 map units fail the tests, and the game avoids showing cities closer than 30 units in the same round.
- Spread across the country; if the ranking is very lopsided, swap the smallest ones for important regional capitals.
- Up to 30. Fewer is fine (a small country, a failed fetch): carry on with what you have. The tests only need 8 so rounds can be built.

## Sites (about 50, one per city or commune)

- Famous, recognisable places: monuments, castles, cathedrals, bridges, mountains, natural wonders, historic sites.
- **At most one per city/commune**: the `city` field must be unique across the list.
  The `city` field is **shown to the player** ("Monument · Rabat"): use a real, readable town name, never a note like "(ruines)".
  (Paris has the Eiffel Tower, not also Sacré-Cœur.) A site may be in a commune that is also in `cities`.
- Spread across the country, not only the capital region. Use the site's own coordinates (Wikidata), not the city centre.
- Aim for 50, but any number works, even none: if you cannot find enough well-sourced landmarks, stop at the count you have rather than pad. With none, leave "monuments" out of the `meta.ts` title.
  Name = how people say it in the legend language, with its article if the language uses one.

## Foods (culinary specialties, about 30, optional)

- Regional specialties tied to **one town or village of origin** (cheese, charcuterie, pastry, dish, protected product),
  placed on that town. One specialty per `place`; two dishes never share a town.
- Pick famous, distinctive ones, spread across the country; skip dishes eaten everywhere (a plain omelette, pizza in Italy).
- Origins are often legends or disputed: say so in the fact ("selon la légende", "attribué à", "serait né"), and use the
  official origin (AOP/IGP or national equivalent) only when sourced. Verify the town of origin, any date and any label.
- Coordinates of the town (Wikidata), `lat` then `lon`. Beware homonyms (Munster, Lyon…): check the country and the region.
- Fewer than 30, or none, is fine; carry on with what you have. With none, leave "spécialités" out of the `meta.ts` title.

## Rivers

- Every river id of `map.ts` needs `{ name, fact }`; no extra ids.
- `name`: the name in the legend language, with article where the language has one (« La Loire », « L'Allier », « Le Rhône »).
- Rivers the map data lacks entirely (e.g. the Oise, the Isère in France) cannot be added by hand: the courses come from the
  generated map. They are simply absent. With no river at all, leave "rivières" out of the `meta.ts` title.

## Facts

- One sentence, ~15-30 words, in the legend language, interesting for a general audience.
- Concrete and checkable: a date, a number, a name, a record, an event. Each one verified against a source.
- Hedge approximations ("environ 330 m", "plus de 3 000"). Prefer facts that do not change.
- Never repeat the same fact for a city and the site in it; vary the angle.
- No fact for which you have no source. Better a duller verified fact than a vivid doubtful one.
- If sources cannot be reached, write only what you are confident of, and list the unverified items in your final report.

## Strings (languages other than fr/en)

Translate every field of `Strings` (see `src/lib/countries/types.ts`, and the English and French sets in
`src/lib/countries/strings.ts`). Keep `{country}`, `{n}`, `{round}`, `{score}`, `{place}` and `{link}` exactly; keep the tone short and
friendly; keep the data disclaimer in `aboutText`. Also translate `name` and `description` in `meta.ts`.
Typical gotcha: languages with grammatical cases or articles for the country name; prefer wording that uses the name unchanged.
