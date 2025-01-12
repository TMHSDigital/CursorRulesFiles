# Full-Stack Next.js Project Example

This example demonstrates how to use CursorRules in a full-stack Next.js project with App Router.

## Project Setup

1. Create project directory structure:
```
/my-nextjs-project
  /.cursorrules              # Directory for rule files
    project.cursorrules      # Core project rules
    frontend.cursorrules     # Frontend rules
    backend.cursorrules      # API rules
  /src
    /app                     # Next.js App Router
      /api                   # API routes
      /(auth)               # Auth group
      /(dashboard)          # Dashboard group
      /layout.tsx           # Root layout
    /components             # React components
      /ui                   # UI components
      /forms               # Form components
      /layouts            # Layout components
    /lib                    # Shared utilities
      /api                 # API utilities
      /db                  # Database utilities
      /auth               # Auth utilities
    /styles                 # Global styles
  /prisma                   # Database schema
  /public                   # Static assets
  package.json
  next.config.js
```

2. Copy relevant rules:
```bash
# Copy all needed rules
cp rules/core/project.cursorrules .cursorrules/
cp rules/frontend/frontend.cursorrules .cursorrules/
cp rules/backend/backend.cursorrules .cursorrules/
cp rules/testing/testing.cursorrules .cursorrules/
```

## Rule Customization

### Project Rules
```
// project.cursorrules
// Full-stack Next.js setup

// Technology Stack
- Framework: Next.js 14 (App Router)
- Language: TypeScript
- Database: PostgreSQL + Prisma
- Authentication: NextAuth.js
- UI: Tailwind + Shadcn/ui
- State: React Query + Zustand

// Project Structure
/src
  /app                   # Next.js App Router
    /api               # API endpoints
    /(auth)           # Auth routes
    /(dashboard)      # Protected routes
  /components          # React components
    /ui              # UI components
    /forms           # Form components
  /lib                # Shared code
    /api            # API utilities
    /db             # Database
```

### Frontend Rules
```
// frontend.cursorrules
// Next.js specific patterns

// Page Structure
// app/(dashboard)/items/page.tsx
import { Suspense } from 'react'
import { ItemsList, ItemsSkeleton } from '@/components/items'

export default function ItemsPage() {
  return (
    <main className="container py-6">
      <h1 className="text-3xl font-bold">Items</h1>
      <Suspense fallback={<ItemsSkeleton />}>
        <ItemsList />
      </Suspense>
    </main>
  )
}

// Server Component
// components/items/items-list.tsx
import { getItems } from '@/lib/api/items'

export async function ItemsList() {
  const items = await getItems()
  
  return (
    <div className="grid gap-4">
      {items.map(item => (
        <ItemCard key={item.id} item={item} />
      ))}
    </div>
  )
}

// Client Component
'use client'

import { useOptimistic } from 'react'
import { updateItem } from '@/lib/api/items'

export function ItemCard({ item }) {
  const [optimisticItem, updateOptimisticItem] = useOptimistic(
    item,
    (state, update) => ({ ...state, ...update })
  )

  async function handleUpdate(data) {
    updateOptimisticItem(data)
    await updateItem(item.id, data)
  }

  return (
    <div className="card">
      {/* Card content */}
    </div>
  )
}
```

### Backend Rules
```
// backend.cursorrules
// Next.js API routes

// API Route Handler
// app/api/items/route.ts
import { NextResponse } from 'next/server'
import { getServerSession } from 'next-auth'
import { z } from 'zod'

import { db } from '@/lib/db'
import { authOptions } from '@/lib/auth'

const createItemSchema = z.object({
  title: z.string().min(1).max(100),
  description: z.string().optional(),
})

export async function POST(req: Request) {
  try {
    // Auth check
    const session = await getServerSession(authOptions)
    if (!session) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      )
    }

    // Validation
    const json = await req.json()
    const body = createItemSchema.parse(json)

    // Database operation
    const item = await db.item.create({
      data: {
        ...body,
        userId: session.user.id,
      },
    })

    return NextResponse.json(item)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: error.issues },
        { status: 400 }
      )
    }

    return NextResponse.json(
      { error: 'Internal Server Error' },
      { status: 500 }
    )
  }
}
```

## Database Integration

### Prisma Schema
```prisma
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  items     Item[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Item {
  id          String   @id @default(cuid())
  title       String
  description String?
  user        User     @relation(fields: [userId], references: [id])
  userId      String
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([userId])
}
```

### Database Utilities
```typescript
// lib/db.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const db = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = db
}
```

## Testing Setup

### Component Testing
```typescript
// components/ui/button.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Button } from './button'

describe('Button', () => {
  it('should_render_button_with_children', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('should_handle_click_events', async () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click me</Button>)
    
    await userEvent.click(screen.getByText('Click me'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

### API Testing
```typescript
// app/api/items/route.test.ts
import { createMocks } from 'node-mocks-http'
import { POST } from './route'

describe('/api/items', () => {
  it('should_create_item_when_authenticated', async () => {
    const { req, res } = createMocks({
      method: 'POST',
      body: {
        title: 'Test Item',
        description: 'Test Description',
      },
    })

    const response = await POST(req)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.title).toBe('Test Item')
  })
})
```

## Best Practices

1. Next.js patterns:
   - Use App Router features
   - Implement proper loading states
   - Handle errors gracefully
   - Optimize for performance

2. Full-stack considerations:
   - Share types between frontend/backend
   - Handle API errors consistently
   - Implement proper validation
   - Use proper authentication

3. Database practices:
   - Use Prisma migrations
   - Implement proper relations
   - Handle transactions
   - Optimize queries

4. Testing approach:
   - Test components in isolation
   - Test API endpoints
   - Test database operations
   - Test authentication flows 