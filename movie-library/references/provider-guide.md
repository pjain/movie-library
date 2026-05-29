# Provider Guide

## Recommended providers

### TMDB

Use for:

- Title search and disambiguation
- Stable TMDB IDs and IMDb external IDs
- Directors, cast, genres, runtime, production companies
- Similar/recommended movies
- Watch provider links where available

Useful concepts:

- Search movie by title.
- Fetch movie details with append-to-response for credits, external IDs, recommendations, similar titles, and watch providers when the tool supports it.
- For a person query, search person, then fetch movie credits and filter by crew job or cast role.
- For production house/company query, search company, then discover movies by company.

### OMDb

Use for:

- IMDb rating
- Rotten Tomatoes rating when included in the `Ratings` array
- Basic fallback metadata

Match carefully using IMDb ID when possible. Title-only OMDb searches can return the wrong remake, sequel, or regional title.

### Watchmode, JustWatch-compatible providers, TMDB watch providers, or equivalent

Use for:

- Current streaming availability by country/region
- Access type: subscription, rent, buy, free, ads, unavailable
- Deep links when available

Availability changes often. Refresh availability for every direct user request about where to watch.

## Fallback behavior

If a source is unavailable:

1. Use another structured provider before generic web search.
2. If generic web search is required, prefer official provider pages, movie database pages, or reputable entertainment databases.
3. Mark missing fields as `unknown`; do not infer ratings or availability.
4. Mention which fields could not be verified.

## Geography handling

- Store `profile.default_region` after first-run setup.
- Accept natural language regions such as "India", "US", "UK", "Canada", and normalize when possible.
- If the user says "I'm traveling in India this week," use `IN` for the current query only unless they say to change the default.
- When provider results are returned by country code, pass the normalized region code to the API/tool.

## Common user intents

- "find movie X" -> metadata + ratings + where to watch.
- "add X" -> resolve title, store in watchlist.
- "I watched X" -> move from watchlist to watched.
- "remove X" -> remove from all local lists after ambiguity check.
- "recommend movies like X" -> similar/recommendations + user history + availability.
- "find all movies by X" -> determine whether X is a person/company/genre, then query structured credits/discover endpoints.
