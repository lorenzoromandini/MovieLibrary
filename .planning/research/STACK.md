# Stack Research

**Domain:** Media/Movie Library Web Application
**Researched:** 2025-03-02
**Confidence:** HIGH

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Node.js | 20.x LTS | Runtime | Mature ecosystem, excellent async handling for file operations, widespread support |
| Express.js | 4.18+ | Backend Framework | Minimal, flexible, excellent middleware ecosystem, battle-tested for file serving |
| React | 18.x | Frontend Framework | Component-based architecture perfect for poster grids, excellent state management options |
| TypeScript | 5.x | Language | Type safety for complex data models (movies, metadata), better IDE support |
| SQLite | 3.x | Database | Zero-config, file-based storage perfect for single-admin personal use, handles metadata well |
| Multer | 1.4+ | File Upload | Industry standard for Express, handles large files with streaming, multipart form support |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| axios | 1.6+ | HTTP Client | Fetching metadata from TMDb/OMDb APIs, reliable retry handling |
| sharp | 0.33+ | Image Processing | Poster image resizing/optimization, format conversion |
| bcryptjs | 2.4+ | Password Hashing | Admin authentication, secure credential storage |
| jsonwebtoken | 9.x | JWT Auth | Session management for admin access |
| winston | 3.11+ | Logging | Structured logging for uploads, errors, API calls |
| dotenv | 16.x | Config | Environment variables for API keys, paths |
| cors | 2.8+ | CORS | Controlled access configuration |
| helmet | 7.x | Security | Security headers for web app |
| express-rate-limit | 7.x | Rate Limiting | Prevent abuse of download endpoints |
| fuse.js | 7.x | Fuzzy Search | Client-side search across movie titles, actors |
| react-query | 5.x | Data Fetching | Server state management, caching, background updates |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| nodemon | Auto-restart dev server | Essential for rapid iteration |
| ts-node | TypeScript execution | Run TS directly without compilation step |
| concurrently | Run multiple processes | Start backend + frontend with one command |
| eslint | Code linting | TypeScript + React configuration |
| prettier | Code formatting | Consistent code style |
| vite | Build tool | Fast HMR, optimized production builds |

## Installation

```bash
# Core
npm install express@^4.18 react@^18 react-dom@^18 typescript@^5 sqlite3@^5 multer@^1.4

# Supporting
npm install axios@^1.6 sharp@^0.33 bcryptjs@^2.4 jsonwebtoken@^9 winston@^3.11 dotenv@^16 cors@^2.8 helmet@^7 express-rate-limit@^7 fuse.js@^7 @tanstack/react-query@^5

# Dev dependencies
npm install -D nodemon ts-node concurrently eslint prettier vite @types/express @types/react @types/node
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| SQLite | PostgreSQL | If multiple users need concurrent admin access, if scaling beyond personal use |
| Express | Fastify | If needing 20%+ better performance, if building larger API surface |
| React | Vue | If developer prefers Vue's template syntax, similar capabilities |
| REST API | GraphQL | If frontend needs highly flexible queries, overkill for this use case |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| MongoDB | Overkill for structured metadata, adds deployment complexity | SQLite for simple single-user, PostgreSQL if scaling |
| MongoDB GridFS | Unnecessary complexity for file storage | Direct filesystem storage with metadata in DB |
| WebSocket/Socket.io | No real-time features needed | Standard HTTP with polling for progress |
| Video streaming protocols (HLS, DASH) | Explicitly download-only per requirements | Direct file serving with proper headers |
| Client-side frameworks (Next.js, Nuxt) | Overkill for personal app, adds complexity | React SPA with Vite |

## Stack Patterns by Variant

**If deploying to cloud server:**
- Use PostgreSQL instead of SQLite for remote storage
- Use S3-compatible object storage for video files instead of local filesystem
- Add nginx reverse proxy for serving files

**If keeping strictly local/personal:**
- SQLite is perfect choice
- Local filesystem storage is fine
- No need for cloud-specific libraries

**If adding transcoding later:**
- Add ffmpeg-wasm for client-side transcoding
- Or ffmpeg CLI integration for server-side
- Note: This adds significant complexity

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| sharp@0.33 | Node.js 18+ | Requires libvips native dependencies |
| sqlite3@5 | Node.js 16+ | May need build tools on some systems |
| React 18 | React-DOM 18 | Must match major versions |

## Sources

- Express.js official docs — verified middleware compatibility
- TMDb API documentation — confirmed v3 API stability
- Sharp documentation — version 0.33 current stable
- React 18 release notes — concurrent features, StrictMode changes

---
*Stack research for: Private Movie Library*
*Researched: 2025-03-02*
