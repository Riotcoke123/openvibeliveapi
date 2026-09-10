<div align="center">

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
  - [GET /api/auth/refresh](#get-apiauthrefresh)
  - [GET /api/easter-egg/daily](#get-apieaster-eggdaily)
  - [GET /api/chat/gif/providers](#get-apichatgifproviders)
  - [GET /api/home/pulse](#get-apihomepulse)
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

Made for [openvibe.live](https://openvibe.live/)

</div>
