# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)
- **PDF Generation**: pdfkit (externalized from esbuild bundle)

## Application: Resumease

An AI-powered career platform with resume building, ATS analysis, bullet enhancement, job matching, and portfolio generation.

### Features
- **Landing page** — hero, AI tools showcase, template preview, footer
- **Resume Builder** — split-screen with live preview, 3 templates (Modern, Classic, Minimal), PDF download
- **ATS Analyzer** — AI scores resume 0-100, highlights missing keywords, suggests improvements
- **AI Bullet Enhancer** — converts rough notes into professional resume bullets
- **Job Description Matcher** — match %, matched/missing keywords, tailored suggestions
- **Portfolio Generator** — 6-step form, 5 color schemes, generates shareable HTML portfolio page
- **Contact page** — form with database storage

### AI Integration
- Uses Replit AI Integrations (OpenAI proxy) — `gpt-5-mini` for all analysis/generation tasks
- Env vars: `AI_INTEGRATIONS_OPENAI_BASE_URL`, `AI_INTEGRATIONS_OPENAI_API_KEY` (auto-provisioned)
- Server package: `@workspace/integrations-openai-ai-server`

### Routes
- `GET /` — Landing page
- `GET /builder` — Resume builder editor
- `GET /contact` — Contact form
- `GET /ats-analyzer` — ATS Resume Analyzer
- `GET /bullet-improver` — AI Bullet Enhancer
- `GET /job-matcher` — Job Description Matcher
- `GET /portfolio-generator` — Portfolio Generator (step-by-step)
- `GET /portfolio/:slug` — View generated portfolio

### API Endpoints
- `POST /api/resumes` — Save a resume
- `GET /api/resumes` — List saved resumes
- `POST /api/resumes/generate-pdf` — Generate and download resume as PDF
- `POST /api/contacts` — Submit contact form message

### Database Tables
- `resumes` — id, name, email, phone, education, experience, skills, template, created_at
- `contacts` — id, name, email, message, created_at

## Structure

```text
artifacts-monorepo/
├── artifacts/              # Deployable applications
│   ├── api-server/         # Express API server
│   │   └── src/
│   │       ├── lib/pdf-generator.ts    # PDF generation with pdfkit
│   │       └── routes/
│   │           ├── resumes.ts          # Resume CRUD + PDF
│   │           └── contacts.ts         # Contact form
│   └── resumease/          # React + Vite frontend
│       └── src/
│           ├── pages/      # Home, Builder, Contact
│           ├── components/ # Navbar, Footer, ResumePreview
│           └── App.tsx
├── lib/                    # Shared libraries
│   ├── api-spec/           # OpenAPI spec + Orval codegen config
│   ├── api-client-react/   # Generated React Query hooks
│   ├── api-zod/            # Generated Zod schemas from OpenAPI
│   └── db/
│       └── src/schema/
│           ├── resumes.ts
│           └── contacts.ts
└── scripts/                # Utility scripts
```

## TypeScript & Composite Projects

Every package extends `tsconfig.base.json` which sets `composite: true`. The root `tsconfig.json` lists all packages as project references. This means:

- **Always typecheck from the root** — run `pnpm run typecheck` (which runs `tsc --build --emitDeclarationOnly`). This builds the full dependency graph so that cross-package imports resolve correctly. Running `tsc` inside a single package will fail if its dependencies haven't been built yet.
- **`emitDeclarationOnly`** — we only emit `.d.ts` files during typecheck; actual JS bundling is handled by esbuild/tsx/vite...etc, not `tsc`.
- **Project references** — when package A depends on package B, A's `tsconfig.json` must list B in its `references` array. `tsc --build` uses this to determine build order and skip up-to-date packages.

## Root Scripts

- `pnpm run build` — runs `typecheck` first, then recursively runs `build` in all packages that define it
- `pnpm run typecheck` — runs `tsc --build --emitDeclarationOnly` using project references

## Key Notes

- pdfkit is **externalized** from esbuild bundle (fontkit dependency issue with @swc/helpers)
- PDF generation supports 3 template styles: modern, classic, minimal
- Frontend uses React Hook Form + Zod for validation
- Framer Motion for animations
