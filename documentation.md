documentation-qobuz.md

markdown
# Voria Qobuz HiFi API

A Cloudflare Worker that exposes Qobuz catalog, metadata, artwork, discovery, and playback functionality through a simple HiFi-API-style HTTP interface.

> [!WARNING]
> **AI-generated documentation:** This documentation and the Worker were generated with AI assistance. They may contain mistakes, become outdated, or stop matching Qobuz or Cloudflare behavior.
>
> If you encounter an issue, please open a **GitHub Issue** in the repository where you found this project. Include the endpoint, HTTP status, sanitized response, and relevant Worker logs.
>
> **Never post your Qobuz auth token, App Secret, or other credentials.**

---

# 1. What this Worker does

The Worker provides:

- Track metadata
- Track lookup by Qobuz ID
- Track lookup by ISRC
- Track search
- Album search
- Album metadata
- Artist metadata
- Playlist metadata
- Artwork
- Qobuz playback URL resolution
- 24-bit / 192 kHz playback requests
- 24-bit / 96 kHz playback requests
- CD-quality FLAC
- MP3 320 kbps fallback
- Multiple Qobuz auth tokens
- Token failover
- App ID/App Secret credential fallback
- Shared Cloudflare KV metadata caching
- Short-lived playback URL memory caching
- Similar artists
- Generated radio
- Most-popular discovery
- Featured releases/charts
- Genres
- `/ping` account and playback diagnostics

The Worker accepts:
GET
HEAD
OPTIONS

The API intentionally avoids pretending that unsupported or unverified Qobuz endpoints exist.

2. Requirements

You need:

A Cloudflare account
A Cloudflare Worker
The Qobuz Worker source code
A Qobuz App ID
A Qobuz App Secret
At least one Qobuz user auth token
A Cloudflare KV namespace
The KV namespace bound as:
GENERAL_MUSIC_CACHE
3. Qobuz environment variables
Required
QOBUZ_APP_ID
QOBUZ_APP_SECRET
QOBUZ_USER_AUTH_TOKEN

Example:

QOBUZ_APP_ID=YOUR_APP_ID
QOBUZ_APP_SECRET=YOUR_APP_SECRET
QOBUZ_USER_AUTH_TOKEN=YOUR_USER_AUTH_TOKEN
4. Multiple Qobuz accounts/tokens

The Worker supports up to ten user auth tokens.

Use:

QOBUZ_USER_AUTH_TOKEN
QOBUZ_USER_AUTH_TOKEN_2
QOBUZ_USER_AUTH_TOKEN_3
QOBUZ_USER_AUTH_TOKEN_4
QOBUZ_USER_AUTH_TOKEN_5
QOBUZ_USER_AUTH_TOKEN_6
QOBUZ_USER_AUTH_TOKEN_7
QOBUZ_USER_AUTH_TOKEN_8
QOBUZ_USER_AUTH_TOKEN_9
QOBUZ_USER_AUTH_TOKEN_10

You do not need to configure all ten.

For example:

QOBUZ_APP_ID=APP_ID
QOBUZ_APP_SECRET=APP_SECRET

QOBUZ_USER_AUTH_TOKEN=TOKEN_1
QOBUZ_USER_AUTH_TOKEN_2=TOKEN_2
QOBUZ_USER_AUTH_TOKEN_3=TOKEN_3

The Worker can fail over between these tokens.

5. Slot-specific App credentials

The Worker also supports:

QOBUZ_APP_ID_2
QOBUZ_APP_SECRET_2

QOBUZ_APP_ID_3
QOBUZ_APP_SECRET_3

and so on.

For example:

QOBUZ_APP_ID=APP_ID_1
QOBUZ_APP_SECRET=APP_SECRET_1
QOBUZ_USER_AUTH_TOKEN=TOKEN_1

QOBUZ_APP_ID_2=APP_ID_2
QOBUZ_APP_SECRET_2=APP_SECRET_2
QOBUZ_USER_AUTH_TOKEN_2=TOKEN_2

The Worker prefers a token's own App credentials when available.

It can also attempt other configured App ID/App Secret pairs when necessary.

This matters because Qobuz catalog access and playback access can fail independently.

6. Never publish Qobuz credentials

Do not commit these to GitHub:

QOBUZ_APP_SECRET
QOBUZ_USER_AUTH_TOKEN
QOBUZ_USER_AUTH_TOKEN_2
QOBUZ_USER_AUTH_TOKEN_3
...

Do not include credentials in:

GitHub Issues
Screenshots
Discord
Logs
Source code
Documentation

Use Cloudflare's secret/environment-variable system.

Your public repository should contain placeholders only.

7. Cloudflare KV setup

The Worker uses one shared KV namespace:

GENERAL_MUSIC_CACHE

The same namespace can be shared with the Deezer Worker.

Qobuz cache keys use:

music:qobuz:

Deezer cache keys use:

music:deezer:

The prefixes prevent collisions.

Create the namespace

In Cloudflare:

Open Workers & Pages
Open KV
Create a namespace
Name it something like:
GENERAL_MUSIC_CACHE
Bind the namespace

Open your Worker:

Go to Settings
Open Bindings
Add KV Namespace
Set the binding name exactly to:
GENERAL_MUSIC_CACHE
Select your KV namespace
Deploy the Worker

The Worker will then receive:

env.GENERAL_MUSIC_CACHE
8. What gets cached?

The KV cache is intended for relatively stable Qobuz catalog metadata.

Examples:

Track metadata
ISRC lookups
Album metadata
Artist metadata
Playlist metadata

The cache prefix is:

music:qobuz:
9. Do not permanently cache Qobuz stream URLs

Qobuz playback URLs are signed and time-sensitive.

Do not store:

streamUrl

as permanent KV metadata.

The Worker instead uses a short-lived Worker-memory cache for playback resolution.

Current playback URL memory cache:

20 seconds

This is intended to reduce repeated playback resolution for requests arriving close together.

It is not intended to permanently store signed URLs.

10. Find your Worker URL

Suppose Cloudflare gives your Worker this URL:

https://qobuz-api.example.workers.dev

That becomes your API base URL.

Replace it with your actual Worker URL in the examples below.

11. /

Returns API information.

GET /

It can show:

API version
Provider
Capabilities
Cache configuration
Available endpoints
Configured token count
Verified Qobuz methods
12. /info-api

Alias for /.

GET /info-api

This is useful for applications that want to inspect the API's capabilities.

13. /track

Main rich track endpoint.

By Qobuz ID
GET /track?id=90957991
By ISRC
GET /track?isrc=USUG11600976
By search
GET /track?q=Blinding%20Lights&artist=The%20Weeknd

The response can contain:

Track metadata
Artist
Album
Artwork
Format
MIME type
Stream URL
Bit depth
Sampling rate
Duration
Playback diagnostics
Token slot
14. /info

Track lookup endpoint.

GET /info?id=90957991

or:

GET /info?isrc=USUG11600976
15. Playback quality

Supported quality values include:

Value	Preferred format
best	24/192 → 24/96 → 16/44.1 → MP3 320
192	24-bit / up to 192 kHz
hires192	24-bit / up to 192 kHz
hires_192	24-bit / up to 192 kHz
96	24-bit / up to 96 kHz
hires96	24-bit / up to 96 kHz
hires_96	24-bit / up to 96 kHz
flac	16-bit / 44.1 kHz
cd	16-bit / 44.1 kHz
16	16-bit / 44.1 kHz
lossless	16-bit / 44.1 kHz
mp3	MP3 320 kbps
320	MP3 320 kbps
mp3_320	MP3 320 kbps
16. best quality

The best mode requests the highest available format first.

Normal order:

24-bit / 192 kHz
        ↓
24-bit / 96 kHz
        ↓
16-bit / 44.1 kHz FLAC
        ↓
MP3 320 kbps

This is intentional because Qobuz catalog metadata can sometimes under-report the actual master available through playback.

The actual resolved playback format is treated as authoritative.

17. Qobuz format IDs

The Worker uses these Qobuz format IDs:

ID	Format
5	MP3 320 kbps
6	FLAC 16-bit / 44.1 kHz
7	FLAC 24-bit / up to 96 kHz
27	FLAC 24-bit / up to 192 kHz
18. /stream

Playback endpoint.

By ID
GET /stream?id=90957991
By ISRC
GET /stream?isrc=USUG11600976
Select quality
GET /stream?id=90957991&quality=192
19. Direct redirect mode

Add:

stream=1

Example:

GET /stream?id=90957991&quality=flac&stream=1

If Qobuz provides a normal direct URL, the Worker can redirect the client to it.

20. Important: segmented playback

Modern Qobuz playback can return a segmented stream rather than a single direct file.

For example, Qobuz can provide:

url_template

containing:

$SEGMENT$

along with information such as:

n_segments

This is not equivalent to a normal direct streamUrl.

The Worker intentionally does not pretend a segmented URL template is a direct playable file URL.

If direct redirect mode encounters a segmented stream, the Worker can return:

409 SEGMENTED_STREAM

with information about the segmented stream.

21. /stream?proxy=1

The Worker can proxy an already-resolved Qobuz CDN URL.

Example:

GET /stream?proxy=1&url=ENCODED_QOBUZ_URL

The proxy:

Supports HTTP Range requests
Passes ranges upstream
Returns relevant content headers
Uses:
Cache-Control: no-store

Do not treat this proxy URL as a permanent cache.

22. /search

General search.

GET /search?q=Blinding%20Lights
Track search
/search?q=Blinding%20Lights&type=track
Album search
/search?q=After%20Hours&type=album
Artist search
/search?q=The%20Weeknd&type=artist
Playlist search
/search?q=Workout&type=playlist
Everything
/search?q=The%20Weeknd&type=all
ISRC
/search?isrc=USUG11600976

or:

/search?i=USUG11600976
23. Search aliases

The Worker supports:

Parameter	Meaning
q	General search
query	General search
s	General search
a	Artist search
al	Album search
p	Playlist search
i	ISRC lookup

Examples:

/search?s=Blinding%20Lights
/search?a=The%20Weeknd
/search?al=After%20Hours
/search?p=Workout
24. Search pagination

Use:

limit
offset

Example:

/search?q=The%20Weeknd&type=track&limit=25&offset=25
25. Search stream enrichment

Track search can enrich results with a direct playback URL.

This is more expensive than metadata-only search.

If you only need catalog results, use:

/search?q=The%20Weeknd&type=track&include_stream=0

This is recommended for:

Search suggestions
Browsing
Metadata-only interfaces
Large result lists

Then resolve playback when the user actually presses play.

26. /album
GET /album?id=ALBUM_ID

Example:

/album?id=163064

Pagination:

/album?id=163064&limit=25&offset=0

The response can contain:

Album metadata
Artist
Artwork
Release information
Track list
Track count
Pagination
Streamability information
27. /artist
GET /artist?id=ARTIST_ID

The response can contain:

Artist metadata
Biography
Releases
Related artists
Discography information
28. /playlist
GET /playlist?id=PLAYLIST_ID

Pagination:

/playlist?id=PLAYLIST_ID&limit=50&offset=0
29. /cover

Artwork lookup:

GET /cover?id=TRACK_ID

Track-specific:

GET /cover?track_id=TRACK_ID

Album-specific:

GET /cover?album_id=ALBUM_ID

The response provides normalized artwork URLs.

30. /lyrics
GET /lyrics?id=TRACK_ID

The current stable Qobuz Worker intentionally returns:

501 Not Implemented

Qobuz does not provide a stable lyrics endpoint through the catalog API surface used by this Worker.

This is intentional.

For Voria, use a separate lyrics extension/provider instead of pretending Qobuz supplies lyrics.

31. /recommendations
Query-based discovery
GET /recommendations?q=The%20Weeknd

This uses Qobuz's verified:

most-popular/get

discovery functionality.

The response identifies this as:

most_popular

This should not be interpreted as personalized recommendations.

32. Personalized recommendations

Qobuz has a recommendation flow based around:

dynamic/suggest

which requires listening context.

The current Worker intentionally does not pretend this is a simple GET request.

A request such as:

/recommendations

without enough context can return:

501

explaining that verified personalized recommendations require listening context.

This is intentional API honesty rather than a missing feature by accident.

33. /radio
GET /radio?id=ARTIST_OR_TRACK_ID

or:

/radio?artist_id=ARTIST_ID

The Worker generates a radio queue using verified Qobuz catalog functionality.

The current generated-radio path uses:

artist/getSimilarArtists
+
catalog/search

The response identifies the radio as:

generated_from_qobuz_catalog

This is not a claim that Qobuz exposes a stable native /radio/get endpoint.

34. /artist/similar
GET /artist/similar?id=ARTIST_ID

Uses Qobuz's:

artist/getSimilarArtists

method.

35. /album/similar
GET /album/similar?id=ALBUM_ID

The current stable Worker returns:

501

for this route.

This is deliberate because the Worker does not have a verified stable Qobuz album-similarity endpoint.

Use:

/artist/similar

or:

/radio

instead.

36. /chart
GET /chart

The route uses:

album/getFeatured

By default it requests:

type=new-releases

Example:

/chart?type=new-releases

Optional genre:

/chart?type=new-releases&genre_id=6

Pagination:

/chart?limit=25&offset=0

The exact accepted featured types ultimately depend on Qobuz.

37. /genre
GET /genre

This uses Qobuz's:

genre/list

endpoint.

38. /ping

The Qobuz /ping endpoint performs an actual account/playback diagnostic.

GET /ping

The Worker:

Searches Qobuz for a known track
Checks catalog access
Attempts modern signed playback
Tests lossless playback
Determines the stream type
Reports the credential slot used

Example:

{
  "slot": 1,
  "status": "active",
  "can_catalog": true,
  "can_playback": true,
  "can_lossless": true,
  "playback_signing_mode": "modern_session_file_url",
  "playback_stream_type": "segmented"
}
39. Catalog access vs playback access

These are separate concepts.

A token may have:

can_catalog: true

while:

can_playback: false

This usually means the credentials are sufficient for catalog operations but not sufficient for playback.

The /ping diagnostics are designed to make this distinction obvious.

40. Token failover

The Worker supports up to ten Qobuz auth tokens.

It tracks recent token failures in Worker memory.

A failed token can temporarily enter a cooldown period.

Current cooldown:

30 seconds

This prevents repeatedly hammering a known-bad credential on every request.

41. App credential fallback

Qobuz playback depends on more than just the user auth token.

The Worker keeps these credentials separate:

App ID
App Secret
User Auth Token

For a token slot, the Worker prefers its matching App credentials.

If necessary, it can attempt another configured App ID/App Secret pair.

This means configurations like this are supported:

QOBUZ_APP_ID=APP_ID
QOBUZ_APP_SECRET=APP_SECRET

QOBUZ_USER_AUTH_TOKEN=TOKEN_1
QOBUZ_USER_AUTH_TOKEN_2=TOKEN_2
QOBUZ_USER_AUTH_TOKEN_3=TOKEN_3
42. Modern Qobuz playback

Modern Qobuz playback uses:

session/start

followed by:

file/url

The Worker handles the required session/authentication information.

The modern endpoint can return either:

A direct playback URL
A segmented stream template

A segmented response can contain:

url_template
n_segments
key
sampling_rate
format_id

The Worker does not automatically turn a segmented stream template into a fake direct URL.

43. Legacy direct URL resolution

For places where the API specifically needs a direct signed URL, the Worker can use:

track/getFileUrl

This is especially useful for search stream enrichment.

That is why /search can return a direct streamUrl even though modern Qobuz playback may return a segmented stream.

44. Cache architecture
Worker memory

Used for:

Playback URLs
Temporary metadata
Request coalescing
Token state
Cloudflare KV

Binding:

GENERAL_MUSIC_CACHE

Prefix:

music:qobuz:

Used for stable catalog metadata.

Qobuz

If the Worker has no usable cached data, it requests the information from Qobuz.

45. Request coalescing

When several requests ask for the same track at nearly the same time, the Worker can share an in-flight lookup.

This is useful for:

Search pages
Autoplay queues
Multiple clients
Rapid repeated playback requests
46. CORS

The Worker supports permissive CORS.

This makes it suitable for:

Web apps
PWAs
Flutter Web
Browser music players
Other HTTP clients

If you expose the Worker publicly, remember that anyone who can reach it can use the Qobuz functionality provided by your configured credentials.

47. Errors

Errors are returned as structured JSON.

Example:

{
  "error": "Track not found",
  "status": 404,
  "provider": "qobuz"
}

Common statuses:

Status	Meaning
200	Success
204	OPTIONS/CORS response
400	Invalid or missing parameter
404	Resource not found
405	HTTP method not supported
409	Requested direct stream is actually segmented
429	Upstream rate limit
500	Worker/configuration error
501	Capability intentionally unavailable/unverified
502	Upstream/provider failure
48. Quick-start examples
Search
https://YOUR-WORKER.workers.dev/search?q=Blinding%20Lights&type=track
Search without stream enrichment
https://YOUR-WORKER.workers.dev/search?q=Blinding%20Lights&type=track&include_stream=0
Track by ISRC
https://YOUR-WORKER.workers.dev/track?isrc=USUG11600976
24/192
https://YOUR-WORKER.workers.dev/stream?isrc=USUG11600976&quality=192
Best available
https://YOUR-WORKER.workers.dev/stream?isrc=USUG11600976&quality=best
Direct redirect
https://YOUR-WORKER.workers.dev/stream?isrc=USUG11600976&quality=best&stream=1
Health check
https://YOUR-WORKER.workers.dev/ping
49. Recommended Voria flow

For normal playback:

ISRC
  ↓
/track?isrc=...
  ↓
metadata + playback information
  ↓
playback

For browsing:

/search?q=...
  ↓
catalog results
  ↓
user selects track
  ↓
/track?id=...
  ↓
playback

For fast metadata-only searching:

/search?q=...&include_stream=0

Then resolve playback only after the user selects a track.

For autoplay:

current track
  ↓
/radio?id=...
  ↓
generated Qobuz catalog queue

For discovery:

/recommendations?q=...
50. Stability philosophy

This Worker intentionally favors:

verified behavior

over:

pretending every feature exists

Some routes therefore return 501.

For example:

/lyrics

and:

/album/similar

can intentionally report that the capability is unavailable through the stable API surface currently used by this Worker.

That is preferable to guessing an undocumented endpoint and having the entire API explode when Qobuz changes something.

The /info-api response exposes the Qobuz methods currently used internally.

51. Verified Qobuz API methods

The Worker is based around Qobuz methods that have been identified and tested for the relevant functionality.

These include:

track/search
album/search
artist/search
playlist/search
catalog/search

track/get
album/get
artist/page
artist/getSimilarArtists
artist/getReleasesList

playlist/get
playlist/getUserPlaylists

genre/list
album/getFeatured
most-popular/get

session/start
file/url
track/getFileUrl

The Worker intentionally does not rely on guessed endpoints such as:

radio/get
track/getRadio
track/getSimilar
catalog/getRecommendations
catalog/getFeatured
catalog/getCharts
chart/get

unless they are explicitly implemented and verified in a future version.

52. Qobuz API caveat

Qobuz can change its API, authentication behavior, playback system, signatures, or response formats.

Some playback functionality used by this Worker comes from behavior associated with the Qobuz web player rather than a guaranteed permanent public API contract.

Therefore, this project should be treated as a compatibility layer rather than a permanent guarantee that every endpoint will continue working forever.

53. Security

Never publish:

QOBUZ_APP_SECRET
QOBUZ_USER_AUTH_TOKEN
QOBUZ_USER_AUTH_TOKEN_2
QOBUZ_USER_AUTH_TOKEN_3
...

Do not include them in:

Git commits
GitHub Issues
Screenshots
Discord
Public logs
Error reports

If credentials are accidentally exposed, replace/rotate them.

54. Troubleshooting
/ping says catalog access failed

Check:

QOBUZ_APP_ID
QOBUZ_APP_SECRET
QOBUZ_USER_AUTH_TOKEN

Then run:

/ping

again.

Catalog works but playback fails

Check:

can_catalog
can_playback
can_lossless
playback_error
playback_signing_mode
playback_stream_type

A token can be valid for catalog access without being usable for playback.

/stream?stream=1 returns 409

Qobuz returned a segmented stream.

Inspect:

stream_type
url_template
n_segments

Do not assume that the URL template is a normal FLAC file URL.

Search is slow

If you do not need playback URLs for every result:

/search?q=...&include_stream=0

Track search with playback enrichment is more expensive because it has to resolve playback URLs.

KV does not appear to cache

Check that the Worker binding is exactly:

GENERAL_MUSIC_CACHE

The Worker can still function without KV, so a missing binding may not immediately look like a fatal configuration error.

Token works for catalog but not playback

Try another configured token.

You can also configure slot-specific credentials:

QOBUZ_APP_ID_2
QOBUZ_APP_SECRET_2
QOBUZ_USER_AUTH_TOKEN_2

The Worker can attempt cross-slot App credential fallback.

Everything suddenly breaks

Qobuz can change its API or web-player behavior.

Check:

/ping
Cloudflare Worker logs
/info-api
Recent repository commits
GitHub Issues
55. Disclaimer

This project is not an official Qobuz API client.

It is a compatibility layer using Qobuz catalog functionality and web-player playback behavior.

Qobuz can change its API, authentication, signing, playback flow, or response formats.

This documentation and the Worker were AI-generated with human-directed development and testing. They are not guaranteed to remain correct forever.

If you encounter a bug, broken endpoint, changed response, authentication failure, Cloudflare problem, or playback issue, please use the repository's GitHub Issues page when viewing the repo.

When opening an issue, include:

Endpoint
Query parameters, with secrets removed
HTTP status
Sanitized response
Cloudflare Worker logs
Expected behavior
Actual behavior

Never include your Qobuz App Secret, auth token, or other private credentials in an issue.
