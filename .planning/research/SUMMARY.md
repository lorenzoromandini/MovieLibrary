# Research Summary

**Project:** Private Movie Library Download Service
**Researched:** 2025-03-02
**Confidence:** HIGH

## Key Findings

### Stack Recommendation

**Core Stack:** Node.js 20.x + Express + React 18 + TypeScript + SQLite + Multer

**Rationale:**
- Mature, well-documented ecosystem
- SQLite ideal for single-admin personal use (zero config, file-based)
- React perfect for poster grid UI with component reuse
- TypeScript essential for complex metadata types
- Multer standard for handling large file uploads

**Key Libraries:**
- **axios** — Reliable HTTP client for TMDb/OMDb APIs
- **sharp** — Image optimization for posters
- **fuse.js** — Client-side fuzzy search
- **react-query** — Server state management with caching

**What to Avoid:**
- MongoDB — Overkill for structured metadata
- Streaming protocols (HLS/DASH) — Explicitly out of scope
- Next.js/Nuxt — Overkill for personal app
- WebSocket — No real-time features needed

### Table Stakes Features

**Must Have for v1:**
1. Large file upload with progress indication
2. Automatic metadata fetch from TMDb
3. Manual metadata editing capability
4. Poster grid with responsive layout
5. Search by title (fuzzy matching)
6. Filter by genre/year
7. Movie detail page with download
8. Private access control

**Anti-Features (Don't Build):**
- User registration system — adds complexity for private app
- Video streaming — explicitly out of scope
- Transcoding — CPU intensive, unnecessary for downloads
- Public sharing links — complicates security

### Architecture Insights

**Three-Layer Architecture:**
1. **Frontend (React SPA)** — Grid, search, detail views
2. **Backend (Express API)** — Routes, services, external APIs
3. **Data Layer** — SQLite + file system + TMDb/OMDb

**Key Patterns:**
- Service layer for business logic separation
- Repository pattern for database access
- Store files outside web root, serve via authenticated endpoints
- Download and cache posters locally (don't hot-link)

**Build Order:**
1. Database + File storage (foundation)
2. Upload + Metadata fetch (core workflow)
3. Grid + Search (user interface)
4. Detail + Download (complete flow)
5. Polish + Admin improvements

### Critical Pitfalls to Avoid

1. **Large File Upload Failures** — Use streaming, add progress, configure timeouts
2. **Incorrect Metadata Matching** — Always require confirmation, show confidence scores
3. **Search Performance** — Implement FTS from start, don't use LIKE '%text%'
4. **Storage Security** — Files outside web root, authenticated downloads only
5. **Poster Handling** — Cache locally, optimize sizes, lazy load
6. **Schema Rigidity** — Use JSON columns for extensible metadata

### Complexity Assessment

| Component | Complexity | Risk Level |
|-----------|------------|------------|
| File Upload | MEDIUM | HIGH (large files) |
| Metadata Fetch | MEDIUM | MEDIUM (API dependencies) |
| Database Design | LOW | LOW |
| Search | MEDIUM | MEDIUM (performance) |
| Poster Grid | LOW | LOW |
| Download | LOW | MEDIUM (security) |

**Overall:** Moderate complexity project. Main risks are file upload handling and search performance at scale.

### TMDb API Considerations

**Free Tier Limits:**
- 40 requests per 10 seconds
- Sufficient for personal use
- Implement caching and rate limiting

**Key Endpoints:**
- `/search/movie` — Title/year search
- `/movie/{id}` — Full details
- `/configuration` — Image base URLs

**Fallback Strategy:**
- Primary: TMDb
- Secondary: OMDb (IMDb data)
- Tertiary: Manual entry

### Performance Targets

**Search:** <500ms for 500 movies (use FTS)
**Upload:** Support 10GB+ files (streaming, resume)
**Grid:** Load 50 posters in <2 seconds (lazy loading)
**Download:** Stream efficiently, no memory buffering

### Security Priorities

1. Files served only through authenticated endpoints
2. Cryptographically secure file IDs (not sequential)
3. Rate limiting on downloads
4. Input validation on all user inputs
5. API keys stored server-side only

## Research-to-Requirements Mapping

| Research Finding | Requirement Category |
|------------------|---------------------|
| Large file upload patterns | UPLOAD-01, UPLOAD-02 |
| TMDb API integration | META-01, META-02 |
| Manual metadata correction | META-03 |
| SQLite FTS for search | SRCH-01, SRCH-02 |
| Poster grid patterns | UI-01, UI-02 |
| Security patterns | SEC-01, SEC-02 |
| Responsive design | UI-03 |

## Next Steps

1. Define requirements based on research findings
2. Create phases around build order
3. Plan Phase 1: Database + File Upload + Basic API
4. Plan Phase 2: Metadata System + Image Handling
5. Plan Phase 3: Search + Grid Interface
6. Plan Phase 4: Detail Page + Download
7. Plan Phase 5: Admin Polish + Security

---

## Research Files

- **STACK.md** — Technology recommendations with versions
- **FEATURES.md** — Table stakes, differentiators, anti-features
- **ARCHITECTURE.md** — Component structure and patterns
- **PITFALLS.md** — Common mistakes and prevention strategies

---
*Research synthesis for: Private Movie Library*
*Synthesized: 2025-03-02*
