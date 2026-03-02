# Requirements: Private Movie Library Download Service

**Defined:** 2025-03-02
**Core Value:** Admin uploads a movie file with minimal identifying info; the system automatically enriches it with complete metadata; end users can quickly browse, search, and download.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### File Upload & Storage

- [ ] **UPLOAD-01**: Admin can upload video files (mp4, mkv, etc.) up to 50GB through web interface
- [ ] **UPLOAD-02**: Upload shows progress indicator (percentage and estimated time remaining)
- [ ] **UPLOAD-03**: System validates file type (whitelist: mp4, mkv, avi, mov, webm)
- [ ] **UPLOAD-04**: Files are stored securely outside web root with cryptographically secure IDs
- [ ] **UPLOAD-05**: Failed uploads can be resumed without starting over

### Metadata Management

- [ ] **META-01**: System fetches movie metadata from TMDb using title/year or TMDb ID
- [ ] **META-02**: System fetches metadata from OMDb (IMDb data) as fallback
- [ ] **META-03**: Fetched metadata includes: title, Italian title, director, actors (top 10), year, runtime, genres, plot summary, poster URL
- [ ] **META-04**: System downloads and caches poster image locally
- [ ] **META-05**: Admin can manually edit any metadata field after fetch
- [ ] **META-06**: Admin can preview and confirm auto-fetched metadata before saving
- [ ] **META-07**: Metadata is stored in extensible format allowing future field additions

### User Interface - Grid

- [ ] **UI-01**: Users see movie collection in visual grid with poster, title, and year
- [ ] **UI-02**: Grid is responsive (1 column mobile, 2-4 tablet, 5-6 desktop)
- [ ] **UI-03**: Posters lazy-load as user scrolls
- [ ] **UI-04**: Missing posters show placeholder with movie title
- [ ] **UI-05**: Grid shows loading state while fetching movies
- [ ] **UI-06**: Empty state shown when no movies exist

### User Interface - Detail

- [ ] **UI-07**: Clicking poster opens detail page with full metadata
- [ ] **UI-08**: Detail page shows: large poster, titles, year, director, actors, runtime, genres, plot
- [ ] **UI-09**: Detail page includes prominent Download button
- [ ] **UI-10**: Download initiates browser file download (not streaming)
- [ ] **UI-11**: Detail page includes back navigation to grid

### Search & Filter

- [ ] **SRCH-01**: Users can search by movie title (fuzzy matching on original and Italian titles)
- [ ] **SRCH-02**: Users can search by actor or director name
- [ ] **SRCH-03**: Users can filter by release year or year range
- [ ] **SRCH-04**: Users can filter by genre(s) (multi-select)
- [ ] **SRCH-05**: Search results update in real-time as user types
- [ ] **SRCH-06**: Search completes in under 500ms for 500+ movies
- [ ] **SRCH-07**: Filters can be combined (search + year + genre)

### Security & Access Control

- [ ] **SEC-01**: Download endpoints require authentication
- [ ] **SEC-02**: Files cannot be accessed via direct URL (must go through API)
- [ ] **SEC-03**: Rate limiting prevents download abuse (configurable per IP)
- [ ] **SEC-04**: Admin interface requires authentication
- [ ] **SEC-05**: Registration endpoint is public and requires name, surname, and email
- [ ] **SEC-06**: Upon registration, user account is created with PENDING status
- [ ] **SEC-07**: Pending users cannot access movie content
- [ ] **SEC-08**: System generates unique confirmation token upon registration
- [ ] **SEC-09**: Admin can view all pending registrations in admin panel
- [ ] **SEC-10**: Admin can approve pending registration
- [ ] **SEC-11**: Admin can reject pending registration
- [ ] **SEC-12**: Upon approval, system automatically sends confirmation token to user's email
- [ ] **SEC-13**: Upon rejection, system automatically sends rejection email to user
- [ ] **SEC-14**: User enters confirmation token on public token validation page
- [ ] **SEC-15**: Token validation activates user account and creates authenticated session
- [ ] **SEC-16**: Each confirmation token is single-use and expires after activation
- [ ] **SEC-17**: Invalid/expired tokens show appropriate error message
- [ ] **SEC-18**: Authenticated sessions persist via secure httpOnly cookies
- [ ] **SEC-19**: Home page shows registration form to unauthenticated users
- [ ] **SEC-20**: Authenticated users see movie grid; pending users see "approval pending" message

### Admin Interface

- [ ] **ADMIN-01**: Admin can view all uploaded movies
- [ ] **ADMIN-02**: Admin can edit any movie's metadata
- [ ] **ADMIN-03**: Admin can delete movies (with confirmation)
- [ ] **ADMIN-04**: Admin can re-fetch metadata for existing movies
- [ ] **ADMIN-05**: Admin can upload new movies with identifier input
- [ ] **ADMIN-06**: Admin sees upload queue/progress
- [ ] **ADMIN-07**: Admin can view all pending user registrations
- [ ] **ADMIN-08**: Admin can approve pending registration (sends confirmation token via email)
- [ ] **ADMIN-09**: Admin can reject pending registration (sends rejection email)
- [ ] **ADMIN-10**: Admin can view approved/active users
- [ ] **ADMIN-11**: Admin can revoke user access

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Search Enhancement

- **SRCH-08**: Full-text search across plot summaries
- **SRCH-09**: Filter by minimum rating
- **SRCH-10**: Filter by production country
- **SRCH-11**: Saved searches/bookmarks

### User Experience

- **UI-12**: Backdrop image display in detail view
- **UI-13**: Movie collections/series grouping
- **UI-14**: Recently added section
- **UI-15**: Watch list/favorites (per user, if multi-user added)

### Admin Features

- **ADMIN-07**: Bulk upload (multiple files at once)
- **ADMIN-08**: Duplicate detection and warning
- **ADMIN-09**: Collection statistics dashboard
- **ADMIN-10**: Import from CSV/spreadsheet
- **ADMIN-11**: Backup/restore functionality

### Performance & Scale

- **PERF-01**: CDN integration for posters
- **PERF-02**: Database migration to PostgreSQL (if scaling)
- **PERF-03**: Redis caching for popular queries
- **PERF-04**: Video thumbnail generation

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Video streaming | Explicitly download-only per requirements |
| Transcoding | CPU intensive, unnecessary complexity for download-only use |
| Public sharing links | Would require complex permission model |
| Mobile app | Web-only requirement |
| Automatic download from external sources | Legal gray area, reliability issues |
| Multi-language UI | Italian focus specified, can add later |
| Social features (comments, ratings) | Not mentioned in requirements |
| Subtitle management | Not mentioned, adds complexity |
| Video playback in browser | Explicitly download-only |
| Password-based login for regular users | Token-based authentication sufficient |
| Multiple user roles | Single user role (authenticated user) |
| User profile management | Not needed for basic access |
| Password reset functionality | Tokens are single-use, admin can generate new ones |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| UPLOAD-01 | Phase 1 | Pending |
| UPLOAD-02 | Phase 1 | Pending |
| UPLOAD-03 | Phase 1 | Pending |
| UPLOAD-04 | Phase 1 | Pending |
| UPLOAD-05 | Phase 2 | Pending |
| META-01 | Phase 2 | Pending |
| META-02 | Phase 2 | Pending |
| META-03 | Phase 2 | Pending |
| META-04 | Phase 2 | Pending |
| META-05 | Phase 3 | Pending |
| META-06 | Phase 2 | Pending |
| META-07 | Phase 1 | Pending |
| UI-01 | Phase 3 | Pending |
| UI-02 | Phase 3 | Pending |
| UI-03 | Phase 3 | Pending |
| UI-04 | Phase 3 | Pending |
| UI-05 | Phase 3 | Pending |
| UI-06 | Phase 3 | Pending |
| UI-07 | Phase 4 | Pending |
| UI-08 | Phase 4 | Pending |
| UI-09 | Phase 4 | Pending |
| UI-10 | Phase 4 | Pending |
| UI-11 | Phase 4 | Pending |
| SRCH-01 | Phase 3 | Pending |
| SRCH-02 | Phase 3 | Pending |
| SRCH-03 | Phase 3 | Pending |
| SRCH-04 | Phase 3 | Pending |
| SRCH-05 | Phase 3 | Pending |
| SRCH-06 | Phase 3 | Pending |
| SRCH-07 | Phase 3 | Pending |
| SEC-01 | Phase 1 | Pending |
| SEC-02 | Phase 1 | Pending |
| SEC-03 | Phase 4 | Pending |
| SEC-04 | Phase 1 | Pending |
| SEC-05 | Phase 1 | Pending |
| ADMIN-01 | Phase 3 | Pending |
| ADMIN-02 | Phase 3 | Pending |
| ADMIN-03 | Phase 3 | Pending |
| ADMIN-04 | Phase 3 | Pending |
| ADMIN-05 | Phase 1 | Pending |
| ADMIN-06 | Phase 1 | Pending |

**Phase Mapping Summary:**

| Phase | Name | Requirement Count |
|-------|------|-------------------|
| Phase 1 | Foundation | 11 |
| Phase 2 | Upload & Metadata Core | 6 |
| Phase 3 | Grid, Search & Admin | 13 |
| Phase 4 | Detail, Download & Rate Limiting | 6 |
| Phase 5 | Admin Polish & Security Hardening | 0* |

**Coverage:**
- v1 requirements: 33 total
- Mapped to phases: 33
- Unmapped: 0 ✓

---
*Requirements defined: 2025-03-02*
*Last updated: 2025-03-02 after initial definition*
