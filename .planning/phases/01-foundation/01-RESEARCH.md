# Phase 1 Research: Foundation

**Phase:** 01-foundation  
**Researched:** 2025-03-02  
**Focus:** Database schema, file storage, authentication, upload infrastructure

---

## Research Findings

### Database Schema Design (SQLite)

**Key Requirements:**
- Extensible metadata storage (META-07)
- Relational data for movies
- Support for future fields without migrations

**Recommended Schema:**

```sql
-- Core movies table with extensible metadata
CREATE TABLE movies (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  file_id TEXT UNIQUE NOT NULL, -- Secure file ID (UUID)
  file_path TEXT NOT NULL,      -- Path to video file
  file_size BIGINT,             -- File size in bytes
  file_type TEXT,               -- Video format (mp4, mkv, etc.)
  
  -- Core metadata (normalized)
  original_title TEXT,
  italian_title TEXT,
  release_year INTEGER,
  runtime INTEGER,              -- Minutes
  plot_summary TEXT,
  poster_path TEXT,             -- Local poster file path
  
  -- Extensible metadata (JSON for flexibility)
  metadata_json TEXT,           -- Stores: director, actors, genres, etc.
  
  -- Timestamps
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Separate table for search optimization
CREATE TABLE movie_actors (
  movie_id INTEGER REFERENCES movies(id),
  actor_name TEXT,
  PRIMARY KEY (movie_id, actor_name)
);

CREATE TABLE movie_genres (
  movie_id INTEGER REFERENCES movies(id),
  genre TEXT,
  PRIMARY KEY (movie_id, genre)
);
```

**Rationale:**
- Normalized core fields for query performance
- JSON blob for extensibility (director, rating, etc.)
- Separate actor/genre tables for efficient filtering
- Secure file_id (UUID) prevents enumeration attacks

### File Storage Architecture

**Security Requirements:**
- Files outside web root (SEC-02)
- Cryptographically secure IDs (UPLOAD-04)
- No direct URL access

**Recommended Structure:**

```
project/
├── server/
│   └── uploads/              # Outside web root
│       └── videos/
│           ├── 7a/          # First 2 chars of UUID (sharding)
│           │   └── 7a3f...
│           └── e4/
│               └── e4b8...
├── client/
│   └── public/              # Web root - NO uploads here
```

**File ID Strategy:**
- Generate UUID v4: `7a3f...e4b8`
- Use first 2 chars as subdirectory for filesystem performance
- Store full UUID in database as `file_id`

### Authentication Strategy

**Requirements:**
- Admin-only access (SEC-04, SEC-05)
- Download endpoints protected (SEC-01)
- No public registration

**Recommended Approach: JWT-based**

1. **Admin Setup:** Hardcoded or environment-based credentials
   - Single admin user (no registration needed)
   - Credentials set via environment variables
   - Simple login form → JWT token → httpOnly cookie

2. **Token Strategy:**
   - JWT with 24h expiry
   - httpOnly, secure, sameSite cookies
   - No refresh token (admin can re-login)

3. **Middleware:**
   - `requireAuth` middleware for protected routes
   - Applied to: upload, download, admin endpoints
   - Public endpoints: none (all require auth)

**Environment Variables:**
```bash
ADMIN_USERNAME=admin
ADMIN_PASSWORD_HASH=$2b$10$...  # bcrypt hash
JWT_SECRET=your-secret-key
```

### Large File Upload Strategy

**Requirements:**
- Support 50GB+ files (UPLOAD-01)
- Progress indication (UPLOAD-02)
- Resume capability (UPLOAD-05)

**Recommended Stack:**

1. **Multer with Streaming:**
   - Use `multer` with custom storage engine
   - Stream to filesystem (not memory)
   - Chunked upload with `Content-Range` support

2. **Progress Tracking:**
   - Client-side: XMLHttpRequest with progress event
   - Server-side: Track bytes received vs total
   - WebSocket or SSE for real-time updates

3. **Resume Strategy (for v1.1):**
   - Check if partial file exists
   - Resume from last received byte
   - Store partial files in temp directory

**Express Configuration:**
```javascript
// Extended timeouts for large uploads
app.use(express.json({ limit: '1mb' }));  // API requests only
app.use(express.urlencoded({ extended: true, limit: '1mb' }));

// Upload route has custom timeout
app.post('/api/admin/upload', 
  (req, res, next) => {
    req.setTimeout(0); // No timeout for upload
    next();
  },
  uploadHandler
);
```

### Upload Progress Implementation

**Options Evaluated:**

1. **WebSocket** — Best for real-time, adds complexity
2. **Server-Sent Events (SSE)** — Simpler, one-way, good fit
3. **Polling** — Simple but inefficient

**Recommendation:** SSE for Phase 1

**Flow:**
1. Client uploads via fetch/XHR
2. Server tracks progress in memory or temp file
3. Client connects to `/api/admin/upload-progress/:uploadId`
4. Server streams progress events
5. Client updates UI

### TypeScript Types

**Core Types:**

```typescript
// Movie model
interface Movie {
  id: number;
  fileId: string;
  filePath: string;
  fileSize: number;
  fileType: string;
  originalTitle: string | null;
  italianTitle: string | null;
  releaseYear: number | null;
  runtime: number | null;
  plotSummary: string | null;
  posterPath: string | null;
  metadata: MovieMetadata;
  createdAt: Date;
  updatedAt: Date;
}

// Extensible metadata
interface MovieMetadata {
  director?: string;
  actors?: string[];
  genres?: string[];
  rating?: number;
  originalLanguage?: string;
  productionCountries?: string[];
  // Future fields can be added here
}

// Upload session
interface UploadSession {
  uploadId: string;
  filename: string;
  totalBytes: number;
  receivedBytes: number;
  status: 'uploading' | 'completed' | 'failed';
  fileId: string | null;
}
```

### Project Structure

**Recommended Layout:**

```
movie-library/
├── server/                      # Backend
│   ├── src/
│   │   ├── config/             # DB, auth, paths
│   │   ├── models/             # TypeScript types
│   │   ├── services/           # Business logic
│   │   ├── routes/             # API routes
│   │   ├── middleware/         # Auth, error handling
│   │   └── index.ts            # Entry point
│   ├── uploads/                # File storage (gitignored)
│   └── package.json
├── client/                      # Frontend
│   ├── src/
│   │   ├── components/         # React components
│   │   ├── pages/              # Page components
│   │   ├── services/           # API client
│   │   └── App.tsx
│   └── package.json
├── shared/                      # Shared types
│   └── types/
├── .planning/
└── package.json (root with workspaces)
```

---

## Key Decisions for Phase 1

### 1. Database: SQLite
**Decision:** Use SQLite for Phase 1  
**Rationale:** Zero config, file-based, perfect for single-admin use  
**Migration path:** Can upgrade to PostgreSQL later if needed

### 2. File Storage: Local Filesystem
**Decision:** Store files locally in `server/uploads/`  
**Rationale:** Simple, fast, no cloud dependencies  
**Security:** Files served only through authenticated API routes

### 3. Authentication: JWT + Environment Credentials
**Decision:** Single hardcoded admin, JWT tokens  
**Rationale:** No registration needed, simple to implement  
**Security:** bcrypt password hashing, httpOnly cookies

### 4. Upload Strategy: Multer + SSE Progress
**Decision:** Multer for upload handling, SSE for progress  
**Rationale:** Battle-tested, streaming support, good progress UX  
**Note:** Resume capability can be added in Phase 2

### 5. API Design: REST
**Decision:** RESTful API endpoints  
**Rationale:** Simple, well-understood, fits CRUD operations  
**Base:** `/api/v1/...`

---

## Technical Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| File upload failures | Streaming writes, proper error handling, temp files |
| Storage exhaustion | Disk space checks before upload, cleanup on failure |
| Security breaches | UUID file IDs, auth middleware, path traversal prevention |
| Database locks | SQLite is fine for single user, WAL mode for concurrency |
| CORS issues | Configure Express CORS middleware properly |

---

## Integration Points

### External Services (Phase 2)
- TMDb API — will need API key configuration
- OMDb API — fallback metadata source

### Internal Dependencies
- Phase 1 → Phase 2: File storage, auth, upload working
- Phase 1 → Phase 3: Database schema must support search

---

## Performance Targets

- Upload: Support 50GB+ files with progress updates
- Auth: <100ms login response
- Database: <50ms for simple queries

---

## What NOT to Build in Phase 1

- ❌ Resume capability (deferred to Phase 2)
- ❌ Metadata fetching (Phase 2)
- ❌ Search functionality (Phase 3)
- ❌ User management (single admin only)
- ❌ Cloud storage integration
- ❌ CDN integration

---

*Research for Phase 1: Foundation*
*Created: 2025-03-02*
