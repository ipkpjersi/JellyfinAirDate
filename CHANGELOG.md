# Changelog

All notable changes to Jellyfin Air Date are listed here, newest first. Each entry matches a version bump in both the Chrome and Firefox manifests. The notes for a version can be copy-pasted into the Firefox Add-ons release notes field.

## 0.2.0

- The MyAnimeList, AniList, and TVmaze names and scores the extension adds are now clickable and open the matching entry in a new tab: the score line opens the anime's MyAnimeList or AniList page (the per-episode score opens that episode's own MyAnimeList page), and the source name next to each air date opens the show on TVmaze or AniList. Jellyfin already links IMDb and TMDB itself, so those are left alone.
- The links do not change how any of the text looks: they take the colour and opacity they already had and only show an underline while hovered or keyboard-focused.

## 0.1.0

- Initial release, ported from Plex Air Date 0.9.2 with the same feature set: the current, latest, and next episode air dates on show, season, and episode pages, the year a season aired on season pages, the MyAnimeList (MAL) score for anime, and MAL's per-episode poll score on episode pages. The data sources (TVmaze for air dates, Tenrai's MyAnimeList API and AniList for anime scores), the caching, the stored MAL ids, and the rate-limit handling are unchanged.
- Rewrote the page detection for Jellyfin. The extension reads the item type and id that Jellyfin stamps on the detail-page buttons, so show, season, and episode pages are identified exactly rather than inferred from the layout, and movies and other item types are skipped. The season and episode numbers come from the detail page's subtitle line.
- The row is inserted at the top of the detail page's primary content, directly below Jellyfin's own year/runtime/rating line and above the version/video/audio/subtitle selects. It is deliberately not placed inside the detail ribbon that holds those lines: Jellyfin's desktop layout pins that element to a fixed height (`height: 7.2em` with a matching negative top margin so it overlays the backdrop), so extra lines added in there overflow the box and paint on top of the content below instead of pushing it down - on episode pages, which have the most lines, the MyAnimeList per-episode score landed on top of the "Video 1080p" line.
- The row is scoped to the currently visible detail page. Jellyfin keeps previously visited pages in the DOM rather than removing them, so rows on those cached pages are cleared to prevent a stale row reappearing when navigating back.
- The row inherits the page's text colour and dims its secondary parts with opacity instead of fixed colours, so it reads correctly on both Jellyfin's dark and light themes.
