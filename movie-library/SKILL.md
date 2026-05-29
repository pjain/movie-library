---
name: movie-library
description: movie discovery and personal watch tracking for openclaw, hermes, and compatible agents. use when the user asks to find movies, compare imdb or rotten tomatoes ratings, locate where a movie streams in their geography, add movies to a watchlist, mark movies watched, remove movies from lists, recommend similar movies, or find films by director, actor, genre, or production house. designed for agents with existing web/api/file tools and local json persistence rather than bundled api execution.
---

# Movie Library

## Purpose

Use this skill to manage a user's movie discovery and watch history. The skill instructs the agent to use available web/API tools for movie metadata, ratings, streaming availability, recommendations, and filmography search, while storing the user's watchlist and watched history in a local JSON file.

Do not run shell commands or execute bundled API scripts for normal use. Prefer the agent's existing web, API, file-read, and file-write tools.

## First-run setup

On the first use, establish the user's default geography for streaming availability.

1. Check whether a local data file already exists at the configured path. Default path: `~/.movie-library/movies.json`.
2. If no profile exists, ask: "What geography should I use by default for streaming availability? You can also tell me later when you are temporarily in another geography."
3. Store the answer as `profile.default_region` using a stable country/region code when possible, such as `US`, `IN`, or `GB`.
4. If the user says they are temporarily in another geography, use that region for the current lookup only unless they explicitly ask to change their default.

Use `references/schemas.md` for the JSON schema and examples.

## Data source preferences

Use the best available tool/provider in this order. If a preferred source is unavailable, use the next reliable source and disclose the limitation briefly.

1. **Movie metadata, IDs, cast, crew, genres, production companies:** TMDB or another structured movie database.
2. **IMDb and Rotten Tomatoes ratings:** OMDb if available, because it can return IMDb ratings and Rotten Tomatoes ratings in one response when matched correctly. Otherwise use trustworthy web/API sources and cite or name the source.
3. **Where to watch by geography:** Watchmode, JustWatch-compatible data, TMDB watch providers, or another current streaming availability API. Availability changes often; always perform a fresh lookup unless the user explicitly asks to use cached data.
4. **Recommendations:** Combine structured "similar/recommendations" endpoints, genre/person/company metadata, the local watched list, watchlist exclusions, and the user's stated preferences.

Never fabricate ratings or availability. If a rating or provider result is missing, return `unknown` or explain that the source did not provide it.

## Local storage rules

Maintain a local JSON file at `~/.movie-library/movies.json` unless the user or runtime config specifies a different path.

- Create the file if it does not exist.
- Preserve existing entries and unknown fields.
- Normalize titles with year and stable external IDs when available.
- De-duplicate using a stable ID first, then normalized title + year.
- Use ISO dates such as `2026-05-29` for `added_at`, `watched_at`, and `updated_at`.
- When writing the file, make a best effort to preserve valid JSON formatting and avoid destructive overwrites.
- If filesystem access is unavailable, maintain a response-local representation and tell the user persistence is unavailable.

## Task workflows

### Find Movie

Input may be one movie title or multiple titles. For each movie:

1. Resolve the title to the most likely movie. If multiple plausible matches exist, use the user's context; if still ambiguous, show the top few candidates with year/director and ask which one.
2. Retrieve metadata: title, year, director, primary cast, genre, runtime, plot summary, and stable IDs where available.
3. Retrieve IMDb rating and Rotten Tomatoes rating.
4. Determine the geography:
   - Use a temporary region mentioned by the user for this request.
   - Otherwise use `profile.default_region`.
   - If neither exists, ask for geography before checking availability.
5. Retrieve current streaming availability by geography, distinguishing subscription, rent, buy, free/ad-supported, and unavailable.
6. Return a concise table or structured list.
7. Offer to add one or more results to the watchlist only when that is a natural next step.

Preferred output fields:

- Title
- Year
- IMDb rating
- Rotten Tomatoes rating
- Where to watch in selected geography
- Notes on ambiguity or missing data

### Add Movie

When the user asks to add a movie to the watchlist:

1. Resolve the movie and fetch basic metadata if missing.
2. If the movie already exists in `watched`, ask whether to also keep it on the watchlist or skip adding.
3. If it already exists in `watchlist`, update metadata and `updated_at`; do not duplicate.
4. Add the normalized record to `watchlist` with `status: "watchlist"`, `added_at`, source IDs, ratings if known, and any user notes.
5. Save the JSON file and confirm the addition with title and year.

### Watched

When the user says they watched a movie:

1. Resolve the movie against `watchlist`, `watched`, and fresh lookup results.
2. Add or update the record in `watched` with `status: "watched"` and `watched_at`.
3. Move it out of `watchlist` unless the user explicitly asks to keep it there.
4. Capture optional user rating, notes, favorite status, or rewatch flag if provided; do not require these.
5. Save the JSON file and confirm.

### Remove

When the user asks to remove a movie:

1. Search all local lists: `watchlist`, `watched`, and any auxiliary lists.
2. If there is one clear match, remove it from all lists.
3. If there are multiple matches, ask which one before deleting.
4. Optionally add a tombstone to `removed` only if useful for preventing immediate re-addition; otherwise deletion is sufficient.
5. Confirm exactly what was removed.

### Recommend

Recommendations are optional but should work when the available tools support them.

For requests like "recommend movies similar to X" or "recommend genre Y":

1. Load the user's watched list, watchlist, preferences, and default geography.
2. Build candidate movies from similar-title endpoints, genre searches, director/actor/company overlaps, and reputable curated sources if needed.
3. Exclude movies already in `watched` unless the user asks for rewatch ideas.
4. De-prioritize movies already in `watchlist` unless the user asks what to watch next from the watchlist.
5. Filter or annotate by current streaming availability in the selected geography.
6. Return 5-10 recommendations with a one-sentence rationale for each.

When recommendations are based on sparse history, say so and ask the user to add a few favorites or preferred genres to improve future recommendations.

### Find by director, actor, genre, or production house

For requests like "find all movies by Greta Gerwig," "movies starring Tabu," or "A24 films":

1. Identify the entity type: director, actor, genre, production company/house, or ambiguous.
2. Use a structured source such as TMDB when available.
3. For people, distinguish directed-by from acted-in when relevant.
4. Sort results by relevance, release date, popularity, or rating based on the user's request. Default to well-known/relevant first for broad queries.
5. Optionally annotate which results are already watched or on the watchlist.
6. If asked, filter by geography-specific streaming availability.

## V2 integrations: Notion and Airtable

For v1, store lists locally in JSON only.

If the user asks to use Notion or Airtable:

1. Explain that this requires authorization and workspace/database/table details.
2. Ask for the appropriate authorization/setup path supported by the runtime. Do not request secrets in an insecure chat surface if the platform has secure setup or environment-variable support.
3. Do not migrate or sync until the user explicitly authorizes the integration and confirms the destination.
4. Continue to keep a local JSON backup unless the user asks not to.

## Safety and reliability

- Do not scrape sites that prohibit automated access when a structured API is available.
- Do not guess ratings, providers, or availability.
- Treat streaming availability as time-sensitive and refresh it for each lookup.
- Keep API keys out of chat transcripts when the runtime supports secure configuration.
- Do not overwrite or delete the local JSON file without reading the existing content first.
- For ambiguous titles, clarify before modifying lists.

## Reference files

- `references/schemas.md`: local JSON schema, movie record shape, and output templates.
- `references/provider-guide.md`: recommended providers, fields to request, and fallback behavior.
