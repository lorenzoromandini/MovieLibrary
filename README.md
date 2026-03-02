# Private Movie Library Download Service

A personal web-based movie collection manager with automatic metadata enrichment and direct file downloads. Built for private use — no streaming, no public registration, just a clean interface for browsing and downloading your movie collection.

## What This Is

This service lets you:
1. **Upload** movie files (up to 50GB) via an admin interface
2. **Automatically fetch metadata** from TMDb/OMDb (titles, posters, cast, plot, etc.)
3. **Browse** your collection in a visual poster grid
4. **Search and filter** by title, actor, director, year, genre
5. **Download** files directly (no streaming)

**Key Features:**
- Automatic metadata fetching saves hours of manual entry
- Secure file storage with UUID-based access
- Responsive poster grid (mobile to desktop)
- Real-time upload progress with resume capability
- JWT-based authentication (single admin)
- SQLite database (zero configuration)

## Quick Start

```bash
# Clone the repository
git clone git@github.com:lorenzoromandini/MovieLibrary.git
cd MovieLibrary

# Install dependencies
npm install

# Set up environment
cp server/.env.example server/.env
# Edit server/.env with your admin credentials and API keys

# Initialize database
npm run db:init

# Start development server
npm run dev

# Open http://localhost:3000
```

## Architecture

```
MovieLibrary/
├── server/              # Backend (Node.js + Express + SQLite)
│   ├── src/
│   │   ├── config/     # Database, auth, paths
│   │   ├── models/     # TypeScript types
│   │   ├── services/   # Business logic
│   │   ├── routes/     # API endpoints
│   │   └── middleware/ # Auth, validation
│   └── uploads/        # Secure file storage (gitignored)
├── client/             # Frontend (React + Vite)
│   └── src/
│       ├── components/ # Reusable UI components
│       ├── pages/      # Page components
│       └── services/   # API client
└── .planning/          # Project documentation
```

### Technology Stack

- **Backend:** Node.js 20.x, Express 4.x, TypeScript 5.x, SQLite 3.x
- **Frontend:** React 18.x, TypeScript 5.x, Vite
- **File Upload:** Multer with streaming support
- **Authentication:** JWT with httpOnly cookies
- **Metadata:** TMDb API (primary), OMDb API (fallback)

## Configuration

### Environment Variables

Create `server/.env`:

```env
# Admin credentials
ADMIN_USERNAME=admin
ADMIN_PASSWORD_HASH=$2b$10$...  # bcrypt hash

# JWT
JWT_SECRET=your-secret-key-here

# TMDb API (get from https://www.themoviedb.org/settings/api)
TMDB_API_KEY=your-tmdb-api-key

# OMDb API (optional fallback, get from http://www.omdbapi.com/apikey.aspx)
OMDB_API_KEY=your-omdb-api-key

# Server
PORT=3000
NODE_ENV=development
```

Generate bcrypt hash:
```bash
node -e "console.log(require('bcryptjs').hashSync('your-password', 10))"
```

## Usage

### Admin Workflow

1. **Login** at `/admin/login`
2. **Upload movies** at `/admin/upload`
   - Select video file (mp4, mkv, avi, mov, webm)
   - Optionally provide title/year for better metadata matching
   - Watch upload progress
   - Review fetched metadata
   - Confirm to add to library
3. **Manage library** at `/admin/movies`
   - Edit metadata manually if needed
   - Re-fetch metadata
   - Delete movies

### User Workflow

1. **Browse** movies at `/` — poster grid with lazy loading
2. **Search** by title, actor, or director
3. **Filter** by year range and genres
4. **Click** poster for details
5. **Download** via download button

## Project Phases

This project is built in 5 phases:

| Phase | Focus | Status |
|-------|-------|--------|
| 1 | Foundation — Database, auth, file storage | In Progress |
| 2 | Upload & Metadata — TMDb/OMDb integration | Pending |
| 3 | Grid & Search — Browse, filter, admin management | Pending |
| 4 | Detail & Download — Complete download flow | Pending |
| 5 | Polish & Hardening — Security review, edge cases | Pending |

See `.planning/ROADMAP.md` for detailed requirements and success criteria.

## Development

### Commands

```bash
# Install all dependencies
npm install

# Run database initialization
npm run db:init

# Start development servers (frontend + backend)
npm run dev

# Run type checking
npm run typecheck

# Run linting
npm run lint

# Build for production
npm run build

# Start production server
npm start
```

### Project Structure

- **`.planning/`** — Project documentation (requirements, roadmap, research)
- **`server/`** — Express backend with SQLite database
- **`client/`** — React frontend with TypeScript
- **`shared/`** — Shared types between client and server

### Security Considerations

- Files stored outside web root with UUID-based access
- All download endpoints require authentication
- Rate limiting on download endpoints
- JWT stored in httpOnly, secure cookies
- No public registration (controlled access only)
- Path traversal protection on all file operations

## API Documentation

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/login` | Login with credentials |
| POST | `/api/auth/logout` | Logout and clear session |
| GET | `/api/auth/me` | Get current user |

### Movies

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/movies` | List all movies |
| GET | `/api/movies/:id` | Get movie details |
| POST | `/api/admin/upload` | Upload new movie |
| GET | `/api/admin/upload-progress/:id` | Upload progress (SSE) |
| PUT | `/api/admin/movies/:id` | Update movie metadata |
| DELETE | `/api/admin/movies/:id` | Delete movie |

### Download

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/download/:fileId` | Download file (authenticated) |

## Contributing

This is a personal project. Feel free to fork and adapt for your own use.

## License

MIT — See LICENSE file

## Acknowledgments

- Metadata from [TMDb](https://www.themoviedb.org/) and [OMDb](http://www.omdbapi.com/)
- Built with the GSD (Get Shit Done) workflow methodology

---

*Built for personal movie collections. Not for redistribution of copyrighted content.*
