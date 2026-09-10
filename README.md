<div align="center">

<img width="553" height="241" alt="Untitled" src="https://github.com/user-attachments/assets/50f3dca2-04cd-409f-bfb2-e431f957ab13" />

**Public documentation for the OpenVibe network API**

[![Website](https://img.shields.io/badge/site-openvibe.live-6f42c1?style=flat-square)](https://openvibe.live/)
[![Status](https://img.shields.io/badge/status-live-brightgreen?style=flat-square)](https://openvibe.live/)
[![License](https://img.shields.io/badge/license-GPL--3.0-blue?style=flat-square)](https://github.com/Riotcoke123/openvibeliveapi/tree/main#)

[openvibe.live](https://openvibe.live/) · formerly [`hobostreamerapi`](https://github.com/Riotcoke123/hobostreamerapi)

</div>

## Table of Contents

- [Overview](#overview)
- [Base URLs](#base-urls)
- [Authentication](#authentication)
- [Endpoints](#endpoints)
  - [GET /api/updates](#get-apiupdates)
  - [GET /api/streams](#get-apistreams)
  - [GET /api/streams/recently-online](#get-apistreamsrecently-online)
  - [GET /api/streams/recent-vods](#get-apistreamsrecent-vods)
  - [GET /api/pastes](#get-apipastes)
  - [GET /api/clips](#get-apiclips)
  - [GET /api/auth/refresh](#get-apiauthrefresh)
  - [GET /api/easter-egg/daily](#get-apieaster-eggdaily)
  - [GET /api/chat/gif/providers](#get-apichatgifproviders)
  - [GET /api/home/pulse](#get-apihomepulse)
  - [GET /api/home/hero](#get-apihomehero)
  - [GET /api/live-events](#get-apilive-events)
  - [OpenVibe.Games — GET /api/game/canvas/state](#openvibegames--get-apigamecanvasstate)
  - [OpenVibe.Games — GET /api/game/leaderboard/mining](#openvibegames--get-apigameleaderboardmining)
- [Error Handling](#error-handling)
- [Changelog Source](#changelog-source)
- [License](#license)

## Overview

The OpenVibe API powers [openvibe.live](https://openvibe.live/), a streaming/community network that tracks live streamers, publishes a commit-based changelog ("Arena" updates), runs daily easter-egg puzzles, streams live site events over SSE, and connects to a companion multiplayer games network at [openvibe.games](https://openvibe.games/).

This document covers the **publicly accessible** endpoints only. Endpoints that require an authenticated session (cookies/JWT) are noted as such.

> This repository was previously named `hobostreamerapi`.

## Base URLs

| Service | Base URL | Description |
|---|---|---|
| OpenVibe Live | `https://openvibe.live` | Core API — streams, updates, auth, chat, home pulse, live events |
| OpenVibe Games | `https://openvibe.games` | Multiplayer browser games network, shares an OpenVibe account |

## Authentication

Most read-only endpoints below (`/api/updates`, `/api/streams`, `/api/easter-egg/daily`, `/api/chat/gif/providers`, `/api/home/pulse`, `/api/live-events`) are **public** and require no authentication.

Session-bound endpoints, such as `/api/auth/refresh`, expect an existing auth cookie/session. Calling them without one returns a `404 Not Found` rather than a `401`, so a `404` on an auth route usually means **no active session was sent**, not that the route doesn't exist.

```http
GET /api/auth/refresh HTTP/1.1
Host: openvibe.live
```

```json
{ "error": "Not found" }
```

## Endpoints

### `GET /api/updates`

Returns the site's recent commit history, used to power the public "what's new" / Arena changelog feed.

**Query Parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `limit` | integer | No | Number of commits to return. Defaults to a server-defined value if omitted. |

**Example Request**

```http
GET /api/updates?limit=15 HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{
  "commits": [
    {
      "hash": "b81694173a3836f72345805a22b03db8ab5f36be",
      "short": "b816941",
      "subject": "TTS playback fix (user report): media-src data:, same-origin clip URLs for tests/previews, blob→data retry",
      "author": "Alex",
      "date": "2026-08-31T16:45:18-07:00"
    },
    {
      "hash": "898b30e25ed078ce686eefda67d827f723712aab",
      "short": "898b30e",
      "subject": "Arena: taunt bubble grid areas (speaker beside the text)",
      "author": "Alex",
      "date": "2026-08-30T11:01:12-07:00"
    }
  ]
}
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `commits` | array | List of commit objects, most recent first |
| `commits[].hash` | string | Full commit SHA |
| `commits[].short` | string | Abbreviated (7-char) commit SHA |
| `commits[].subject` | string | Commit message subject line |
| `commits[].author` | string | Commit author's display name |
| `commits[].date` | string (ISO 8601) | Commit timestamp, including timezone offset |

### `GET /api/streams`

Returns the current list of tracked streamers/streams across supported platforms.

**Example Request**

```http
GET /api/streams HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{ "streams": [] }
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `streams` | array | List of currently tracked stream objects. Empty when no one is live. |

> The shape of individual stream objects (platform, title, viewer count, thumbnail, live status, etc.) is populated when one or more streams are active; an empty array is returned when nothing is currently live.

### `GET /api/streams/recently-online`

Returns a paginated list of streamers, ordered by recency, along with their managed streams, an AI-generated content overview, and their top active funding goal (if any).

**Query Parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `limit` | integer | No | Number of streamers to return per page. |
| `offset` | integer | No | Number of streamers to skip, for pagination. |

**Example Request**

```http
GET /api/streams/recently-online?limit=12&offset=0 HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{
  "streamers": [
    {
      "user_id": 1,
      "username": "goosely",
      "display_name": "Goosely",
      "avatar_url": "https://openvibe.media/f/screenshots/avatar-1-1786275980625-04833b19.png",
      "profile_color": "#c0965c",
      "last_online_at": "2026-09-10 22:03:09",
      "ai_overview": "Goosely is an IRL streamer who blends live in-person interactions with tech- and retro-computer aesthetics...",
      "ai_overview_short": "Goosely is an IRL streamer who blends live in-person interactions with tech- and retro-computer aesthetics...",
      "managed_streams": [
        {
          "managed_stream_id": 1,
          "slug": "whip",
          "title": "OpenVibe.Live + PowerChat.Live",
          "protocol": "webrtc",
          "last_live_at": "2026-09-10 22:03:09",
          "vod_thumbnail": "https://openvibe.media/t/vod-2580-1787936300946.jpg"
        }
      ],
      "top_goal": {
        "title": "Raspi Robot",
        "current": 100,
        "target": 8000
      }
    }
  ],
  "total": 71,
  "limit": 12,
  "offset": 0,
  "hasMore": true
}
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `streamers` | array | Page of streamer objects, most recently online first |
| `streamers[].user_id` | integer | Streamer's internal user ID |
| `streamers[].username` | string | Streamer's username |
| `streamers[].display_name` | string | Streamer's display name |
| `streamers[].avatar_url` | string \| null | Avatar image URL |
| `streamers[].profile_color` | string | Hex color associated with the streamer's profile |
| `streamers[].last_online_at` | string | Timestamp the streamer was last online |
| `streamers[].ai_overview` | string \| null | Full AI-generated summary of the streamer's content |
| `streamers[].ai_overview_short` | string \| null | Truncated version of `ai_overview` |
| `streamers[].managed_streams` | array | Streams/channels managed under this streamer |
| `streamers[].managed_streams[].managed_stream_id` | integer | Managed stream ID |
| `streamers[].managed_streams[].slug` | string \| null | URL slug for the stream, if set |
| `streamers[].managed_streams[].title` | string | Stream title |
| `streamers[].managed_streams[].protocol` | string | Streaming protocol (`webrtc`, `rtmp`) |
| `streamers[].managed_streams[].last_live_at` | string | Timestamp this managed stream was last live |
| `streamers[].managed_streams[].vod_thumbnail` | string \| null | Thumbnail URL for the latest VOD, if available |
| `streamers[].top_goal` | object (optional) | The streamer's top active funding goal, if any |
| `streamers[].top_goal.title` | string | Goal title |
| `streamers[].top_goal.current` | number | Amount currently raised |
| `streamers[].top_goal.target` | number | Goal target amount |
| `total` | integer | Total number of streamers available across all pages |
| `limit` | integer | Page size used for this response |
| `offset` | integer | Offset used for this response |
| `hasMore` | boolean | Whether additional pages are available beyond this one |

### `GET /api/streams/recent-vods`

Returns a paginated list of the most recently created VODs (recorded broadcasts) across the network.

**Query Parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `limit` | integer | No | Number of VODs to return per page. |
| `offset` | integer | No | Number of VODs to skip, for pagination. |

**Example Request**

```http
GET /api/streams/recent-vods?limit=12&offset=0 HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{
  "vods": [
    {
      "unique_views": 2,
      "id": 3384,
      "app_id": "live",
      "stream_id": 2341,
      "managed_stream_id": 104,
      "user_id": 327,
      "title": "JapaneseOldGuy's Stream",
      "description": "",
      "status": "ready",
      "duration": 13427,
      "duration_seconds": 13427,
      "file_size": 10366166967,
      "file_path": "vod-live-3384-1789050233492.mp4",
      "playback_url": "https://openvibe.media/v/3384",
      "thumbnail_url": "https://openvibe.media/t/vod-3384-1789063683687.jpg",
      "storage_provider": "local",
      "visibility": "public",
      "is_public": true,
      "health_status": "ok",
      "clips_only": false,
      "is_recording": false,
      "ai_overview": null,
      "ai_analyzed_at": null,
      "view_count": 2,
      "created_at": "2026-09-10 14:23:53",
      "meta": {
        "protocol": "rtmp",
        "mode": "vod",
        "clips_only": false,
        "part": 1
      },
      "username": "JapaneseOldGuy",
      "display_name": "JapaneseOldGuy",
      "avatar_url": "https://openvibe.media/p/safe-frost-74/raw"
    }
  ],
  "total": 595,
  "limit": 12,
  "offset": 0,
  "hasMore": true
}
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `vods` | array | Page of VOD objects, most recently created first |
| `vods[].id` | integer | VOD ID |
| `vods[].unique_views` | integer | Count of unique viewers |
| `vods[].view_count` | integer | Total view count |
| `vods[].app_id` | string | Application/stream namespace (e.g. `live`) |
| `vods[].stream_id` | integer | ID of the originating live stream session |
| `vods[].managed_stream_id` | integer | ID of the managed stream this VOD belongs to |
| `vods[].user_id` | integer | Streamer's internal user ID |
| `vods[].username` / `display_name` | string | Streamer identity |
| `vods[].avatar_url` | string \| null | Streamer's avatar image URL |
| `vods[].title` | string | VOD title |
| `vods[].description` | string | VOD description (may be empty) |
| `vods[].status` | string | Processing status (e.g. `ready`) |
| `vods[].duration` / `duration_seconds` | integer | VOD length in seconds |
| `vods[].file_size` | integer | File size in bytes |
| `vods[].file_path` | string | Storage path/filename of the VOD file |
| `vods[].playback_url` | string | Public playback URL |
| `vods[].thumbnail_url` | string | Thumbnail image URL |
| `vods[].storage_provider` | string | Backing storage provider (e.g. `local`, `b2`) |
| `vods[].visibility` | string | Visibility setting (e.g. `public`) |
| `vods[].is_public` | boolean | Whether the VOD is publicly viewable |
| `vods[].health_status` | string | File/processing health check result |
| `vods[].clips_only` | boolean | Whether the VOD only contains clips |
| `vods[].is_recording` | boolean | Whether the VOD is still actively recording |
| `vods[].ai_overview` | string \| null | AI-generated summary of the VOD content, if analyzed |
| `vods[].ai_analyzed_at` | string \| null | Timestamp the AI analysis was run |
| `vods[].created_at` | string | Timestamp the VOD was created |
| `vods[].meta` | object | Additional metadata (protocol, mode, part number, etc.) |
| `total` | integer | Total number of VODs available across all pages |
| `limit` | integer | Page size used for this response |
| `offset` | integer | Offset used for this response |
| `hasMore` | boolean | Whether additional pages are available beyond this one |

### `GET /api/pastes`

Returns a paginated list of "pastes" — user-generated screenshots (and other paste types) captured from streams, each with AI-generated tags and a summary.

**Query Parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `limit` | integer | No | Number of pastes to return per page. |
| `offset` | integer | No | Number of pastes to skip, for pagination. |

**Example Request**

```http
GET /api/pastes?limit=10&offset=0 HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{
  "pastes": [
    {
      "unique_views": 2,
      "id": 901,
      "app_id": "live",
      "slug": "hot-jay-77",
      "user_id": 1,
      "type": "screenshot",
      "title": "Stream Sparks with Chat Buzz",
      "content": null,
      "language": "text",
      "visibility": "public",
      "stream_id": 2342,
      "screenshot_url": "/p/hot-jay-77/screenshot",
      "metadata": "{\"page_url\":null,\"user_agent\":\"node\",\"original_name\":\"live-2342-132.jpg\",\"size_bytes\":35611,\"mime_type\":\"image/jpeg\"}",
      "burn_after_read": false,
      "forked_from": null,
      "pinned": false,
      "views": 2,
      "copies": 0,
      "likes": 0,
      "is_nsfw": false,
      "ai_summary": "A streaming dashboard interface showing a chat buzz stream with colored tag-like buttons and a dark layout, plus a webcam feed of a person in the bottom-left corner.",
      "ai_tags": "[\"stream\",\"dashboard\",\"chat\",\"tags\",\"webcam\",\"live\"]",
      "ai_analyzed_at": "2026-09-10 21:46:40",
      "url": "/p/hot-jay-77",
      "raw_url": "/p/hot-jay-77/raw",
      "created_at": "2026-09-10 21:46:15",
      "updated_at": "2026-09-10 21:46:15",
      "username": "goosely",
      "display_name": "Goosely",
      "profile_color": "#c0965c",
      "avatar_url": "https://openvibe.media/f/screenshots/avatar-1-1786275980625-04833b19.png"
    }
  ],
  "total": 683,
  "limit": 10,
  "offset": 0
}
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `pastes` | array | Page of paste objects, most recent first |
| `pastes[].id` | integer | Paste ID |
| `pastes[].slug` | string | URL slug for the paste (used in `url`/`raw_url`) |
| `pastes[].app_id` | string | Application/stream namespace (e.g. `live`) |
| `pastes[].user_id` | integer | Author's internal user ID |
| `pastes[].username` / `display_name` | string | Author identity |
| `pastes[].profile_color` | string | Hex color associated with the author's profile |
| `pastes[].avatar_url` | string \| null | Author's avatar image URL |
| `pastes[].type` | string | Paste type (e.g. `screenshot`) |
| `pastes[].title` | string | Paste title |
| `pastes[].content` | string \| null | Text content, for text-type pastes |
| `pastes[].language` | string | Syntax/content language (e.g. `text`) |
| `pastes[].visibility` | string | Visibility setting (e.g. `public`) |
| `pastes[].stream_id` | integer \| null | Originating live stream session, if captured from one |
| `pastes[].screenshot_url` | string | Relative URL to the screenshot image |
| `pastes[].metadata` | string (JSON) | Stringified JSON blob with capture metadata (`page_url`, `user_agent`, `original_name`, `size_bytes`, `mime_type`) |
| `pastes[].burn_after_read` | boolean | Whether the paste is deleted after first view |
| `pastes[].forked_from` | integer \| null | ID of the paste this was forked from, if any |
| `pastes[].pinned` | boolean | Whether the paste is pinned |
| `pastes[].views` / `unique_views` | integer | Total and unique view counts |
| `pastes[].copies` | integer | Number of times the paste content was copied |
| `pastes[].likes` | integer | Like count |
| `pastes[].is_nsfw` | boolean | Whether the paste is flagged NSFW |
| `pastes[].ai_summary` | string \| null | AI-generated description of the paste content |
| `pastes[].ai_tags` | string (JSON array) | Stringified JSON array of AI-generated tags |
| `pastes[].ai_analyzed_at` | string \| null | Timestamp the AI analysis was run |
| `pastes[].url` | string | Relative URL to view the paste |
| `pastes[].raw_url` | string | Relative URL to the raw asset |
| `pastes[].created_at` / `updated_at` | string | Timestamps for creation and last update |
| `total` | integer | Total number of pastes available across all pages |
| `limit` | integer | Page size used for this response |
| `offset` | integer | Offset used for this response |

### `GET /api/clips`

Returns a paginated list of clips (short highlight cuts from VODs), each with playback info and, when processed, an AI-generated overview.

**Query Parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `limit` | integer | No | Number of clips to return per page. |
| `offset` | integer | No | Number of clips to skip, for pagination. |

**Example Request**

```http
GET /api/clips?limit=12&offset=0 HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{
  "clips": [
    {
      "unique_views": 3,
      "id": 349,
      "app_id": "live",
      "vod_id": 3122,
      "stream_id": 2335,
      "channel_user_id": 80,
      "user_id": 80,
      "title": "A Computer Screen Shows A Discord/Stream Chat Window",
      "description": "",
      "status": "ready",
      "start_time": 4718,
      "end_time": 4732.427,
      "duration": 14.427,
      "duration_seconds": 14.427,
      "file_path": "clip-1788950355082-wmz87p.webm",
      "playback_url": "https://openvibe.media/c/349",
      "thumbnail_url": "https://openvibe.media/t/clip-349-1788950504280.jpg",
      "visibility": "public",
      "is_public": true,
      "storage_provider": "local",
      "auto_generated": false,
      "ai_overview": "The video presents a chaotic, multi-monitor livestream layout that looks like a futuristic cockpit...",
      "ai_analyzed_at": null,
      "view_count": 3,
      "created_at": "2026-09-09 10:39:15",
      "username": "Maticus",
      "display_name": "Maticus",
      "avatar_url": "https://openvibe.media/f/screenshots/avatar-80-1786485794298-0a925ac2.jpg",
      "profile_color": "#17722e",
      "source_streamer_id": 80,
      "source_streamer_username": "Maticus",
      "source_streamer_display_name": "Maticus",
      "streamer_username": "Maticus",
      "streamer_display_name": "Maticus",
      "streamer_avatar_url": "https://openvibe.media/f/screenshots/avatar-80-1786485794298-0a925ac2.jpg",
      "ai_overview_short": "The video presents a chaotic, multi-monitor livestream layout that looks like a futuristic cockpit, with several windows, chat, and memes arranged around the screen."
    }
  ],
  "total": 294,
  "limit": 12,
  "offset": 0,
  "hasMore": true,
  "streamers": [],
  "activeFilter": null
}
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `clips` | array | Page of clip objects, most recent first |
| `clips[].id` | integer | Clip ID |
| `clips[].vod_id` | integer | ID of the source VOD the clip was cut from |
| `clips[].stream_id` | integer | ID of the originating live stream session |
| `clips[].app_id` | string | Application/stream namespace (e.g. `live`) |
| `clips[].channel_user_id` / `user_id` | integer | Owning channel / clip creator's user ID |
| `clips[].username` / `display_name` | string | Clip creator identity |
| `clips[].profile_color` | string | Hex color associated with the creator's profile |
| `clips[].avatar_url` | string | Creator's avatar image URL |
| `clips[].source_streamer_id` / `_username` / `_display_name` | mixed | Identity of the streamer the clip was sourced from |
| `clips[].streamer_username` / `streamer_display_name` / `streamer_avatar_url` | string | Display identity shown for the clip's streamer |
| `clips[].title` | string | Clip title |
| `clips[].description` | string | Clip description (may be empty) |
| `clips[].status` | string | Processing status (`ready` or `failed`) |
| `clips[].start_time` / `end_time` | number | Offsets (seconds) into the source VOD marking the clip bounds |
| `clips[].duration` / `duration_seconds` | number | Clip length in seconds |
| `clips[].file_path` | string \| null | Storage filename; `null` if processing failed |
| `clips[].playback_url` | string | Public playback URL |
| `clips[].thumbnail_url` | string \| null | Thumbnail image URL; `null` if processing failed |
| `clips[].visibility` | string | Visibility setting (e.g. `public`) |
| `clips[].is_public` | boolean | Whether the clip is publicly viewable |
| `clips[].storage_provider` | string | Backing storage provider (e.g. `local`) |
| `clips[].auto_generated` | boolean | Whether the clip was auto-generated rather than manually cut |
| `clips[].ai_overview` / `ai_overview_short` | string \| null | Full and truncated AI-generated summaries of the clip content |
| `clips[].ai_analyzed_at` | string \| null | Timestamp the AI analysis was run |
| `clips[].view_count` / `unique_views` | integer | Total and unique view counts |
| `clips[].created_at` | string | Timestamp the clip was created |
| `total` | integer | Total number of clips available across all pages |
| `limit` | integer | Page size used for this response |
| `offset` | integer | Offset used for this response |
| `hasMore` | boolean | Whether additional pages are available beyond this one |
| `streamers` | array | Streamer filter options (empty when unfiltered) |
| `activeFilter` | string \| null | Currently applied streamer filter, if any |

### `GET /api/auth/refresh`

Refreshes an authenticated session/token. **Requires an active session** — this is not a public/anonymous endpoint.

**Example Request (no session)**

```http
GET /api/auth/refresh HTTP/1.1
Host: openvibe.live
```

**Example Response — `404 Not Found`**

```json
{ "error": "Not found" }
```

| Field | Type | Description |
|---|---|---|
| `error` | string | Error message. `"Not found"` is returned when no valid session is present. |

### `GET /api/easter-egg/daily`

Returns metadata for the site's daily easter-egg puzzle.

**Example Request**

```http
GET /api/easter-egg/daily HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{
  "egg": {
    "date": "2026-09-10",
    "title": "The Daily Secret",
    "hints": [
      "The old gamers knew: four winds and a few good letters open the way.",
      "Watch the arrows and whisper the letters, in the order the day decided.",
      "No shame in guessing — the brave stumble onto it."
    ],
    "codeLength": 8,
    "effect": "rainbow",
    "nextResetAt": 1789084800000,
    "foundCount": 0,
    "solved": false
  }
}
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `egg.date` | string (`YYYY-MM-DD`) | Date the puzzle applies to |
| `egg.title` | string | Puzzle title |
| `egg.hints` | array of strings | Ordered list of hints |
| `egg.codeLength` | integer | Expected length of the solution code |
| `egg.effect` | string | Visual reward effect shown on solve |
| `egg.nextResetAt` | integer (Unix ms) | Timestamp when the next puzzle becomes available |
| `egg.foundCount` | integer | Number of users who have solved today's puzzle |
| `egg.solved` | boolean | Whether the requesting session has already solved it |

### `GET /api/chat/gif/providers`

Returns which GIF providers are currently enabled for chat, and which one is used by default.

**Example Request**

```http
GET /api/chat/gif/providers HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{
  "providers": {
    "tenor": false,
    "giphy": false
  },
  "defaultProvider": null
}
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `providers` | object | Map of provider name → enabled status |
| `providers.tenor` | boolean | Whether Tenor GIF search is enabled |
| `providers.giphy` | boolean | Whether Giphy GIF search is enabled |
| `defaultProvider` | string \| null | The default provider used for GIF search, or `null` if none are enabled |

### `GET /api/home/pulse`

Returns a snapshot of homepage activity: active donation goals, the latest tip, the newest follow, top supporters/earners, notable moments, and the latest changelog entry.

**Example Request**

```http
GET /api/home/pulse HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{
  "goals": [
    {
      "id": 1,
      "title": "Raspi Robot",
      "current_amount": 100,
      "target_amount": 8000,
      "image_url": "/data/offline/goal-1-mso4lvgt.webp",
      "username": "goosely",
      "display_name": "Goosely"
    },
    {
      "id": 2,
      "title": "AI API Upgrades",
      "current_amount": 0,
      "target_amount": 3000,
      "image_url": "/data/offline/goal-1-mso4ntl5.webp",
      "username": "goosely",
      "display_name": "Goosely"
    },
    {
      "id": 3,
      "title": "Daily Goal",
      "current_amount": 0,
      "target_amount": 5000,
      "image_url": null,
      "username": "JapaneseOldGuy",
      "display_name": "JapaneseOldGuy"
    }
  ],
  "latestTip": {
    "amount": 100,
    "created_at": "2026-08-27 03:00:14",
    "from_username": "goosely",
    "from_display": "Goosely",
    "to_username": "goosely",
    "to_display": "Goosely"
  },
  "newestFollow": {
    "created_at": "2026-09-10 08:54:34",
    "follower_username": "Gupta_Khan",
    "follower_display": "Gupta_Khan",
    "streamer_username": "goosely",
    "streamer_display": "Goosely"
  },
  "topSupporters": [],
  "topEarners": [
    {
      "username": "Maticus",
      "display_name": "Maticus",
      "avatar_url": "https://openvibe.media/f/screenshots/avatar-80-1786485794298-0a925ac2.jpg",
      "total": 4710
    },
    {
      "username": "Little_Buddy",
      "display_name": "Little_Buddy",
      "avatar_url": "https://openvibe.media/f/screenshots/avatar-309-1786894249817-11d48f77.png",
      "total": 275
    },
    {
      "username": "Gupta_Khan",
      "display_name": "Gupta_Khan",
      "avatar_url": null,
      "total": 265
    }
  ],
  "moments": [],
  "latestUpdate": {
    "short": "b816941",
    "subject": "TTS playback fix (user report): media-src data:, same-origin clip URLs for tests/previews, blob→data retry",
    "date": "2026-08-31T16:45:18-07:00"
  }
}
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `goals` | array | Active donation/funding goals |
| `goals[].id` | integer | Goal ID |
| `goals[].title` | string | Goal title |
| `goals[].current_amount` | number | Amount currently raised |
| `goals[].target_amount` | number | Goal target amount |
| `goals[].image_url` | string \| null | Goal image, or `null` if none set |
| `goals[].username` / `display_name` | string | Streamer the goal belongs to |
| `latestTip` | object | Most recent tip sent site-wide |
| `latestTip.amount` | number | Tip amount |
| `latestTip.created_at` | string | Timestamp of the tip |
| `latestTip.from_username` / `from_display` | string | Sender |
| `latestTip.to_username` / `to_display` | string | Recipient |
| `newestFollow` | object | Most recent follow site-wide |
| `newestFollow.follower_username` / `follower_display` | string | Follower |
| `newestFollow.streamer_username` / `streamer_display` | string | Streamer being followed |
| `topSupporters` | array | Top tip senders (may be empty) |
| `topEarners` | array | Top tip recipients |
| `topEarners[].username` / `display_name` | string | Earner identity |
| `topEarners[].avatar_url` | string \| null | Avatar image URL |
| `topEarners[].total` | number | Total amount earned |
| `moments` | array | Notable site moments (may be empty) |
| `latestUpdate` | object | Most recent changelog entry (see [`/api/updates`](#get-apiupdates)) |

### `GET /api/home/hero`

Returns the data behind the homepage hero section: network-wide stats (with daily/weekly/monthly deltas), a viewer trend timeseries, a rotating media showcase (clips/VODs/pastes), notable moments, and rotating AI-generated slogans.

**Example Request**

```http
GET /api/home/hero HTTP/1.1
Host: openvibe.live
```

**Example Response — `200 OK`**

```json
{
  "stats": {
    "viewersNow": 0,
    "hoursWatched": 4159,
    "vibesTipped": 0,
    "activeSubs": 0,
    "pointsEarned": 789800,
    "pointsSpent": 575,
    "redemptions": 0,
    "supporters": 0,
    "vibesBought": 500,
    "goalsActive": 3,
    "goalsReached": 0,
    "vods": 595,
    "clips": 294,
    "liveSessions": 2342,
    "streamers": 71,
    "chatMessages": 68706,
    "users": 332,
    "anons": 27628,
    "follows": 62,
    "emotes": 40,
    "pastes": 693,
    "aiMemories": 13070,
    "pasteImages": 627,
    "pasteText": 66,
    "streamHours": 341,
    "weeklyActive": 35,
    "weeklyVisitors": 89,
    "liveNow": 0,
    "recent": {
      "users": { "d": 0, "w": 5, "m": 30 },
      "messages": { "d": 264, "w": 2394, "m": 12070 },
      "follows": { "d": 1, "w": 1, "m": 3 }
    },
    "viewerTrend": [
      { "t": "1788996218", "viewers": 0, "live_streams": 0 },
      { "t": "1789001618", "viewers": 2, "live_streams": 1 }
    ]
  },
  "media": [
    {
      "kind": "clip",
      "title": "A Man Wearing A Hat Stands At The",
      "thumbnail": "https://openvibe.media/t/clip-339-1788698827505.jpg",
      "href": "/clip/339"
    },
    {
      "kind": "vod",
      "title": "JapaneseOldGuy's Stream",
      "thumbnail": "https://openvibe.media/t/vod-3032-1788961342285.jpg",
      "href": "/vod/3032"
    },
    {
      "kind": "paste",
      "title": "When Repo Teases a Tiny Stream",
      "thumbnail": "https://openvibe.media/p/dusk-horse-18/screenshot",
      "text": null,
      "href": "/p/dusk-horse-18"
    }
  ],
  "moments": [],
  "slogans": {
    "audiences": ["finditfixit rant observers", "test-obs standby crew", "castroedwin IRL watchers"],
    "quips": ["we ship what we test, proudly", "AI takes notes, humans take the memes"],
    "ai": true,
    "updated_at": 1789070974804,
    "next_at": 1789092574804
  }
}
```

**Response Fields**

| Field | Type | Description |
|---|---|---|
| `stats` | object | Network-wide aggregate statistics |
| `stats.viewersNow` / `liveNow` | integer | Current live viewer count / current number of live streams |
| `stats.hoursWatched` / `streamHours` | integer | Total hours watched / total hours streamed |
| `stats.vibesTipped` / `vibesBought` | integer | Total tip currency ("vibes") tipped / purchased |
| `stats.activeSubs` / `supporters` | integer | Active subscriber and supporter counts |
| `stats.pointsEarned` / `pointsSpent` | integer | Total loyalty points earned / spent site-wide |
| `stats.redemptions` | integer | Total point redemptions |
| `stats.goalsActive` / `goalsReached` | integer | Count of currently active / completed funding goals |
| `stats.vods` / `clips` / `pastes` | integer | Total counts of VODs, clips, and pastes |
| `stats.liveSessions` | integer | Total historical live session count |
| `stats.streamers` / `users` / `anons` | integer | Streamer, registered user, and anonymous visitor counts |
| `stats.chatMessages` | integer | Total chat messages sent site-wide |
| `stats.follows` / `emotes` | integer | Total follows and custom emotes |
| `stats.aiMemories` | integer | Total AI-generated "memories"/moments recorded |
| `stats.pasteImages` / `pasteText` | integer | Breakdown of pastes by type |
| `stats.weeklyActive` / `weeklyVisitors` | integer | Weekly active user and visitor counts |
| `stats.recent` | object | Per-metric deltas keyed by metric name, each with `d` (day), `w` (week), `m` (month) counts |
| `stats.viewerTrend` | array | Timeseries of viewer/live-stream counts, one point per interval |
| `stats.viewerTrend[].t` | string (Unix seconds) | Timestamp of the data point |
| `stats.viewerTrend[].viewers` | integer | Concurrent viewers at that point |
| `stats.viewerTrend[].live_streams` | integer | Concurrent live streams at that point |
| `media` | array | Rotating showcase of recent clips, VODs, and pastes |
| `media[].kind` | string | Media type: `clip`, `vod`, or `paste` |
| `media[].title` | string | Media title |
| `media[].thumbnail` | string | Thumbnail image URL |
| `media[].text` | string \| null | Text content, for text-type pastes |
| `media[].href` | string | Relative link to the media item |
| `moments` | array | Notable AI-flagged moments (may be empty) |
| `slogans` | object | Rotating homepage tagline data |
| `slogans.audiences` | array of strings | Pool of audience-callout phrases |
| `slogans.quips` | array of strings | Pool of slogan/quip phrases |
| `slogans.ai` | boolean | Whether slogans are AI-generated |
| `slogans.updated_at` | integer (Unix ms) | When the current slogan set was generated |
| `slogans.next_at` | integer (Unix ms) | When the next slogan rotation will occur |

### `GET /api/live-events`

Server-Sent Events (SSE) stream of real-time site events (stream go-live notifications, heartbeats, etc.). Keep the connection open and read events as they arrive.

**Example Request**

```http
GET /api/live-events HTTP/1.1
Host: openvibe.live
Accept: text/event-stream
```

**Example Stream**

```
retry: 10000

: connected

: hb

event: stream-live
data: {"repeat":false,"username":"goosely","display_name":"Goosely","avatar_url":"https://openvibe.media/f/screenshots/avatar-1-1786275980625-04833b19.png","title":"OpenVibe.Live + PowerChat.Live","stream_id":2343,"slug":"whip","managed_id":1,"at":1789082807661}

: hb
```

**Event Types**

| Event | Description |
|---|---|
| `retry` | Reconnection interval in milliseconds sent by the server |
| `: connected` | Comment line confirming the connection is established |
| `: hb` | Heartbeat comment sent periodically to keep the connection alive |
| `stream-live` | Fired when a streamer goes live |

**`stream-live` Payload Fields**

| Field | Type | Description |
|---|---|---|
| `repeat` | boolean | Whether this is a repeat notification for an already-live stream |
| `username` | string | Streamer's username |
| `display_name` | string | Streamer's display name |
| `avatar_url` | string | Streamer's avatar image URL |
| `title` | string | Stream title |
| `stream_id` | integer | Internal stream ID |
| `slug` | string | URL slug for the stream |
| `managed_id` | integer | Managed stream identifier |
| `at` | integer (Unix ms) | Timestamp the event was fired |

### OpenVibe.Games — `GET /api/game/canvas/state`

Part of the companion [openvibe.games](https://openvibe.games/) network — free multiplayer games that run in the browser, sharing a single OpenVibe account across the network. No installs or launchers required.

**Example Request**

```http
GET /api/game/canvas/state HTTP/1.1
Host: openvibe.games
```

> This endpoint lives on a separate host (`openvibe.games`, not `openvibe.live`). Returns the current state of the shared multiplayer canvas game.

### OpenVibe.Games — `GET /api/game/leaderboard/mining`

Returns the leaderboard for the "mining" game on the OpenVibe.Games network.

**Example Request**

```http
GET /api/game/leaderboard/mining HTTP/1.1
Host: openvibe.games
```

> This endpoint lives on a separate host (`openvibe.games`, not `openvibe.live`). Returns ranked player standings for the mining game.

## Error Handling

| Status | Meaning |
|---|---|
| `200` | Request succeeded |
| `404` | Route not found, **or** an auth-gated route was called without a valid session (see [`/api/auth/refresh`](#get-apiauthrefresh)) |

Errors are returned as JSON in the form:

```json
{ "error": "<message>" }
```

## Changelog Source

The [`/api/updates`](#get-apiupdates) feed is generated directly from the live git commit history of the OpenVibe codebase, so it reflects real shipped changes rather than a hand-maintained changelog.

## License

Licensed under [GPL-3.0](https://github.com/Riotcoke123/openvibeliveapi/tree/main#).

<div align="center">

Made for [openvibe.live](https://openvibe.live/) 🐐

</div>
