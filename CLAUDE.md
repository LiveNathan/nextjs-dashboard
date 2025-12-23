# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 15 dashboard application (based on the Next.js App Router Course) that demonstrates modern React patterns with TypeScript. It's a financial dashboard for managing customers and invoices with authentication support via NextAuth.

## Development Commands

```bash
# Start development server with Turbopack
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Seed the database with initial data
# Navigate to http://localhost:3000/seed after starting dev server
```

## Architecture

### Directory Structure

- `app/` - Next.js App Router directory
  - `lib/` - Core utilities and data layer
    - `definitions.ts` - TypeScript type definitions for all data models
    - `data.ts` - Database queries using postgres.js
    - `utils.ts` - Utility functions (currency formatting, date formatting, pagination)
    - `placeholder-data.ts` - Seed data for development
  - `ui/` - React components organized by feature
    - `dashboard/` - Dashboard-specific components (cards, charts, navigation)
    - `invoices/` - Invoice management components (forms, tables, status badges)
    - `customers/` - Customer management components
  - `page.tsx` - Homepage
  - `layout.tsx` - Root layout with global styles

### Data Layer

The application uses **postgres.js** (not Prisma) for database access. All database queries are in `app/lib/data.ts`:
- Connection string is read from `POSTGRES_URL` environment variable with SSL required
- All queries use tagged template literals: `sql<Type[]>\`SELECT...\``
- Currency amounts are stored in cents in the database and formatted to dollars in the UI

Key data access patterns:
- Pagination uses `ITEMS_PER_PAGE = 6` constant with LIMIT/OFFSET
- Search uses `ILIKE` for case-insensitive pattern matching
- Parallel query execution with `Promise.all()` for dashboard cards

### Type System

All types are manually defined in `app/lib/definitions.ts`. Key entities:
- `User`, `Customer`, `Invoice` (with 'pending' | 'paid' status)
- `Revenue` - Monthly revenue data for charts
- Separate types for forms vs display (e.g., `InvoiceForm` vs `InvoicesTable`)
- Raw vs formatted types (e.g., `LatestInvoiceRaw` has numeric amount, `LatestInvoice` has formatted string)

### Styling

- **Tailwind CSS v4** with custom configuration in `tailwind.config.ts`
- Custom colors: blue.400, blue.500, blue.600
- Custom grid: `grid-cols-13` for complex layouts
- Shimmer animation defined in keyframes
- `@tailwindcss/forms` plugin included
- Global styles in `app/ui/global.css`

### Path Aliases

TypeScript is configured with `@/*` alias mapping to the root directory. Use `@/app/...` for imports.

## Environment Variables

Required variables (see `.env.example`):
- `POSTGRES_URL` - Main PostgreSQL connection string
- `POSTGRES_PRISMA_URL` - Prisma-compatible connection string
- `POSTGRES_URL_NON_POOLING` - Non-pooling connection
- `POSTGRES_USER`, `POSTGRES_HOST`, `POSTGRES_PASSWORD`, `POSTGRES_DATABASE`
- `AUTH_SECRET` - Generate with `openssl rand -base64 32`
- `AUTH_URL` - Default: `http://localhost:3000/api/auth`

## Package Manager

This project uses **pnpm** (not npm or yarn). Note the special configuration in package.json for building native dependencies (bcrypt, sharp).

## Authentication

The project includes NextAuth v5 beta configuration (though auth files may not be fully implemented in this starter template). Authentication logic would typically be in `auth.ts` and `auth.config.ts` at the root.

## Database Seeding

The `/seed` route handler (`app/seed/route.ts`) can be accessed via HTTP to populate the database with initial data from `app/lib/placeholder-data.ts`.
