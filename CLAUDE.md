# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spotter is a fitness tracking application built with Next.js 15.5.1, React 19, and TypeScript. It's currently a development prototype that uses mock authentication and localStorage for data persistence.

**Important**: The working codebase is consolidated in the root `spotter-free/` directory.

## Development Commands

Navigate to `spotter-free/` before running commands:

```bash
# Start development server (Turbopack disabled due to Windows compatibility issues)
npm run dev

# Build for production (includes TypeScript checking and ESLint)
npm run build

# Start production server
npm run start

# Run ESLint for code quality
npm run lint

# Type checking (configured to fail builds on errors)
npx tsc --noEmit
```

## Architecture Overview

### Directory Structure
- `spotter-free/src/app/` - Next.js App Router pages and API routes
  - `add/` - Add workout functionality with edit page
  - `api/` - API routes (health, instagram-fetch, ocr)
  - `calendar/` - Calendar view page
  - `library/` - Workout library page
  - `settings/` - User settings page
  - `workout/[id]/` - Dynamic workout detail pages with edit functionality
- `src/components/` - Reusable React components
  - `auth/` - Authentication components (login.tsx)
  - `layout/` - Layout components (header.tsx, mobile-nav.tsx)
  - `ui/` - Base UI components (shadcn/ui style - button, card, input, etc.)
- `src/store/` - Zustand state management 
  - `index.ts` - Auth store with persistent login state
- `src/lib/` - Utility functions and shared logic
  - `workout/reify.ts` - Workout data transformation utilities
  - `igParser.ts` & `igParser_toV1.ts` - Instagram parsing logic
  - `editable-workout-table.tsx` - Complex table component
  - `utils.ts` - Common utilities (cn function for className merging)

### Key Technologies
- **Framework**: Next.js 15 with App Router (Turbopack disabled on Windows)
- **UI**: React 19 with TypeScript, Tailwind CSS
- **State**: Zustand for client state management
- **Build**: Standalone output mode configured for containerization
- **Authentication**: Mock auth system (needs production replacement)

### State Management
Uses Zustand with persistence for authentication state:
- `useAuthStore` in `src/store/index.ts` handles login/logout
- Currently implements mock authentication (any email/password works)
- State persists to localStorage under 'spotter-auth' key

### Styling System
- **Framework**: Tailwind CSS with custom dark theme
- **Colors**: Custom color palette defined for fitness app
  - Background: `#0C0F12` (Dark navy)
  - Primary: `#00D0BD` (Cyan accent)
  - Secondary: `#7A7EFF` (Purple accent)
- **Components**: shadcn/ui style component library in `src/components/ui/`
- **Fonts**: Geist Sans and Geist Mono from Google Fonts

## Development Guidelines

### Code Quality
- TypeScript strict mode enabled - all type errors fail builds
- ESLint configured with Next.js and TypeScript rules
- Custom ESLint rules configured in `eslint.config.mjs`:
  - `@typescript-eslint/no-explicit-any`: "warn"
  - `@typescript-eslint/no-unused-vars`: "warn" 
  - `react/no-unescaped-entities`: "warn"
  - `prefer-const`: "warn"
- Path aliases configured: `@/*` maps to `./src/*`
- Next.js config enforces TypeScript and ESLint during builds

### API Routes
Located in `src/app/api/` with endpoints:
- `/api/health` - Health check endpoint
- `/api/ocr` - OCR processing capabilities (Tesseract.js integration)
- `/api/instagram-fetch` - Instagram content extraction

### Component Patterns
- Uses React 19 features and patterns
- Functional components with hooks
- Custom UI components follow consistent patterns
- Authentication components handle routing and state

### External Integrations
- **Tesseract.js**: Client-side OCR capabilities via `/api/ocr`
- **AWS SDK**: Textract integration (`@aws-sdk/client-textract`) for document processing
- **Instagram**: Custom parser (`igParser.ts`) for workout content extraction
- **Zustand**: State management with localStorage persistence

## Production Considerations

### Current Limitations (Development Prototype)
- Mock authentication system - replace before production
- localStorage-based data persistence - needs backend database
- No session management or token expiration
- Missing security headers and CSP configuration

### Deployment Configuration
- Dockerfile and docker-compose.yml included for containerization
- AWS deployment scripts available (`deploy-to-aws.ps1`)
- Next.js standalone output mode configured
- Images configured for localhost domain (update for production)

### Testing
No test framework currently configured. When adding tests:
- Consider Jest + React Testing Library for unit tests
- Add E2E tests with Playwright or Cypress
- Set up CI/CD pipeline with test automation

## Key Features & Data Flow

### Authentication System
- Mock authentication in `src/store/index.ts` using Zustand
- Any email/password combination works in development
- User state persists to localStorage under 'spotter-auth' key
- Default demo credentials: `demo@spotter.com` / `password`

### Workout Data Processing
- Instagram content parsing via custom `igParser.ts` and `igParser_toV1.ts`
- AST (Abstract Syntax Tree) to workout data transformation in `reify.ts`
- Editable workout table component for data manipulation
- OCR processing for workout images via Tesseract.js

### UI/UX Architecture
- Dark theme with custom fitness app color palette
- Mobile-responsive design with dedicated mobile navigation
- Tailwind CSS with shadcn/ui component patterns
- Geist font family for modern typography