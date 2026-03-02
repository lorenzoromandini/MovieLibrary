# Private Movie Library Download Service

## What This Is

A personal web-based movie collection manager and downloader for private use. The admin uploads movie files and provides identifying information (IMDb ID, TMDb ID, title/year), and the system automatically fetches rich metadata from online sources. End users (friends, family) browse a visual grid of movie posters, search/filter the collection, and download video files directly.

## Core Value

Admin uploads a movie file with minimal identifying info; the system automatically enriches it with complete metadata; end users can quickly browse, search, and download.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Admin can upload large video files (mp4, mkv, etc.) through web interface
- [ ] Admin can provide movie identifiers (IMDb ID, TMDb ID, IMDb URL, Wikipedia URL, or title+year)
- [ ] System fetches metadata from online sources (title, director, actors, year, runtime, genres, plot, poster)
- [ ] Admin can manually edit/correct any metadata field
- [ ] System stores movie files securely with associated metadata
- [ ] End users see visual grid of movie posters with titles and years
- [ ] Grid is responsive (adapts to screen size)
- [ ] Users can search by title, actors, director, genres
- [ ] Users can filter by year/range, genre, director, actor, rating
- [ ] Movie detail page shows full metadata and download link
- [ ] Direct file download (no streaming)
- [ ] Access remains private/controlled (no public registration)

### Out of Scope

- **Video streaming** — Explicitly download-only per requirements
- **Public registration** — Private/controlled access only
- **User accounts for end users** — Optional login mentioned, not required
- **Social features** — No sharing, commenting, ratings from users
- **Automatic download from external sources** — Admin manually downloads and uploads
- **Mobile app** — Web-only
- **Multiple language support beyond Italian** — Only original + Italian titles mentioned

## Context

**User Roles:**
- Admin: Single person (the owner) who uploads and manages content
- End Users: Friends, family, small controlled group who browse and download

**Key Workflows:**
1. Admin workflow: Upload file → Provide identifiers → System fetches metadata → Admin reviews/edits → Movie published
2. End user workflow: Browse grid → Search/filter → View details → Download file

**Metadata Sources Needed:**
- IMDb (via ID or URL)
- TMDb (The Movie Database)
- Wikipedia (as fallback)

**File Handling:**
- Large video files (mp4, mkv, etc.)
- Secure storage required
- Direct download delivery

**UI Inspiration:**
- Streaming catalog-style poster grid
- Clean, modern visual design
- Responsive layout (mobile to desktop)

## Constraints

- **Access Control**: Private/controlled only — no public registration
- **Delivery Mode**: Download only — no streaming functionality
- **Metadata Language**: Italian localization preferred where available
- **Storage**: Must handle large video files efficiently
- **Security**: Files must be stored securely, not publicly accessible

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| No streaming, download-only | Explicit requirement from user | — Pending |
| IMDb/TMDb as primary metadata sources | Industry standard, comprehensive APIs available | — Pending |
| Web-only (no mobile app) | Simpler deployment, matches private use case | — Pending |
| No end-user authentication required | Matches "optional login" requirement | — Pending |

---
*Last updated: 2025-03-02 after initialization*
