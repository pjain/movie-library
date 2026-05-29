# Movie Library Schemas

## Local data file

Default path: `~/.movie-library/movies.json`

```json
{
  "profile": {
    "default_region": null,
    "preferred_platforms": [],
    "preferred_genres": [],
    "favorite_movies": [],
    "favorite_people": [],
    "updated_at": null
  },
  "watchlist": [],
  "watched": [],
  "removed": []
}
```

## Movie record

```json
{
  "title": "Dune: Part Two",
  "original_title": "Dune: Part Two",
  "year": 2024,
  "status": "watchlist",
  "ids": {
    "tmdb": "693134",
    "imdb": "tt15239678"
  },
  "ratings": {
    "imdb": "8.5",
    "rotten_tomatoes": "92%"
  },
  "genres": ["Science Fiction", "Adventure"],
  "directors": ["Denis Villeneuve"],
  "cast": ["Timothee Chalamet", "Zendaya", "Rebecca Ferguson"],
  "production_companies": ["Legendary Pictures"],
  "runtime_minutes": 166,
  "where_to_watch": {
    "region": "US",
    "checked_at": "2026-05-29",
    "providers": [
      {
        "platform": "Max",
        "access_type": "subscription",
        "deep_link": null
      }
    ]
  },
  "added_at": "2026-05-29",
  "watched_at": null,
  "user_rating": null,
  "notes": "",
  "updated_at": "2026-05-29"
}
```

## De-duplication

Use this order:

1. Match by `ids.imdb` if available.
2. Match by `ids.tmdb` if available.
3. Match by normalized `title` + `year`.
4. If the year is missing, treat as ambiguous if more than one plausible match exists.

## Status values

- `watchlist`: user wants to watch.
- `watched`: user has watched.
- `removed`: optional tombstone only; do not rely on this as a primary list.

## Find Movie output template

```markdown
| Movie | IMDb | Rotten Tomatoes | Where to watch in {REGION} | Notes |
|---|---:|---:|---|---|
| Dune: Part Two (2024) | 8.5 | 92% | Max subscription; Apple TV rent/buy | Availability checked today |
```

## Recommendation output template

```markdown
1. **Arrival (2016)** — Similar thoughtful sci-fi tone and directed by Denis Villeneuve. Available on: {providers}. IMDb: {rating}; RT: {score}.
```
