# Pitfalls Research

**Domain:** Media/Movie Library Web Application
**Researched:** 2025-03-02
**Confidence:** HIGH

## Critical Pitfalls

### Pitfall 1: Large File Upload Failures

**What goes wrong:**
Movie uploads fail silently or time out, especially for 4K/Blu-ray quality files (10-50GB). Users retry multiple times causing storage waste.

**Why it happens:**
Default Express body parsers have size limits. Browser timeouts occur before upload completes. Network interruptions on large files.

**How to avoid:**
- Configure multer with appropriate limits (100GB+)
- Use streaming upload handlers, not buffering
- Implement chunked upload with resume capability
- Add upload progress indication (WebSocket or SSE)
- Set appropriate proxy timeouts (nginx/Apache if used)

**Warning signs:**
- Uploads consistently fail at certain file sizes
- Server memory usage spikes during uploads
- Timeouts logged before completion

**Phase to address:**
Phase 1 (File Upload System) — Core infrastructure

---

### Pitfall 2: Incorrect Metadata Matching

**What goes wrong:**
System fetches wrong movie metadata (wrong year, wrong film with similar name), admin doesn't notice, library becomes unreliable.

**Why it happens:**
- Title collisions (remakes, same name different years)
- TMDb search returns best match, not necessarily correct
- No validation step before confirming match

**How to avoid:**
- Require confirmation for auto-matched metadata
- Show multiple search results, let admin select
- Display match confidence score
- Allow manual override at any time
- Store raw search results for audit

**Warning signs:**
- Multiple movies with same title in library
- User complaints about wrong metadata
- Admin spending time correcting matches

**Phase to address:**
Phase 2 (Metadata System) — Critical UX feature

---

### Pitfall 3: Search Performance Degradation

**What goes wrong:**
Search becomes slow (5+ seconds) as library grows to 500+ movies, especially for actor/director searches.

**Why it happens:**
- LIKE '%text%' queries on large text fields
- No indexing on searchable columns
- Searching across JSON fields without proper indexing

**How to avoid:**
- Implement SQLite FTS (Full-Text Search) from start
- Index title, actors, director columns
- Consider client-side search (Fuse.js) for smaller libraries
- Denormalize actor data if needed for performance

**Warning signs:**
- Search response time increases with library size
- Database CPU usage spikes during searches
- Queries timing out

**Phase to address:**
Phase 3 (Search System) — Design in from start

---

### Pitfall 4: Storage Security Breaches

**What goes wrong:**
Movie files accessible via direct URL guessing, allowing unauthorized downloads.

**Why it happens:**
- Files stored in publicly accessible directory
- Predictable file naming (sequential IDs)
- No authentication on download endpoints

**How to avoid:**
- Store files outside web root
- Generate cryptographically secure file IDs
- Always serve through authenticated endpoints
- Add rate limiting to download endpoints
- Validate user session on every download

**Warning signs:**
- Unusual download patterns
- Files accessed without corresponding UI interactions
- Security scanning tools flagging accessible files

**Phase to address:**
Phase 1 (File Upload System) — Security foundation

---

### Pitfall 5: Poster Image Handling Issues

**What goes wrong:**
Broken images in grid, slow page loads, excessive bandwidth usage from unoptimized posters.

**Why it happens:**
- Hot-linking to external image URLs (breaks if TMDb changes)
- No image optimization (serving 2000px images in 200px grid)
- No fallback for missing posters

**How to avoid:**
- Download and cache posters locally
- Generate multiple sizes (thumbnail, medium, full)
- Use Sharp for image optimization
- Implement placeholder/fallback images
- Lazy load images in grid

**Warning signs:**
- Grid loads slowly with many images
- 404s in console from image requests
- High bandwidth usage

**Phase to address:**
Phase 2 (Metadata System) — Image handling sub-component

---

### Pitfall 6: Database Schema Rigidity

**What goes wrong:**
Can't add new metadata fields (e.g., rating, production company) without migrations, data loss, or complex workarounds.

**Why it happens:**
- Over-normalized schema requiring joins for simple queries
- No JSON column for flexible metadata
- Hardcoded field names throughout codebase

**How to avoid:**
- Use SQLite JSON columns for extensible metadata
- Separate fixed fields (id, file_path) from flexible metadata
- Design schema for evolution from start
- Version your database migrations

**Warning signs:**
- "We need to add X but it requires changing 10 files"
- Migration conflicts during development
- Data loss during schema changes

**Phase to address:**
Phase 1 (Database Design) — Foundation

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Store file paths as strings | Simple, works immediately | Hard to move files, no validation | Only for MVP, plan migration |
| Skip input validation | Faster development | Security issues, crashes | Never in production |
| Hardcoded config values | Quick to test | Can't deploy to different environments | Only local dev |
| No API rate limiting on TMDb | Simpler code | Hit rate limits, API keys banned | Never |

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| TMDb API | Not handling rate limits | Implement exponential backoff, respect 429 responses |
| TMDb API | No fallback when TMDb is down | Cache results, provide manual entry fallback |
| TMDb Images | Hot-linking without fallback | Download and serve locally with fallback |
| File Storage | Assuming consistent paths | Use relative paths, validate on startup |

## Performance Traps

Patterns that work at small scale but fail as usage grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Loading all movies to client | Browser memory issues | Pagination, server-side filtering | 100+ movies |
| No image lazy loading | Slow page loads | Implement intersection observer | 50+ posters |
| Synchronous file reads | Event loop blocking | Always use streams | Concurrent downloads |
| Unindexed searches | Slow queries | FTS or proper indexes | 500+ movies |
| Full metadata in list view | Large payloads | Select only needed fields | 100+ movies |

## Security Mistakes

Domain-specific security issues beyond general web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Exposing file storage path | Path traversal attacks | Store outside web root, serve via controller |
| No upload file type validation | Malware uploads | Whitelist video extensions, validate MIME types |
| Predictable download URLs | Unauthorized access | Use UUIDs, authenticate every request |
| No download rate limiting | Bandwidth abuse | Implement per-IP limits |
| Storing API keys in frontend | Credential theft | Only use keys server-side |
| SQL injection in search | Database compromise | Use parameterized queries, validate inputs |

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| No upload progress | Users think upload failed | Progress bar with percentage |
| Grid without filtering | Hard to find movies | Search + filters always visible |
| Missing poster placeholders | Broken-looking UI | Placeholder with movie title |
| Click to download immediately | Accidental downloads | Confirmation or detail page first |
| No metadata preview before save | Wrong matches committed | Preview modal with edit option |
| Infinite scroll on slow connection | Frustrating, can't skip | Pagination with page numbers |

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **File Upload:** Often missing progress indication, resume capability — verify both
- [ ] **Metadata Fetch:** Often missing error handling, manual override — verify fallback
- [ ] **Search:** Often missing fuzzy matching, partial word search — verify "Star" finds "Star Wars"
- [ ] **Poster Display:** Often missing lazy loading, fallbacks — verify works with slow connection
- [ ] **Download:** Often missing proper headers (filename, content-type) — verify browser handles download correctly
- [ ] **Admin Interface:** Often missing validation, error messages — verify all error paths

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Wrong metadata | LOW | Edit movie, re-fetch, or manual correction |
| Storage full | MEDIUM | Move files to larger disk, update paths |
| TMDb API banned | HIGH | Request new API key, implement better rate limiting |
| Database corruption | HIGH | Restore from backup, implement backups first |
| Slow search | MEDIUM | Add indexes, implement FTS, may require migration |

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Large File Upload Failures | Phase 1 | Upload 10GB+ file successfully |
| Storage Security Breaches | Phase 1 | Verify files not accessible via direct URL |
| Database Schema Rigidity | Phase 1 | Add test field without code changes |
| Incorrect Metadata Matching | Phase 2 | Admin can correct auto-matched metadata |
| Poster Image Handling | Phase 2 | All posters load, optimized sizes exist |
| Search Performance | Phase 3 | Search completes in <500ms with 500 movies |

## Sources

- Express.js large file upload best practices
- TMDb API rate limiting documentation
- SQLite performance optimization guides
- Media server security considerations

---
*Pitfalls research for: Private Movie Library*
*Researched: 2025-03-02*
