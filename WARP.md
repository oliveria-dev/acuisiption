# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a Node.js backend API for an acquisitions system built with Express.js. The application uses a modern ES modules setup with TypeScript-like features through Node.js's built-in import maps and path aliases.

## Key Technologies

- **Runtime**: Node.js (ES modules)
- **Framework**: Express.js 5.x
- **Database**: PostgreSQL with Neon serverless
- **ORM**: Drizzle ORM
- **Authentication**: JWT with bcrypt password hashing
- **Validation**: Zod schemas
- **Logging**: Winston
- **Code Quality**: ESLint + Prettier

## Common Development Commands

### Development

```bash
# Start development server with file watching
npm run dev

# Start production server
npm start
```

### Database Operations

```bash
# Generate database migrations
npm run db:generate

# Apply database migrations
npm run db:migrate

# Open Drizzle Studio (database GUI)
npm run db:studio
```

### Code Quality

```bash
# Run linting
npm run lint

# Fix linting issues automatically
npm run lint:fix

# Format code with Prettier
npm run format

# Check code formatting
npm run format:check
```

## Architecture Overview

### Entry Point Flow

- `src/index.js` → loads environment variables and imports `src/server.js`
- `src/server.js` → imports Express app and starts server
- `src/app.js` → main Express application setup with middleware and routes

### Directory Structure and Import Aliases

The project uses Node.js import maps for clean module imports:

- `#config/*` → `./src/config/*` (database, logger configuration)
- `#controllers/*` → `./src/controllers/*` (request handlers)
- `#middleware/*` → `./src/middleware/*` (custom middleware - currently empty)
- `#models/*` → `./src/models/*` (Drizzle ORM models)
- `#routes/*` → `./src/routes/*` (Express route definitions)
- `#services/*` → `./src/services/*` (business logic layer)
- `#utils/*` → `./src/utils/*` (utility functions)
- `#validations/*` → `./src/validations/*` (Zod validation schemas)

### Application Layers

1. **Routes** (`src/routes/`) - Express route definitions
2. **Controllers** (`src/controllers/`) - Handle HTTP requests/responses
3. **Services** (`src/services/`) - Business logic and data operations
4. **Models** (`src/models/`) - Drizzle ORM table definitions
5. **Validations** (`src/validations/`) - Zod schema validations
6. **Utils** (`src/utils/`) - Shared utility functions

### Database Architecture

- Uses Drizzle ORM with PostgreSQL
- Neon serverless database connection
- Schema files in `src/models/`
- Migrations generated in `drizzle/` directory
- Connection configured in `src/config/database.js`

### Authentication Flow

- JWT-based authentication with httpOnly cookies
- Password hashing with bcrypt (10 rounds)
- User model with roles (user/admin)
- Token utilities in `src/utils/jwt.js`
- Cookie utilities in `src/utils/cookies.js`

### Logging Strategy

- Winston logger configured in `src/config/logger.js`
- File logging: `logs/error.log` and `logs/combined.log`
- Console logging in development
- Morgan HTTP request logging integrated with Winston

## Environment Configuration

Required environment variables (create `.env` file):

```env
PORT=3000
DATABASE_URL=postgresql://...
JWT_SECRET=your-secret-key-here
NODE_ENV=development
LOG_LEVEL=info
```

## Development Patterns

### Error Handling

- Controllers use try-catch with next(error) for error propagation
- Custom error messages for business logic violations
- Validation errors formatted using `formatValidationError` utility

### Validation Pattern

- Zod schemas defined in `src/validations/`
- Controllers validate using `schema.safeParse()`
- Formatted error responses for validation failures

### Service Layer Pattern

- Business logic separated into service functions
- Database operations abstracted from controllers
- Reusable functions for common operations

## Code Style Guidelines

### ESLint Configuration

- 2-space indentation
- Single quotes for strings
- Semicolons required
- Unix line endings
- Arrow functions preferred
- No unused variables (except with `_` prefix)

### Import/Export Patterns

- ES modules with `import`/`export`
- Use path aliases (`#config/*`) instead of relative paths
- Named exports for utilities, default exports for main modules

## Testing Strategy

Currently no tests are set up, but the project structure supports adding:

- Unit tests for services and utilities
- Integration tests for API endpoints
- Database tests using test database

## Development Workflow

1. Make changes to source files
2. The `--watch` flag in `npm run dev` automatically restarts the server
3. Use `npm run lint` and `npm run format` before committing
4. Database schema changes require running `npm run db:generate` and `npm run db:migrate`

## Key Files to Understand

- `src/app.js` - Express app configuration and middleware setup
- `src/config/database.js` - Database connection and Drizzle setup
- `drizzle.config.js` - Drizzle ORM configuration
- `eslint.config.js` - Code linting rules and patterns
