# Architecture Research

**Domain:** Media/Movie Library Web Application
**Researched:** 2025-03-02
**Confidence:** HIGH

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend (React SPA)                  │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Poster Grid │  │   Search    │  │   Detail    │          │
│  │   View      │  │   Filter    │  │    View     │          │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │
│         │                │                │                   │
├───────┴────────────────┴────────────────┴───────────────────┤
│                        Backend API (Express)                 │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    API Routes                        │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐           │    │
│  │  │  Movies  │ │  Admin   │ │ Download │           │    │
│  │  │  Routes  │ │  Routes  │ │  Routes  │           │    │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘           │    │
│  │       │            │            │                   │    │
│  └───────┴────────────┴────────────┴───────────────────┘    │
│         │            │            │                         │
│  ┌──────┴────────────┴────────────┴────────────────────┐    │
│  │                   Services                            │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │    │
│  │  │ Metadata │ │  File    │ │  Search  │            │    │
│  │  │ Service  │ │ Service  │ │ Service  │            │    │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘            │    │
│  └───────┴────────────┴────────────┴────────────────────┘    │
├─────────────────────────────────────────────────────────────┤
│                        Data Layer                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │  SQLite  │  │  File    │  │  External│                  │
│  │ Database │  │ Storage  │  │   APIs   │                  │
│  │(Metadata)│  │(Videos +  │  │(TMDb,    │                  │
│  │          │  │ Posters) │  │ OMDb)    │                  │
│  └──────────┘  └──────────┘  └──────────┘                  │
└─────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| Frontend | User interface, state management | React 18 + React Query |
| API Routes | HTTP endpoints, request validation | Express router |
| Metadata Service | Fetch from TMDb/OMDb, parse responses | axios + data transformation |
| File Service | Upload handling, storage, retrieval | Multer + filesystem |
| Search Service | Query building, result ranking | SQLite FTS or Fuse.js |
| SQLite Database | Metadata persistence | sqlite3 with schema |
| File Storage | Video and poster file storage | Local filesystem (or S3) |

## Recommended Project Structure

```
project/
├── server/                    # Backend (Node.js + Express)
│   ├── src/
│   │   ├── config/           # Configuration (DB, paths, API keys)
│   │   ├── models/           # Data models (Movie, metadata)
│   │   ├── services/         # Business logic (metadata, files, search)
│   │   ├── routes/           # API routes (movies, admin, download)
│   │   ├── middleware/       # Express middleware (auth, validation)
│   │   ├── utils/            # Utilities (API clients, helpers)
│   │   └── index.ts          # Entry point
│   └── uploads/              # File storage (gitignored)
│       ├── videos/           # Movie files
│       └── posters/          # Poster images
├── client/                    # Frontend (React + Vite)
│   ├── src/
│   │   ├── components/       # Reusable components
│   │   ├── pages/            # Page components (Grid, Detail, Admin)
│   │   ├── hooks/            # Custom React hooks
│   │   ├── services/         # API client
│   │   ├── utils/            # Utilities
│   │   ├── App.tsx           # Main app
│   │   └── main.tsx          # Entry point
│   └── index.html
├── .planning/                # Project documentation
└── package.json              # Root with workspaces
```

### Structure Rationale

- **server/ vs client/:** Clear separation of concerns, enables independent deployment
- **services/ pattern:** Business logic isolated from HTTP layer, testable
- **uploads/ gitignored:** Large files shouldn't be version controlled
- **models/:** Central type definitions shared across backend

## Architectural Patterns

### Pattern 1: Service Layer

**What:** Business logic separated from route handlers in dedicated services
**When to use:** When logic needs to be reused across routes or tested independently
**Trade-offs:** More files, but better testability and separation of concerns

**Example:**
```typescript
// services/metadataService.ts
export class MetadataService {
  async fetchFromTMDb(title: string, year?: number): Promise<MovieMetadata> {
    // TMDb API logic
  }
  
  async fetchFromOMDb(imdbId: string): Promise<MovieMetadata> {
    // OMDb API logic
  }
}

// routes/movies.ts
router.post('/movies', async (req, res) => {
  const metadata = await metadataService.fetchFromTMDb(req.body.title, req.body.year);
  // ...
});
```

### Pattern 2: Repository Pattern

**What:** Database access abstracted behind repository classes
**When to use:** When you might switch databases or need complex queries
**Trade-offs:** Additional layer, but easier to change storage

**Example:**
```typescript
// repositories/movieRepository.ts
export class MovieRepository {
  async findById(id: number): Promise<Movie | null> {
    // SQLite query
  }
  
  async search(query: string, filters: Filters): Promise<Movie[]> {
    // Complex query with FTS
  }
}
```

### Pattern 3: API Client Pattern

**What:** External API calls wrapped in typed clients
**When to use:** All external API integrations
**Trade-offs:** More code upfront, but handles errors consistently

**Example:**
```typescript
// utils/tmdbClient.ts
export class TMDBClient {
  private apiKey: string;
  
  async searchMovie(query: string, year?: number): Promise<TMDBMovie[]> {
    // axios call with error handling
  }
  
  async getMovieDetails(id: number): Promise<TMDBMovieDetails> {
    // ...
  }
}
```

## Data Flow

### Request Flow

```
[User Action: Upload Movie]
    ↓
[React: Upload Form] → [Axios: POST /api/admin/movies]
    ↓
[Express: multer middleware] → [Route Handler]
    ↓
[File Service: Save video] → [Metadata Service: Fetch from TMDb]
    ↓
[Database: Insert movie record] → [Response: Success]
    ↓
[React: Show success, refresh grid]
```

### State Management

```
[React Query Cache]
    ↓ (subscriptions)
[Components] ←→ [Mutations] → [API Calls] → [Cache Updates]
```

### Key Data Flows

1. **Upload Flow:** Client → Multer → File Storage + Database
2. **Metadata Fetch:** Route → Service → TMDb API → Transform → Database
3. **Browse Flow:** Client → API → Database → Transform → Client Cache
4. **Download Flow:** Client → API → File Storage → Stream → Browser

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 0-100 movies | Current architecture fine, SQLite + local files |
| 100-1000 movies | Add SQLite FTS for search, consider CDN for posters |
| 1000+ movies | Consider PostgreSQL, Redis for caching, dedicated file server |

### Scaling Priorities

1. **First bottleneck:** Search performance with FTS — implement SQLite FTS or client-side search
2. **Second bottleneck:** File storage size — compress videos or implement chunked uploads

## Anti-Patterns

### Anti-Pattern 1: Storing Video Files in Database

**What people do:** Store video files as BLOBs in SQLite
**Why it's wrong:** Terrible performance, database bloat, backup issues
**Do this instead:** Store files on filesystem, store paths in database

### Anti-Pattern 2: Synchronous File Operations

**What people do:** Use fs.readFileSync for uploads
**Why it's wrong:** Blocks event loop, crashes under load
**Do this instead:** Use async/await with streams for large files

### Anti-Pattern 3: Storing External API Keys in Code

**What people do:** Hardcode TMDb API key in service file
**Why it's wrong:** Security risk, can't rotate keys
**Do this instead:** Environment variables with validation

### Anti-Pattern 4: No Separation Between Admin and Public Routes

**What people do:** Check role in every route handler
**Why it's wrong:** Error-prone, inconsistent enforcement
**Do this instead:** Separate router instances with different middleware

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| TMDb API | HTTP REST with API key | Rate limits: 40 requests/10 seconds (free tier) |
| OMDb API | HTTP REST with API key | 1000 daily limit on free tier, good fallback |
| Image CDN | Direct HTTP fetch | TMDb provides image CDN URLs |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Client ↔ API | HTTP/JSON | RESTful endpoints |
| API ↔ Database | SQLite driver | Local connection |
| API ↔ File Storage | Filesystem | Local paths |

## Sources

- Express.js best practices
- TMDb API documentation
- SQLite full-text search documentation
- React Query architecture patterns

---
*Architecture research for: Private Movie Library*
*Researched: 2025-03-02*
