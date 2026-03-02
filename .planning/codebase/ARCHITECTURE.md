# Architecture Document

**Analysis Date:** 2025-03-02

**Project Status:** Planning phase completed, Phase 1 implementation in progress

## Overview

This is a private movie library download service built as a full-stack web application. The system allows an administrator to upload large video files, automatically fetch metadata from external APIs (TMDb, OMDb), and provides a visual grid interface for invited users to browse, search, and download movies.

**Core Value Proposition:** Admin uploads a movie file with minimal identifying info; the system automatically enriches it with complete metadata; end users can quickly browse, search, and download.

---

## Tech Stack

### Backend

| Technology | Purpose | Configuration |
|------------|---------|---------------|
| **Node.js** 20.x LTS | Runtime | Express.js server |
| **TypeScript** 5.x | Language | Strict mode, ES2022 target |
| **Express.js** 4.18+ | Web Framework | RESTful API design |
| **SQLite** 3.x | Database | File-based (data/movies.db) |
| **Multer** 1.4+ | File Upload | Streaming, chunked upload support |
| **bcryptjs** 2.4+ | Password Hashing | bcrypt for admin credentials |
| **jsonwebtoken** 9.x | Authentication | JWT with 24h expiry, httpOnly cookies |
| **axios** 1.6+ | HTTP Client | TMDb/OMDb API integration |
| **sharp** 0.33+ | Image Processing | Poster optimization |
| **express-rate-limit** 7.x | Rate Limiting | Download abuse prevention |
| **helmet** 7.x | Security | HTTP security headers |
| **cors** 2.8+ | CORS | Configured for controlled access |
| **winston** 3.11+ | Logging | Structured logging |

### Frontend

| Technology | Purpose | Configuration |
|------------|---------|---------------|
| **React** 18.x | UI Framework | Functional components, hooks |
| **TypeScript** 5.x | Language | Shared types with backend |
| **Vite** | Build Tool | Fast HMR, optimized builds |
| **@tanstack/react-query** 5.x | Data Fetching | Server state management |
| **axios** 1.6+ | HTTP Client | Cookie-based auth |
| **fuse.js** 7.x | Fuzzy Search | Client-side title/actor search |

### External APIs

| Service | Purpose | Integration |
|---------|---------|-------------|
| **TMDb API** | Primary metadata source | Movie search, details, posters |
| **OMDb API** | Fallback metadata source | IMDb data via JSON API |
| **Image CDN** | Poster delivery | TMDb's image.tmdb.org |

---

## Database Schema

### SQLite Database

**Location:** `data/movies.db`

**Configuration:**
- WAL mode enabled for concurrency
- Foreign keys enabled
- Stored outside version control

### Core Tables

#### movies

The primary table storing movie metadata and file references.

```sql
CREATE TABLE movies (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  file_id TEXT UNIQUE NOT NULL,        -- Secure file ID (UUID v4)
  file_path TEXT NOT NULL,             -- Path to video file on disk
  file_size BIGINT,                    -- File size in bytes
  file_type TEXT,                      -- Video format (mp4, mkv, etc.)
  original_title TEXT,                 -- Original movie title
  italian_title TEXT,                  -- Italian localized title
  release_year INTEGER,                -- Release year
  runtime INTEGER,                   -- Runtime in minutes
  plot_summary TEXT,                   -- Plot description
  poster_path TEXT,                    -- Path to cached poster image
  metadata_json TEXT,                  -- Extensible metadata (JSON)
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**Indexes:**
```sql
CREATE INDEX idx_movies_file_id ON movies(file_id);        -- Secure lookups
CREATE INDEX idx_movies_original_title ON movies(original_title);  -- Title search
CREATE INDEX idx_movies_release_year ON movies(release_year);      -- Year filtering
```

#### movie_actors

Many-to-many relationship for actors, optimized for filtering.

```sql
CREATE TABLE movie_actors (
  movie_id INTEGER REFERENCES movies(id) ON DELETE CASCADE,
  actor_name TEXT NOT NULL,
  PRIMARY KEY (movie_id, actor_name)
);

CREATE INDEX idx_actors_name ON movie_actors(actor_name);  -- Actor search/filter
```

#### movie_genres

Many-to-many relationship for genres.

```sql
CREATE TABLE movie_genres (
  movie_id INTEGER REFERENCES movies(id) ON DELETE CASCADE,
  genre TEXT NOT NULL,
  PRIMARY KEY (movie_id, genre)
);

CREATE INDEX idx_genres ON movie_genres(genre);            -- Genre filtering
```

#### users

User accounts with token-based authentication.

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  surname TEXT NOT NULL,
  status TEXT DEFAULT 'PENDING',        -- PENDING, APPROVED, REJECTED
  confirmation_token TEXT UNIQUE,       -- Single-use token for activation
  token_expires_at DATETIME,            -- Token expiration
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  last_login_at DATETIME              -- Track last successful login
);

CREATE INDEX idx_users_token ON users(confirmation_token);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_email ON users(email);
```

#### sessions

Active authenticated sessions (httpOnly cookie tracking).

```sql
CREATE TABLE sessions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  session_token TEXT UNIQUE NOT NULL,   -- JWT token identifier
  expires_at DATETIME NOT NULL,
  ip_address TEXT,                    -- For security logging
  user_agent TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_sessions_token ON sessions(session_token);
CREATE INDEX idx_sessions_user ON sessions(user_id);
```

### JSON Metadata Structure

The `metadata_json` column stores extensible data:

```typescript
interface MovieMetadata {
  director?: string;
  actors?: string[];
  genres?: string[];
  rating?: number;              // IMDb/TMDb rating
  votes?: number;               // Number of votes
  originalLanguage?: string;
  productionCountries?: string[];
  tagline?: string;
  budget?: number;
  revenue?: number;
  imdbId?: string;
  tmdbId?: number;
  // Future fields added here
}
```

---

## API Structure

### Base URL

```
/api
```

### Authentication Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/auth/register` | Public | Request access (creates pending account) |
| POST | `/auth/confirm` | Public | Confirm account with token |
| POST | `/auth/login` | Public | User login (admin only for v1) |
| POST | `/auth/logout` | Required | Logout and clear session |
| GET | `/auth/me` | Required | Get current user info |
| GET | `/auth/status` | Optional | Check registration status by email |

### Media Endpoints (Public/Browse)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/media` | Required | List all media (movies, series) with pagination |
| GET | `/media/:id` | Required | Get media details (movie, series, or episode) |
| GET | `/media/:id/download` | Required | Download media file |
| GET | `/media/search` | Required | Search media (title, actor, director) |
| GET | `/media/filter` | Required | Filter by year, genre, type |
| GET | `/media/:id/related` | Required | Get related media items |
| GET | `/series/:id/episodes` | Required | Get episodes for a series |
| GET | `/series/:id/seasons/:season` | Required | Get episodes for specific season |

### Admin Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/admin/movies` | Admin | Upload new movie with file |
| GET | `/admin/upload-progress/:id` | Admin | SSE endpoint for upload progress |
| PUT | `/admin/movies/:id` | Admin | Update movie metadata |
| DELETE | `/admin/movies/:id` | Admin | Delete movie (with confirmation) |
| POST | `/admin/movies/:id/refetch` | Admin | Re-fetch metadata from TMDb |
| GET | `/admin/movies` | Admin | List all movies (admin view) |

### User Management Endpoints (Admin)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/admin/users/pending` | Admin | List pending registrations |
| POST | `/admin/users/:id/approve` | Admin | Approve user (sends token) |
| POST | `/admin/users/:id/reject` | Admin | Reject user (sends email) |
| GET | `/admin/users` | Admin | List all users |
| DELETE | `/admin/users/:id` | Admin | Revoke user access |

### Support Ticket Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/support/tickets` | Required | Submit a support ticket |
| GET | `/support/tickets/my` | Required | List user's own tickets |
| GET | `/support/tickets/:id` | Required | Get ticket details |

### Support Ticket Endpoints (Admin)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/admin/support/tickets` | Admin | List all tickets (with filters) |
| GET | `/admin/support/tickets/:id` | Admin | Get ticket details |
| PUT | `/admin/support/tickets/:id` | Admin | Update ticket (status, response) |
| DELETE | `/admin/support/tickets/:id` | Admin | Delete ticket |

### Movie Request Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/requests` | Required | Submit a movie request |
| GET | `/requests/my` | Required | List user's own requests |
| GET | `/requests/:id` | Required | Get request details |

### Movie Request Endpoints (Admin)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/admin/requests` | Admin | List all requests (with filters) |
| GET | `/admin/requests/:id` | Admin | Get request details |
| PUT | `/admin/requests/:id` | Admin | Update request status (approve/reject/complete) |
| DELETE | `/admin/requests/:id` | Admin | Delete request |

### Related Media Endpoints (Admin)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/admin/media/:id/related` | Admin | Add related media |
| DELETE | `/admin/media/:id/related/:relatedId` | Admin | Remove related media |
| GET | `/admin/media/:id/related` | Admin | Get related media (admin view) |

### Response Format

**Success:**
```json
{
  "success": true,
  "data": { ... }
}
```

**Error:**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid username or password"
  }
}
```

### Status Codes

| Code | Usage |
|------|-------|
| 200 | Success |
| 201 | Created (upload, registration) |
| 400 | Bad Request (validation error) |
| 401 | Unauthorized (missing/invalid token) |
| 403 | Forbidden (not admin) |
| 404 | Not Found |
| 413 | Payload Too Large (file > 50GB) |
| 415 | Unsupported Media Type (invalid file type) |
| 429 | Rate Limited |
| 500 | Server Error |

---

## Frontend Components/Pages

### Page Structure

```
client/src/pages/
├── Home.tsx              # Landing/registration for guests
├── Movies.tsx            # Media grid (authenticated users)
├── MediaDetail.tsx       # Media details + download (movies, series, episodes)
├── SeriesDetail.tsx      # Series overview with episodes
├── Login.tsx             # Admin login (v1 only)
├── TokenEntry.tsx        # Token confirmation page
├── RegistrationStatus.tsx # "Approval Pending" page
├── Support/
│   ├── Contact.tsx       # Contact/support form
│   └── MyTickets.tsx     # User's support tickets
├── Requests/
│   ├── NewRequest.tsx    # Request a movie
│   └── MyRequests.tsx    # User's requests
├── Admin/
│   ├── Dashboard.tsx     # Admin overview
│   ├── Upload.tsx        # Media upload with progress
│   ├── Movies.tsx        # Manage movies
│   ├── Series.tsx        # Manage series
│   ├── Users.tsx         # User management
│   ├── SupportTickets.tsx # Manage support tickets
│   ├── MovieRequests.tsx  # Manage movie requests
│   └── Relationships.tsx  # Manage related media
└── NotFound.tsx          # 404 page
```

### Component Hierarchy

#### Public/Guest Pages

**Home (`/`)**
- For unauthenticated users: Shows registration form
- For pending users: Shows "Approval Pending" message
- For authenticated users: Redirects to Movies grid

**TokenEntry (`/confirm`)**
- Token input field
- Validates token
- Activates account on success
- Creates authenticated session

**RegistrationStatus (`/status`)**
- Check registration by email
- Shows: PENDING, APPROVED, REJECTED states

#### User Pages (Authenticated)

**Movies Grid (`/movies`)**
- Responsive poster grid (1-6 columns based on viewport)
- Lazy loading on scroll
- Search bar (fuzzy title search)
- Filters: Year range, Genre, Actor, Director
- Sorting: Recently added, Title, Year, Rating
- Empty state with helpful message

**MovieDetail (`/movies/:id`)**
- Large poster display
- Full metadata (titles, year, director, actors, runtime, genres, plot)
- Download button (prominent)
- Back navigation to grid

#### Admin Pages (Admin Only)

**Admin Dashboard (`/admin`)**
- Upload queue status
- Total movies count
- Storage usage
- Recent activity

**Admin Upload (`/admin/upload`)**
- File picker with drag-and-drop
- Metadata input (title, year for TMDb fetch)
- Progress bar with SSE
- Success/error states
- Metadata preview (Phase 2)

**Admin Movies (`/admin/movies`)**
- Editable movie list
- Edit metadata modal
- Delete with confirmation
- Re-fetch metadata button

**Admin Users (`/admin/users`)**
- Pending registrations list
- Approve/Reject actions
- Active users list
- Revoke access action

### Shared Components

```
client/src/components/
├── MovieCard.tsx         # Poster card for grid
├── MovieGrid.tsx         # Responsive grid container
├── SearchBar.tsx         # Search with debounce
├── FilterPanel.tsx       # Collapsible filter sidebar
├── UploadProgress.tsx    # Progress bar for uploads
├── LoadingSpinner.tsx    # Loading states
├── ErrorBoundary.tsx     # Error handling
├── Navigation.tsx        # Main nav (authenticated)
└── ProtectedRoute.tsx    # Route guards
```

### API Services

```
client/src/services/
├── api.ts                # Base axios instance
├── auth.ts               # Auth API calls
├── movies.ts             # Movie API calls
├── upload.ts             # Upload with progress
└── search.ts             # Search utilities
```

---

## Authentication System

### Architecture

**Type:** JWT-based session authentication with httpOnly cookies

**Flow:**
1. User registers with name, surname, email (creates PENDING account)
2. Admin approves (generates single-use confirmation token)
3. System emails token to user
4. User enters token on confirmation page
5. System validates token, activates account, creates session
6. Session persisted via httpOnly cookie

### Session Management

**JWT Configuration:**
```typescript
{
  secret: process.env.JWT_SECRET,
  expiresIn: '24h',
  algorithm: 'HS256'
}
```

**Cookie Settings:**
```typescript
{
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 24 * 60 * 60 * 1000, // 24 hours
  path: '/'
}
```

### User States

```
┌─────────────┐    Admin approves    ┌──────────────┐
│   PENDING   │ ────────────────────> │   APPROVED   │
│  (waiting)  │                     │  (can login)  │
└─────────────┘                     └──────────────┘
        │                                    │
        │ Admin rejects                      │ Admin revokes
        ▼                                    ▼
┌─────────────┐                          ┌──────────────┐
│   REJECTED  │                          │   REVOKED    │
│  (deleted)  │                          │  (no access) │
└─────────────┘                          └──────────────┘
```

### Middleware

**`requireAuth`** - Verifies JWT in httpOnly cookie
- Returns 401 if missing/invalid
- Attaches `req.user` to request

**`requireAdmin`** - Extends requireAuth
- Returns 403 if not admin role

**`optionalAuth`** - Allows both authenticated and anonymous
- Attaches user if present
- Continues regardless

### Admin Authentication (Phase 1)

**v1 Implementation:**
- Single admin user configured via environment variables
- Admin credentials: `ADMIN_USERNAME` + `ADMIN_PASSWORD_HASH` (bcrypt)
- No registration endpoint for admin
- Admin bypasses approval flow

**Future:** Multi-admin support via database

---

## File Upload/Download System

### Upload Flow

```
[Client: Upload.tsx]
    │
    ▼ POST /api/admin/upload (multipart/form-data)
[Server: upload.ts route]
    │
    ▼ Multer middleware (streaming to temp)
[Server: uploadService.ts]
    │
    ├─> Track progress in memory (SSE updates)
    ├─> Validate file type (whitelist: mp4, mkv, avi, mov, webm)
    ├─> Check disk space
    │
    ▼ File complete
[Server: fileService.ts]
    │
    ├─> Generate UUID v4 file_id
    ├─> Create sharded directory (e.g., uploads/videos/7a/)
    ├─> Move file to permanent location
    │
    ▼ [Server: movieRepository.ts]
        │
        ├─> Create movie record in database
        ├─> Store file_id, file_path, file_size, file_type
        │
        ▼ Response: { success: true, movie: Movie, uploadId: string }
            │
            ▼ [Client] Shows success, redirect to movie
```

### Progress Tracking (SSE)

```
Client                     Server
  │    GET /api/admin/upload-progress/:id
  │ ───────────────────────────────────────> 
  │                            (SSE stream)
  │ <─ { "type": "progress", "received": 536870912,
  │      "total": 2147483648, "percentage": 25 } ─
  │ <─ { "type": "progress", "received": 1073741824,
  │      "percentage": 50 } ──────────────────────
  │ <─ { "type": "progress", "received": 1610612736,
  │      "percentage": 75 } ──────────────────────
  │ <─ { "type": "complete", "movieId": 42 } ─────
```

### File Storage Structure

```
server/
├── uploads/                    # Gitignored - outside web root
│   ├── videos/                 # Video files
│   │   ├── 7a/                 # First 2 chars of UUID (sharding)
│   │   │   └── 7a3f8d2e-...-e4b8.mp4
│   │   ├── e4/
│   │   │   └── e4b8...-...-f9c2.mkv
│   │   └── ...
│   ├── posters/                # Cached poster images
│   │   └── [tmdb-id].jpg
│   └── temp/                   # Temporary upload chunks
│       └── upload-*.tmp
```

**Security:**
- Files stored outside web root (`client/public/`)
- UUID-based file IDs prevent enumeration
- Sharded directories prevent filesystem limits
- Path traversal protection via validation

### Download Flow

```
[Client: MovieDetail.tsx]
    │
    ▼ GET /api/movies/:id/download
[Server: movies.ts route]
    │
    ▼ requireAuth middleware
[Server: movieRepository.ts]
    │
    ▼ Lookup file_id by movie id
[Server: fileService.ts]
    │
    ▼ Validate file_id, construct path
    │
    ▼ Stream file to response
[Client] Browser receives file, initiates download
```

**Download Security:**
- Authenticated endpoint only
- Rate limiting per IP (configurable)
- File streamed (not loaded into memory)
- Content-Disposition: attachment header
- MIME type detection from file extension

### File Validation

**Allowed Types:**
```typescript
const ALLOWED_VIDEO_TYPES = ['mp4', 'mkv', 'avi', 'mov', 'webm'];
const MAX_FILE_SIZE = 50 * 1024 * 1024 * 1024; // 50GB
```

**Validation Steps:**
1. Extension check (whitelist)
2. MIME type mapping
3. File size check before upload
4. Magic bytes verification (optional)

---

## Metadata Fetching System

### Data Sources

**Primary: TMDb API**
- Free tier: 40 requests/10 seconds
- Endpoints used:
  - `GET /search/movie` - Search by title/year
  - `GET /movie/{id}` - Movie details
  - `GET /movie/{id}/credits` - Cast and crew
  - Image CDN for posters

**Fallback: OMDb API**
- Free tier: 1000 requests/day
- IMDb data via JSON API
- Used when TMDb fails

### Fetch Flow

```
[Admin Upload] → Provide title/year or TMDb ID
        │
        ▼
[metadataService.ts]
        │
        ├─> If TMDb ID provided:
        │      GET /movie/{id}
        ├─> Else:
        │      GET /search/movie?query={title}&year={year}
        │      GET /movie/{first_result_id}
        │
        ├─> GET /movie/{id}/credits (for cast)
        │
        ├─> Download poster to local cache
        │
        ▼
[Transform to MovieMetadata]
        │
        ▼
[Admin Preview UI] → Confirm/Save to database
```

### Metadata Mapping

**TMDb → Application Schema:**

| TMDb Field | App Field | Notes |
|------------|-----------|-------|
| title | original_title | English/original title |
| release_date | release_year | Extract year only |
| runtime | runtime | Minutes |
| overview | plot_summary | |
| poster_path | poster_path | Download and cache locally |
| vote_average | metadata.rating | |
| vote_count | metadata.votes | |
| genres[].name | movie_genres table | Separate table |
| credits.cast[].name | movie_actors table | Top 10 only |
| credits.crew[].name (job: Director) | metadata.director | |
| original_language | metadata.originalLanguage | |
| production_countries[].name | metadata.productionCountries | |
| imdb_id | metadata.imdbId | |
| id | metadata.tmdbId | |

---

## Directory Structure

```
movie-library/
├── server/                      # Backend (Node.js + Express)
│   ├── src/
│   │   ├── config/             # Configuration modules
│   │   │   ├── database.ts     # SQLite connection
│   │   │   ├── auth.ts         # JWT config
│   │   │   ├── paths.ts        # Upload directory paths
│   │   │   └── schema.sql      # Database schema
│   │   ├── controllers/        # Request handlers
│   │   │   ├── authController.ts
│   │   │   ├── movieController.ts
│   │   │   └── adminController.ts
│   │   ├── services/           # Business logic
│   │   │   ├── fileService.ts  # Secure file storage
│   │   │   ├── uploadService.ts # Upload handling
│   │   │   ├── metadataService.ts # TMDb/OMDb fetching
│   │   │   └── userService.ts  # User management
│   │   ├── repositories/       # Database access
│   │   │   ├── movieRepository.ts
│   │   │   └── userRepository.ts
│   │   ├── routes/             # API route definitions
│   │   │   ├── auth.ts
│   │   │   ├── movies.ts
│   │   │   ├── admin.ts
│   │   │   └── upload.ts
│   │   ├── middleware/         # Express middleware
│   │   │   ├── auth.ts         # JWT validation
│   │   │   ├── errorHandler.ts
│   │   │   ├── rateLimit.ts
│   │   │   └── security.ts     # Helmet, CORS
│   │   ├── utils/              # Utility functions
│   │   │   ├── fileUtils.ts    # UUID generation, path sharding
│   │   │   ├── streamUtils.ts  # File streaming
│   │   │   ├── fileValidation.ts
│   │   │   ├── tmdbClient.ts   # TMDb API client
│   │   │   └── omdbClient.ts   # OMDb API client
│   │   ├── types/              # TypeScript types
│   │   │   ├── movie.ts
│   │   │   ├── user.ts
│   │   │   └── api.ts
│   │   └── index.ts            # Server entry point
│   ├── uploads/                # File storage (gitignored)
│   │   ├── videos/
│   │   ├── posters/
│   │   └── temp/
│   └── package.json
├── client/                      # Frontend (React + Vite)
│   ├── src/
│   │   ├── pages/              # Route-level components
│   │   │   ├── Home.tsx
│   │   │   ├── Movies.tsx
│   │   │   ├── MovieDetail.tsx
│   │   │   ├── Login.tsx
│   │   │   ├── TokenEntry.tsx
│   │   │   ├── RegistrationStatus.tsx
│   │   │   └── admin/
│   │   │       ├── Dashboard.tsx
│   │   │       ├── Upload.tsx
│   │   │       ├── Movies.tsx
│   │   │       └── Users.tsx
│   │   ├── components/         # Reusable components
│   │   │   ├── MovieCard.tsx
│   │   │   ├── MovieGrid.tsx
│   │   │   ├── SearchBar.tsx
│   │   │   ├── FilterPanel.tsx
│   │   │   ├── UploadProgress.tsx
│   │   │   ├── LoadingSpinner.tsx
│   │   │   ├── Navigation.tsx
│   │   │   └── ProtectedRoute.tsx
│   │   ├── services/           # API clients
│   │   │   ├── api.ts
│   │   │   ├── auth.ts
│   │   │   ├── movies.ts
│   │   │   └── upload.ts
│   │   ├── hooks/              # Custom React hooks
│   │   │   ├── useAuth.ts
│   │   │   ├── useMovies.ts
│   │   │   ├── useUpload.ts
│   │   │   └── useSearch.ts
│   │   ├── utils/              # Client utilities
│   │   │   └── search.ts
│   │   ├── App.tsx             # Main app component
│   │   └── main.tsx            # Entry point
│   ├── public/                 # Static assets
│   └── package.json
├── shared/                      # Shared types between client/server
│   └── types/
│       └── index.ts
├── data/                        # SQLite database (gitignored)
│   └── movies.db
├── .planning/                   # Project planning documents
├── package.json                 # Root workspace config
└── README.md
```

---

## Security Model

### Authentication Layers

1. **JWT in httpOnly cookie** - Primary session mechanism
2. **Rate limiting** - Prevents brute force (5 login attempts / 15 min)
3. **CORS** - Controlled cross-origin access
4. **Helmet headers** - XSS, clickjacking protection

### File Access Security

1. **UUID file IDs** - Non-sequential, unguessable
2. **Files outside web root** - No direct URL access
3. **Path traversal protection** - Input validation
4. **Authenticated endpoints** - API-only access
5. **Rate limiting** - Download abuse prevention

### API Security

1. **Parameterized queries** - SQL injection prevention
2. **Input validation** - Joi/zod schemas
3. **Error handling** - No stack traces in production
4. **CSP headers** - Content Security Policy

---

## Scaling Considerations

### Current (SQLite + Local Files)
- Suitable for: 0-1000 movies
- Single admin user
- Local deployment

### Future Scaling Path

| Bottleneck | Solution |
|------------|----------|
| Database | PostgreSQL for concurrent access |
| Search | SQLite FTS or Elasticsearch |
| File storage | S3-compatible object storage |
| Posters | CDN integration |
| Sessions | Redis for session storage |

---

## Phase Implementation

### Phase 1: Foundation (In Progress)
- Database schema and types
- Secure file storage
- JWT authentication
- Basic upload with progress

### Phase 2: Metadata Core
- TMDb/OMDb integration
- Metadata fetching
- Poster caching
- Resume capability

### Phase 3: Grid & Search
- Responsive poster grid
- Lazy loading
- Search/filter UI
- Admin movie management

### Phase 4: Detail & Download
- Movie detail page
- Download endpoint
- Rate limiting
- Security hardening

### Phase 5: Admin Polish
- Security audit
- Error handling
- Edge cases
- Production validation

---

## Database Schema

### SQLite Database

**Location:** `data/movies.db`

**Configuration:**
- WAL mode enabled for concurrency
- Foreign keys enabled
- Stored outside version control

### Core Tables

#### media_items (REPLACES movies table - supports both movies and series)

The primary table storing media metadata and file references.

```sql
CREATE TABLE media_items (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  media_type TEXT NOT NULL CHECK(media_type IN ('movie', 'series', 'anime')), -- Type of content
  file_id TEXT UNIQUE,                     -- Secure file ID (UUID v4) - NULL for series without episodes
  file_path TEXT,                        -- Path to video file on disk - NULL for series
  file_size BIGINT,                      -- File size in bytes
  file_type TEXT,                        -- Video format (mp4, mkv, etc.)
  original_title TEXT,                   -- Original movie/title
  italian_title TEXT,                    -- Italian localized title
  release_year INTEGER,                  -- Release year
  runtime INTEGER,                     -- Runtime in minutes
  plot_summary TEXT,                     -- Plot description
  poster_path TEXT,                      -- Path to cached poster image
  metadata_json TEXT,                    -- Extensible metadata (JSON)
  parent_series_id INTEGER REFERENCES media_items(id) ON DELETE SET NULL, -- For episodes: parent series
  season_number INTEGER,                 -- For episodes: season number
  episode_number INTEGER,                -- For episodes: episode number
  episode_title TEXT,                  -- For episodes: episode specific title
  total_seasons INTEGER,                 -- For series: total seasons count
  total_episodes INTEGER,                -- For series: total episodes count
  status TEXT DEFAULT 'ongoing' CHECK(status IN ('ongoing', 'completed', 'cancelled')), -- For series
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**Indexes:**
```sql
CREATE INDEX idx_media_file_id ON media_items(file_id);        -- Secure lookups
CREATE INDEX idx_media_original_title ON media_items(original_title);  -- Title search
CREATE INDEX idx_media_release_year ON media_items(release_year);      -- Year filtering
CREATE INDEX idx_media_type ON media_items(media_type);        -- Type filtering
CREATE INDEX idx_media_series ON media_items(parent_series_id); -- Series episodes lookup
CREATE INDEX idx_media_season_ep ON media_items(parent_series_id, season_number, episode_number);
```

#### movie_actors

Many-to-many relationship for actors, optimized for filtering.

```sql
CREATE TABLE movie_actors (
  media_id INTEGER REFERENCES media_items(id) ON DELETE CASCADE,
  actor_name TEXT NOT NULL,
  PRIMARY KEY (media_id, actor_name)
);

CREATE INDEX idx_actors_name ON movie_actors(actor_name);  -- Actor search/filter
CREATE INDEX idx_actors_media ON movie_actors(media_id);    -- Actor lookup by media
```

#### movie_genres

Many-to-many relationship for genres.

```sql
CREATE TABLE movie_genres (
  media_id INTEGER REFERENCES media_items(id) ON DELETE CASCADE,
  genre TEXT NOT NULL,
  PRIMARY KEY (media_id, genre)
);

CREATE INDEX idx_genres ON movie_genres(genre);            -- Genre filtering
CREATE INDEX idx_genres_media ON movie_genres(media_id);     -- Genre lookup by media
```

#### media_relationships (NEW - for related movies/series)

Bidirectional relationships between media items.

```sql
CREATE TABLE media_relationships (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  source_media_id INTEGER NOT NULL REFERENCES media_items(id) ON DELETE CASCADE,
  target_media_id INTEGER NOT NULL REFERENCES media_items(id) ON DELETE CASCADE,
  relationship_type TEXT NOT NULL CHECK(relationship_type IN (
    'sequel', 'prequel', 'remake', 'spin_off', 'same_universe', 
    'adaptation', 'related', 'part_of_series'
  )),
  description TEXT,                      -- Optional description
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(source_media_id, target_media_id, relationship_type)
);

CREATE INDEX idx_relationships_source ON media_relationships(source_media_id);
CREATE INDEX idx_relationships_target ON media_relationships(target_media_id);
CREATE INDEX idx_relationships_type ON media_relationships(relationship_type);
```

#### users

User accounts with token-based authentication.

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  surname TEXT NOT NULL,
  status TEXT DEFAULT 'PENDING',        -- PENDING, APPROVED, REJECTED
  confirmation_token TEXT UNIQUE,       -- Single-use token for activation
  token_expires_at DATETIME,            -- Token expiration
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  last_login_at DATETIME              -- Track last successful login
);

CREATE INDEX idx_users_token ON users(confirmation_token);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_email ON users(email);
```

#### sessions

Active authenticated sessions (httpOnly cookie tracking).

```sql
CREATE TABLE sessions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  session_token TEXT UNIQUE NOT NULL,   -- JWT token identifier
  expires_at DATETIME NOT NULL,
  ip_address TEXT,                    -- For security logging
  user_agent TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_sessions_token ON sessions(session_token);
CREATE INDEX idx_sessions_user ON sessions(user_id);
```

#### support_tickets (NEW)

User-submitted support tickets and bug reports.

```sql
CREATE TABLE support_tickets (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER REFERENCES users(id) ON DELETE SET NULL,
  ticket_type TEXT NOT NULL CHECK(ticket_type IN ('bug_report', 'general_inquiry', 'technical_issue', 'other')),
  subject TEXT NOT NULL,
  description TEXT NOT NULL,
  status TEXT DEFAULT 'open' CHECK(status IN ('open', 'in_progress', 'resolved', 'closed')),
  priority TEXT DEFAULT 'normal' CHECK(priority IN ('low', 'normal', 'high', 'urgent')),
  admin_response TEXT,                   -- Admin's response
  responded_at DATETIME,                 -- When admin responded
  resolved_at DATETIME,                  -- When ticket was resolved
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_tickets_user ON support_tickets(user_id);
CREATE INDEX idx_tickets_status ON support_tickets(status);
CREATE INDEX idx_tickets_type ON support_tickets(ticket_type);
CREATE INDEX idx_tickets_created ON support_tickets(created_at);
```

#### movie_requests (NEW)

User requests for movies to be added to library.

```sql
CREATE TABLE movie_requests (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER REFERENCES users(id) ON DELETE SET NULL,
  title TEXT NOT NULL,                   -- Requested movie/series title
  year INTEGER,                          -- Release year (if known)
  media_type TEXT DEFAULT 'movie' CHECK(media_type IN ('movie', 'series', 'anime', 'unknown')),
  description TEXT,                      -- Additional details from user
  tmdb_id INTEGER,                       -- TMDb ID if user knows it
  imdb_id TEXT,                          -- IMDb ID if user knows it
  status TEXT DEFAULT 'pending' CHECK(status IN ('pending', 'approved', 'rejected', 'completed')),
  admin_notes TEXT,                      -- Admin's notes/decision reason
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_requests_user ON movie_requests(user_id);
CREATE INDEX idx_requests_status ON movie_requests(status);
CREATE INDEX idx_requests_created ON movie_requests(created_at);
```

### JSON Metadata Structure

The `metadata_json` column stores extensible data:

```typescript
interface MediaMetadata {
  director?: string;
  actors?: string[];
  genres?: string[];
  rating?: number;              // IMDb/TMDb rating
  votes?: number;               // Number of votes
  originalLanguage?: string;
  productionCountries?: string[];
  tagline?: string;
  budget?: number;
  revenue?: number;
  imdbId?: string;
  tmdbId?: number;
  
  // Series-specific fields
  episodeRuntime?: number;      // Average episode runtime for series
  network?: string;             // TV network/streaming service
  seriesType?: string;          // 'tv_series', 'anime', 'miniseries', etc.
  
  // Future fields added here
}
```

---

*Architecture analysis: 2025-03-02*
