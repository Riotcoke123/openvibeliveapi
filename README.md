<div align="center">
# OpenVibe API
 
**Public documentation for the OpenVibe network API**
 
[![Website](https://img.shields.io/badge/site-openvibe.live-6f42c1?style=flat-square)](https://openvibe.live/)
[![Status](https://img.shields.io/badge/status-live-brightgreen?style=flat-square)](https://openvibe.live/)
[![License](https://img.shields.io/badge/license-unlicensed-lightgrey?style=flat-square)](#license)
 
[openvibe.live](https://openvibe.live/) · formerly [`hobostreamerapi`](https://github.com/Riotcoke123/hobostreamerapi)
 
</div>
---
 
## Table of Contents
 
- [Overview](#overview)
- [Base URLs](#base-urls)
- [Authentication](#authentication)
- [Endpoints](#endpoints)
  - [GET /api/updates](#get-apiupdates)
  - [GET /api/streams](#get-apistreams)
  - [GET /api/auth/refresh](#get-apiauthrefresh)
  - [GET /api/easter-egg/daily](#get-apieaster-eggdaily)
  - [OpenVibe.Games — GET /api/game/canvas/state](#openvibegames--get-apigamecanvasstate)
- [Error Handling](#error-handling)
- [Changelog Source](#changelog-source)
- [License](#license)
---
 
## Overview
 
The OpenVibe API powers [openvibe.live](https://openvibe.live/), a streaming/community network that tracks live streamers, publishes a commit-based changelog ("Arena" updates), runs daily easter-egg puzzles, and connects to a companion multiplayer games network at [openvibe.games](https://openvibe.games/).
 
This document covers the **publicly accessible** endpoints only. Endpoints that require an authenticated session (cookies/JWT) are noted as such.
 
> This repository was previously named `hobostreamerapi`.
 
## Base URLs
 
| Service | Base URL | Description |
|---|---|---|
| OpenVibe Live | `https://openvibe.live` | Core API — streams, updates, auth, easter eggs |
| OpenVibe Games | `https://openvibe.games` | Multiplayer browser games network, shares an OpenVibe account |
 
## Authentication
 
Most read-only endpoints below (`/api/updates`, `/api/streams`, `/api/easter-egg/daily`) are **public** and require no authentication.
 
Session-bound endpoints, such as `/api/auth/refresh`, expect an existing auth cookie/session. Calling them without one returns a `404 Not Found` rather than a `401`, so a `404` on an auth route usually means **no active session was sent**, not that the route doesn't exist.
 
```http
GET /api/auth/refresh HTTP/1.1
Host: openvibe.live
```
 
```json
{ "error": "Not found" }
```
 
---
 
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
 
---
 
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
 
---
 
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
 
---
 
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
 
---
 
### OpenVibe.Games — `GET /api/game/canvas/state`
 
Part of the companion [openvibe.games](https://openvibe.games/) network — free multiplayer games that run in the browser, sharing a single OpenVibe account across the network. No installs or launchers required.
 
**Example Request**
 
```http
GET /api/game/canvas/state HTTP/1.1
Host: openvibe.games
```
 
> This endpoint lives on a separate host (`openvibe.games`, not `openvibe.live`). Returns the current state of the shared multiplayer canvas game.
 
---
 
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
 
No license has been declared for this API. Treat all endpoints as read-only and public-facing unless noted otherwise; do not assume redistribution rights over response data.
 
---
 
<div align="center">
Made for [openvibe.live](https://openvibe.live/)
 
</div>
