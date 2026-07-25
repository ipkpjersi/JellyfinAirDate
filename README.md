# JellyfinAirDate

Firefox and Chrome Manifest V3 extension that adds the next airing episode date to Jellyfin
show, season, and episode detail pages when the show is still airing.

The content script reads Jellyfin's detail-page metadata (the item type and id stamped on the
detail buttons, plus the title/subtitle block), searches TVmaze first, falls back to AniList for
anime, and inserts a compact row below Jellyfin's existing year/runtime/rating line.

It is a port of PlexAirDate with the same data sources, caching, rate-limit handling, and
MyAnimeList scoring; only the page detection and row placement differ.

## Load in Firefox

1. Open `about:debugging#/runtime/this-firefox`.
2. Click `Load Temporary Add-on...`.
3. Select `JellyfinAirDate-firefox/manifest.json`.
4. Open or refresh a Jellyfin show, season, or episode page.

## Load in Chrome

1. Open `chrome://extensions`.
2. Enable `Developer mode`.
3. Click `Load unpacked`.
4. Select the `JellyfinAirDate-chrome` folder.
5. Open or refresh a Jellyfin show, season, or episode page.

## Build the distributable zips

The packaged zips in `JellyfinAirDate-chrome/dist/` and `JellyfinAirDate-firefox/dist/` are what
gets uploaded to the Chrome Web Store and addons.mozilla.org.

```sh
scripts/build.sh            # both browsers
scripts/build.sh chrome     # one browser
scripts/build.sh firefox
```

Each run writes `dist/jellyfin-air-date-<browser>-<version>.zip`, taking `<version>` from that
browser's own `manifest.json`, so releasing is:

1. Bump `"version"` in **both** `JellyfinAirDate-chrome/manifest.json` and
   `JellyfinAirDate-firefox/manifest.json` (they are always released together).
2. Add the matching entry at the top of `CHANGELOG.md` (its wording is written to be pasted
   straight into the Firefox Add-ons release notes field).
3. Run `scripts/build.sh`.

`manifest.json` and `src/` must sit at the archive root rather than inside a wrapper folder, which
is what the script produces; the two `src/` directories are kept byte-identical and only the
manifests differ (Firefox adds the `browser_specific_settings` block).

## Scripts

- `scripts/build.sh` - packages the distributable zips (see above).
- `scripts/probe-rate-limits.mjs` - measures the AniList and MyAnimeList rate limits the extension
  has to live within. It bursts the same requests the extension makes until the API returns 429,
  then polls until it recovers, reporting how many got through, how long the burst lasted, and how
  long the reset window is - use it to re-tune the retry/backoff constants in `content.js` from real
  numbers instead of guesses. Run it as
  `node scripts/probe-rate-limits.mjs --api anilist|jikan|both`. It deliberately trips the limits,
  so run it sparingly and not while you are browsing.
- `scripts/probe-jikan-curl.sh` - the same burst/recovery probe for MyAnimeList, via curl. Node's
  `fetch` is 504'd by MyAnimeList's Cloudflare on a TLS fingerprint check before the rate limiter is
  even reached, so it cannot measure that limit; curl's fingerprint passes, like a real browser. Run
  it as `scripts/probe-jikan-curl.sh [max_burst] [poll_seconds] [recovery_seconds]`.

## What it shows

- Show pages: latest and next episode air dates, plus the MyAnimeList score for anime.
- Season pages: the year the season aired, the latest and next episode air dates, and the
  MyAnimeList score for that specific season (each anime season is its own MAL entry).
- Episode pages: the air date of the episode being viewed, the latest and next episode air
  dates, the season's MyAnimeList score, and MAL's per-episode poll score tagged `EP`.

## Notes

- No API key is required.
- The extension is anime-aware: a score line is only added when the title matches on
  MyAnimeList or AniList, since Jellyfin already shows its own community rating.
- The content script is matched broadly so self-hosted Jellyfin servers on any host, port, or
  base path work, but it exits unless the page has a Jellyfin detail-page layout.
- Jellyfin keeps previously visited detail pages in the DOM; the row is scoped to the visible
  page, so navigating back and forth does not leave stale rows behind.
