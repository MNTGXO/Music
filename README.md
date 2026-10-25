# MN Music — Android + FastAPI + MongoDB

MN Music is a native Android music-player starter with an Apple Music-inspired Compose UI, local audio playback, background playback via Media3, YouTube search/discovery (metadata only through the official YouTube Data API), lyrics lookup through LRCLIB, and an optional FastAPI + MongoDB Atlas backend for app playlists and preferences.

## Important platform boundaries
- YouTube Data API search is used for discovery/metadata. This project does **not** extract YouTube audio streams, remove YouTube ads, or bypass YouTube playback restrictions. For YouTube playback, use an approved YouTube player integration and comply with its current terms.
- Background playback is implemented for local files and supported direct audio URLs using Android Media3.
- Lossless playback depends on the actual source/file and device output path. The app cannot turn lossy audio into lossless audio.
- This is a complete starter project, not a production-audited commercial release. Add tests, privacy policy, rate limits, OAuth hardening, and store compliance before publishing.

## Project layout
- `android/` — Kotlin, Jetpack Compose, Media3 playback service, local audio library, lyrics and YouTube discovery.
- `backend/` — FastAPI API, MongoDB Atlas persistence, YouTube search proxy, LRCLIB lyrics proxy.
- `docs/` — setup and architecture notes.

## 1. Backend setup
Requirements: Python 3.11+.

```bash
cd backend
python -m venv .venv
# Windows: .venv\Scripts\activate
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Edit `.env`:
- `MONGODB_URI` — MongoDB Atlas connection string
- `MONGODB_DB` — database name, e.g. `mn_music`
- `YOUTUBE_API_KEY` — optional YouTube Data API v3 key
- `CORS_ORIGINS` — comma-separated allowed origins (for development, `*`)

Run:
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```
Health check: `http://127.0.0.1:8000/health`
API docs: `http://127.0.0.1:8000/docs`

For Android emulator, the host machine is `http://10.0.2.2:8000/`. For a physical Android phone, use your computer's LAN IP or deploy HTTPS backend. Do not expose a development server publicly.

## 2. Android setup
Requirements: Android Studio (current stable), JDK 17, Android SDK 35.

1. Open `android/` in Android Studio.
2. Wait for Gradle sync.
3. In `android/app/build.gradle.kts`, set `BuildConfig.MN_API_BASE_URL` to your backend base URL:
   - Emulator: `http://10.0.2.2:8000/`
   - Physical phone: `http://YOUR_LAN_IP:8000/`
   - Production: HTTPS URL.
4. For local HTTP development, the app has cleartext enabled. Remove that for production and use HTTPS.
5. Press Run on an Android device/emulator.

YouTube search requires a valid `YOUTUBE_API_KEY` configured on the backend. Without it, local music still works.

## 3. Using the app
- **Listen** tab: scans audio exposed by Android MediaStore and plays selected tracks.
- **Search** tab: searches the backend for YouTube metadata. Search results open the YouTube video in the official YouTube app/browser; they are not audio-extracted.
- **Lyrics** tab: search LRCLIB using a track title and artist. Timestamped LRC lines are displayed as synced-style lyrics; the starter UI highlights based on playback position where metadata is available.
- **Playlists** tab: create/list simple cloud playlists (requires MongoDB backend).
- For direct audio URLs from a source you are authorized to use, use the `playUrl()` entry point in `MusicPlaybackService` / extend the UI with your own approved source provider.

## API overview
- `GET /health`
- `GET /api/youtube/search?q=...`
- `GET /api/lyrics?track_name=...&artist_name=...&duration=...`
- `GET /api/playlists?user_id=...`
- `POST /api/playlists`
- `PUT /api/preferences/{user_id}`

This starter uses a generated local anonymous user ID stored on device. For a real multi-user service, replace it with a proper authentication provider (OAuth/OIDC), validate tokens on every request, and never trust a client-supplied user ID.

## Production checklist
- Implement sign-in with a supported identity provider and server-side token validation.
- Add Android runtime permission rationale and automated UI/unit tests.
- Restrict CORS and API quotas; add rate limiting and request validation.
- Keep secrets only on the backend. Never embed the YouTube API key in the Android app.
- Use HTTPS, a privacy policy, deletion/export controls, monitoring and backups.
- Confirm all third-party API/player terms before launch.
