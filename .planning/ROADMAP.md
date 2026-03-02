# Roadmap: Private Movie Library Download Service

**Created:** 2025-03-02
**Phases:** 5
**Requirements:** 33 v1 mapped

## Overview

This roadmap delivers the Private Movie Library Download Service in five coherent phases following the core value proposition: **Admin uploads a movie file with minimal identifying info; the system automatically enriches it with complete metadata; end users can quickly browse, search, and download.**

Phases are structured to deliver vertical slices of functionality:
1. **Foundation** — Infrastructure that everything else depends on
2. **Upload & Metadata** — Core admin workflow (the heart of the system)
3. **Grid & Search** — User browsing capabilities
4. **Detail & Download** — Complete user flow
5. **Admin Polish & Security Hardening** — Production readiness

Each phase delivers a demonstrable, testable capability. Dependencies flow forward: Phase 1 enables Phase 2, Phase 2 enables Phases 3 and 4, etc.

---

## Phases

### Phase 1: Foundation
**Goal:** Infrastructure exists for secure file storage, database operations, and access control with admin upload capability and token-based user authentication.

**Dependencies:** None (ground floor)

**Requirements:** UPLOAD-01, UPLOAD-02, UPLOAD-03, UPLOAD-04, META-07, SEC-01, SEC-02, SEC-04, SEC-05, SEC-06, SEC-07, SEC-08, SEC-09, SEC-10, SEC-11, ADMIN-05, ADMIN-06

**Rationale:** This phase establishes the non-negotiable foundation. Files must be stored securely outside web root (UPLOAD-04). Authentication must exist before any protected operations (SEC-01, SEC-04). Basic upload with progress indicators enables the admin workflow that follows. Metadata storage format must be designed early to support extensibility (META-07). Token-based authentication ensures controlled access - users must request access from admin who generates unique single-use tokens.

**Plans:**
- `01-01-PLAN.md` — Database schema and TypeScript types (updated for token system)
- `01-02-PLAN.md` — Secure file storage system
- `01-03-PLAN.md` — Authentication and upload infrastructure (includes token system)

**Success Criteria:**
1. Admin can log in to a protected interface (requires authentication)
2. Admin can upload a video file (up to 50GB) with real-time progress indicator
3. Uploaded files are stored outside web root with secure IDs, inaccessible by direct URL
4. Database schema supports extensible movie metadata and token-based authentication
5. No public registration endpoint exists (access is controlled)
6. Upload queue shows current uploads and their status
7. Unauthenticated users see access request page (not movie grid)
8. Admin can generate unique single-use access tokens for invited users
9. Token entry form validates tokens and activates user session
10. Authenticated sessions persist via secure httpOnly cookies

---

### Phase 2: Upload & Metadata Core
**Goal:** Uploaded movies are automatically enriched with metadata from external APIs, with admin preview and confirmation.

**Dependencies:** Phase 1 (file storage, upload, auth)

**Requirements:** UPLOAD-05, META-01, META-02, META-03, META-04, META-06

**Rationale:** This is the core value proposition. After uploading a file (Phase 1), the system must fetch and cache rich metadata. Resume capability (UPLOAD-05) handles network interruptions for large files. Dual API strategy (TMDb + OMDb fallback) ensures reliability. Poster caching is essential for the grid display in Phase 3.

**Success Criteria:**
1. After upload, system automatically fetches metadata from TMDb using title/year
2. If TMDb fails, system falls back to OMDb for IMDb data
3. Fetched metadata includes: title, Italian title, director, top 10 actors, year, runtime, genres, plot, poster URL
4. Poster image is downloaded and cached locally
5. Admin can preview fetched metadata and confirm before saving to database
6. Interrupted uploads can be resumed without starting over

---

### Phase 3: Grid, Search & Admin Management
**Goal:** Users can browse the movie collection in a responsive grid with real-time search and filtering; admin can manage movies.

**Dependencies:** Phase 2 (metadata must exist to display and search)

**Requirements:** META-05, UI-01, UI-02, UI-03, UI-04, UI-05, UI-06, SRCH-01, SRCH-02, SRCH-03, SRCH-04, SRCH-05, SRCH-06, SRCH-07, ADMIN-01, ADMIN-02, ADMIN-03, ADMIN-04

**Rationale:** This is the primary user-facing phase. Grid display depends on posters and metadata from Phase 2. Search requires indexed metadata. Manual metadata editing (META-05) and admin management (ADMIN-01..04) are grouped here as they support the content lifecycle. Performance target (SRCH-06) ensures scalability.

**Success Criteria:**
1. Users see a responsive grid of movie posters with titles and years (1 col mobile, 2-4 tablet, 5-6 desktop)
2. Posters lazy-load as user scrolls; missing posters show title placeholder
3. Grid shows loading state during fetch and empty state when no movies exist
4. Users can search by title (fuzzy on original and Italian) with real-time results under 500ms
5. Users can filter by actor/director, year range, and genres (combinable)
6. Admin can view all movies, edit any metadata, delete with confirmation, and re-fetch metadata

---

### Phase 4: Detail, Download & Rate Limiting
**Goal:** Users can view complete movie details and download files; download abuse is prevented via rate limiting.

**Dependencies:** Phase 3 (grid browsing leads to detail view)

**Requirements:** UI-07, UI-08, UI-09, UI-10, UI-11, SEC-03

**Rationale:** This completes the user journey: browse → view details → download. Rate limiting (SEC-03) is grouped here as it protects the download endpoints that are now active. Download must initiate browser file download, not streaming (core constraint).

**Success Criteria:**
1. Clicking a poster opens detail page with full metadata and large poster
2. Detail page displays: large poster, titles (original + Italian), year, director, actors, runtime, genres, plot
3. Prominent Download button initiates browser file download (not streaming)
4. Download endpoints require authentication and are rate-limited per IP (configurable)
5. Detail page includes back navigation to grid
6. Files cannot be accessed via direct URL (must go through authenticated API)

---

### Phase 5: Admin Polish & Security Hardening
**Goal:** Admin workflow is complete and polished; security controls are production-ready.

**Dependencies:** All prior phases (comprehensive hardening)

**Requirements:** (None specific — this phase focuses on integration, testing, and hardening of existing features)

**Rationale:** With core features complete, this phase addresses production readiness. While specific requirements are satisfied in earlier phases, this phase ensures they work cohesively under security review, error handling, and edge cases. It's the "make it production-ready" phase.

**Success Criteria:**
1. All authentication flows are secure against common attacks (CSRF, XSS, brute force)
2. File download streaming is memory-efficient for 10GB+ files
3. Admin interface has clear error handling for API failures (TMDb/OMDb down)
4. System gracefully handles edge cases: empty library, API rate limits, disk full
5. End-to-end workflow validated: upload → fetch metadata → browse → search → download

---

## Traceability

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

**Coverage:** 33/33 requirements mapped ✓

---

## Phase Summary Table

| Phase | Name | Requirements | Key Deliverable |
|-------|------|--------------|-----------------|
| 1 | Foundation | 11 | Secure upload infrastructure with auth |
| 2 | Upload & Metadata Core | 6 | Automatic metadata enrichment with preview |
| 3 | Grid, Search & Admin | 13 | Browse, search, and manage movies |
| 4 | Detail, Download & Rate Limiting | 6 | Complete download flow with abuse protection |
| 5 | Admin Polish & Security Hardening | 3* | Production-ready system |

*Phase 5 focuses on hardening and integration of existing requirements rather than new features.

---

## Dependency Graph

```
Phase 1 (Foundation)
├── Database + File Storage
├── Basic Security (auth, secure storage)
└── Basic Admin Upload
    │
    ▼
Phase 2 (Upload & Metadata Core)
├── Resume Capability
├── Metadata Fetch (TMDb + OMDb)
├── Poster Caching
└── Admin Preview/Confirm
    │
    ├──┬──────────────────┐
    ▼  ▼                  ▼
Phase 3 (Grid & Search)   Phase 4 (Detail & Download)
├── Responsive Grid       ├── Detail View
├── Lazy Loading          ├── Download Endpoint
├── Real-time Search      ├── Rate Limiting
├── Filter System         └── Back Navigation
└── Admin Management
    │
    ▼
Phase 5 (Admin Polish & Security Hardening)
├── Security Audit
├── Error Handling
├── Edge Case Coverage
└── Production Validation
```

---

## Critical Path Notes

**Must complete sequentially:**
1. Phase 1 → Phase 2: Cannot fetch metadata without file storage
2. Phase 2 → Phase 3: Cannot display/search without metadata
3. Phase 3 → Phase 4: Detail view depends on grid navigation

**Can parallelize after Phase 2:**
- Phase 3 (Grid/Search) and Phase 4 (Detail/Download) share Phase 2 as dependency but not each other
- However, user flow suggests sequential delivery: browse → detail → download

**Security considerations:**
- SEC-01, SEC-02, SEC-04, SEC-05 in Phase 1 provide baseline
- SEC-03 (rate limiting) in Phase 4 when downloads go live
- Phase 5 comprehensive security review

---

## Out of Scope Reminders

Per REQUIREMENTS.md, these remain out of scope for v1:
- Video streaming (download-only)
- Transcoding
- User registration (controlled access only)
- Public sharing links
- Mobile app
- Automatic external downloads
- Multi-language UI (Italian focus)
- Social features
- Subtitle management
- In-browser playback

---

*Roadmap created: 2025-03-02*
*Next step: `/gsd-plan-phase 1` to begin Phase 1 planning*
