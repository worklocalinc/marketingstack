# Scaffold Endpoint

Generate a new API endpoint following WorkLocal stack conventions.

## Tech Stack
- Node.js + TypeScript
- Express routes
- Drizzle ORM + Neon Postgres
- Zod for validation

## Directory Structure
```
services/<service-name>/
├── src/
│   ├── routes/      # Express route handlers
│   ├── schema/      # Drizzle schemas
│   ├── services/    # Business logic
│   └── validation/  # Zod schemas
```

## Task
Create endpoint: $ARGUMENTS

Generate the following files:

### 1. Zod Validation Schema
```typescript
// validation/<resource>.ts
import { z } from 'zod';

export const create<Resource>Schema = z.object({
  // fields with validation
});

export type Create<Resource>Input = z.infer<typeof create<Resource>Schema>;
```

### 2. Drizzle Schema (if new table needed)
```typescript
// schema/<resource>.ts
import { pgTable, uuid, text, timestamp } from 'drizzle-orm/pg-core';

export const <resources> = pgTable('<resources>', {
  id: uuid('id').primaryKey().defaultRandom(),
  // columns
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});
```

### 3. Service Layer
```typescript
// services/<resource>.ts
export async function create<Resource>(input: Create<Resource>Input) {
  // business logic
}
```

### 4. Route Handler
```typescript
// routes/<resource>.ts
import { Router } from 'express';
import { create<Resource>Schema } from '../validation/<resource>';

const router = Router();

router.post('/', async (req, res) => {
  const parsed = create<Resource>Schema.safeParse(req.body);
  if (!parsed.success) {
    return res.status(400).json({ error: parsed.error });
  }
  // call service
});

export default router;
```

### 5. Update service README
Add the new endpoint to `services/<service>/README.md`
