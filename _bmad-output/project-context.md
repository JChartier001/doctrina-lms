# Doctrina LMS - Project Context

**Last Updated**: 2026-01-15
**Project Type**: Medical Aesthetics Learning Marketplace
**Phase**: Implementation (Phase 4)
**Sprint**: Sprint 2 - Backend Development

---

## Critical Project Information

### What This Document Is

This is the **bible** that all BMAD agents reference before implementing ANY code. It consolidates:
- Tech stack decisions
- Coding patterns and standards
- Project-specific constraints
- What NOT to do (anti-patterns)

**When implementing:** Always check this file first. When conflicts arise between this file and story requirements, story requirements take precedence, but this file guides HOW to implement.

---

## Project Overview

### Vision

Doctrina LMS is a specialized online learning marketplace connecting **medical aesthetics professionals** with expert instructors offering courses in injectables, lasers, medical spa operations, and advanced aesthetic procedures.

### Target Users

- **Students**: Nurses, Estheticians, Physicians, PAs seeking credentialed education
- **Instructors**: Experienced practitioners teaching medical aesthetics
- **Admins**: Platform management and content moderation

### Strategic Goals

1. Launch MVP with 50+ courses and 50+ verified instructors by Month 3
2. Establish trust through rigorous instructor vetting and CE credit offerings
3. Drive engagement with 70% course completion rate (vs. industry avg 15%)
4. Enable income for instructors with 93% revenue share (Elite tier)

---

## Tech Stack

### Frontend

- **Framework**: Next.js 16.0.1 (App Router)
- **UI Library**: React 19.2.0
- **Styling**: Tailwind CSS 4.1.17
- **Components**: Radix UI (shadcn/ui pattern)
- **Forms**: React Hook Form 7.66.0 + Zod 4.1.12
- **Animation**: Framer Motion 12.23.24
- **Charts**: Recharts 3.3.0
- **Language**: TypeScript 5.9.3 (strict mode)

### Backend

- **Database/Backend**: Convex 1.28.2 (serverless, real-time)
- **Authentication**: Clerk 6.34.5
- **Payments**: Stripe 19.3.0

### Development

- **Testing**: Vitest 4.0.8
- **Package Manager**: Yarn 4.10.3
- **Node Version**: 24.10.0 (Volta)
- **Git Branch**: master

---

## Architecture Principles

### Core Patterns

1. **Server Components by Default** - Use 'use client' only when needed (interactivity, hooks)
2. **Convex for All Backend** - No REST APIs, no direct database access
3. **Type-Safe End-to-End** - TypeScript strict mode, Zod for runtime validation
4. **Real-Time by Default** - Convex provides live data updates without polling
5. **Test-Driven Development** - 100% test coverage required for all backend code

### Design Philosophy

- **Serverless-First**: Minimize infrastructure management
- **Real-Time by Default**: Live data updates without polling
- **Type-Safe End-to-End**: TypeScript from database to UI
- **Progressive Enhancement**: Works with mocked services, scales to production
- **Separation of Concerns**: Clear boundaries between presentation, business logic, and data

---

## Critical Coding Standards

### React Patterns

#### 1. Server vs Client Components

```typescript
// ✅ GOOD - Server Component (default)
export default async function CoursesPage() {
  const { userId } = await auth();
  return <div>Courses</div>;
}

// ✅ GOOD - Client Component (only when needed)
'use client';
import { useQuery } from 'convex/react';

export function CourseList() {
  const courses = useQuery(api.courses.list);
  return <div>{/* Interactive UI */}</div>;
}

// ❌ BAD - Unnecessary 'use client'
'use client';
export function StaticHeader() {
  return <h1>Welcome</h1>; // No interactivity needed
}
```

#### 2. Convex Data Fetching Pattern

```typescript
// ✅ GOOD - Convex hooks for real-time data
'use client';
import { useQuery, useMutation } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function CourseCard({ courseId }) {
  const course = useQuery(api.courses.get, { courseId });
  const enroll = useMutation(api.enrollments.create);

  if (course === undefined) return <LoadingSkeleton />;
  if (course === null) return <NotFound />;

  return <Card>{course.title}</Card>;
}

// ❌ BAD - useState for data (violates useState-less pattern)
const [course, setCourse] = useState(null);
useEffect(() => {
  fetch('/api/courses').then(/* ... */);
}, []);

// ❌ BAD - REST API calls
fetch('/api/courses');
```

#### 3. Form Pattern (React Hook Form + Zod + Controller)

```typescript
// ✅ GOOD - FormProvider + Controller pattern
'use client';
import { FormProvider, Controller, useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  title: z.string().min(1, 'Required'),
  description: z.string().min(10, 'Too short'),
});

type FormData = z.infer<typeof schema>;

export function CourseForm() {
  const methods = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        <Controller
          control={methods.control}
          name="title"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Title</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
      </form>
    </FormProvider>
  );
}

// ❌ BAD - Uncontrolled forms
<form onSubmit={handleSubmit}>
  <input name="title" />
</form>
```

### Convex Backend Patterns

#### 1. Authentication

```typescript
// ✅ GOOD - Always check auth in mutations
export const createCourse = mutation({
  args: { title: v.string() },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error('Unauthorized');

    const user = await getUserByClerkId(ctx, identity.subject);
    if (!user?.isInstructor) throw new Error('Forbidden');

    return await ctx.db.insert('courses', { ...args });
  },
});

// ❌ BAD - No auth check
export const createCourse = mutation({
  handler: async (ctx, args) => {
    return await ctx.db.insert('courses', args); // Anyone can create!
  },
});
```

#### 2. Error Handling

```typescript
// ✅ GOOD - Descriptive error messages
if (!enrollment) {
  throw new Error('Not enrolled in this course');
}

if (quiz.maxAttempts && attempts.length >= quiz.maxAttempts) {
  throw new Error(`Maximum attempts (${quiz.maxAttempts}) reached`);
}

// ❌ BAD - Generic errors
if (!enrollment) throw new Error('Error');
```

#### 3. Queries vs Mutations

```typescript
// ✅ GOOD - Queries for reads, mutations for writes
export const list = query({
  handler: async (ctx) => {
    return await ctx.db.query('courses').collect();
  },
});

export const create = mutation({
  args: { title: v.string() },
  handler: async (ctx, args) => {
    return await ctx.db.insert('courses', args);
  },
});

// ❌ BAD - Mutation for reading data
export const list = mutation({ /* ... */ });
```

### TypeScript Standards

```typescript
// ✅ GOOD - Explicit types
interface CourseCardProps {
  courseId: Id<'courses'>;
  onEnroll: (courseId: Id<'courses'>) => void;
}

export function CourseCard({ courseId, onEnroll }: CourseCardProps) {
  // ...
}

// ❌ BAD - Implicit any
export function CourseCard(props) {
  // TypeScript doesn't know what props contains
}

// ✅ GOOD - Zod for runtime validation
const createCourseSchema = z.object({
  title: z.string().min(1),
  price: z.number().min(0),
});

// ❌ BAD - No runtime validation
function createCourse(data: any) {
  // data could be anything at runtime
}
```

---

## File Organization

### Directory Structure

```
doctrina-lms/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth route group
│   ├── (dashboard)/              # Dashboard route group
│   ├── api/                      # API routes (webhooks only)
│   └── page.tsx                  # Home page
├── components/                   # React components
│   ├── ui/                       # shadcn/ui components
│   └── [feature]/                # Feature-specific components
├── convex/                       # Convex backend
│   ├── schema.ts                 # Database schema
│   ├── _generated/               # Generated types
│   ├── __test__/                 # Convex tests (Vitest)
│   └── [table].ts                # Queries/mutations by table
├── lib/                          # Utility functions
├── hooks/                        # Custom React hooks
├── types/                        # TypeScript type definitions
├── docs/                         # Project documentation
│   ├── stories/                  # Story files
│   ├── EPICS.md                  # Epic definitions
│   ├── PRD.md                    # Product requirements
│   └── sprint-status.yaml        # Sprint tracking
└── _bmad-output/                 # BMAD documentation
    ├── project-context.md        # THIS FILE
    ├── active-sprint.md          # Current sprint info
    └── epic-summaries/           # Epic tech context
```

### File Naming Conventions

- **Components**: PascalCase (e.g., `CourseCard.tsx`)
- **Convex files**: camelCase (e.g., `lessonProgress.ts`)
- **Utilities**: kebab-case (e.g., `format-date.ts`)
- **Tests**: Same as file + `.test.ts` (e.g., `CourseCard.test.tsx`)

---

## Database Schema (Convex)

### Key Tables

1. **users** - All platform users (students, instructors, admins)
   - Key fields: `isInstructor`, `isAdmin` (booleans)
   - Indexes: `by_email`, `by_externalId` (Clerk ID)

2. **courses** - Educational content
   - Key fields: `instructorId`, `price`, `visibility`, `category`
   - Indexes: `by_instructor`, `by_visibility`

3. **courseModules** - Course curriculum sections
   - Key fields: `courseId`, `title`, `order`
   - Indexes: `by_course`

4. **lessons** - Individual lessons within modules
   - Key fields: `moduleId`, `type` (video|document|quiz|assignment)
   - Indexes: `by_module`

5. **quizzes** - Quiz assessments
   - Key fields: `courseId`, `moduleId`, `passingScore`
   - Indexes: `by_course`, `by_module`

6. **quizQuestions** - Questions for quizzes
   - Key fields: `quizId`, `correctAnswer`, `options[]`
   - Indexes: `by_quiz`

7. **quizAttempts** - Student quiz submissions
   - Key fields: `userId`, `quizId`, `score`, `passed`
   - Indexes: `by_user_quiz`

8. **lessonProgress** - Lesson completion tracking
   - Key fields: `userId`, `lessonId`, `completedAt`
   - Indexes: `by_user_lesson`, `by_user`

9. **enrollments** - Course enrollments
   - Key fields: `userId`, `courseId`, `progressPercent`
   - Indexes: `by_user_course`

10. **certificates** - Completion certificates
    - Key fields: `userId`, `courseId`, `verificationCode`
    - Indexes: `by_user`, `by_verification`

**Full schema**: See `docs/DB-SCHEMA.md` for complete details

---

## Testing Requirements

### Coverage Standards

- **ALL Convex backend functions: 100% test coverage** (no exceptions)
- **Critical financial code: 100%** (purchases, payments, enrollments)
- **Frontend components: 85%** (unit + integration tests)

### Testing Framework

```typescript
// convex/__test__/lessonProgress.test.ts
import { convexTest } from 'convex-test';
import { describe, it, expect } from 'vitest';
import schema from '../schema';
import { markComplete } from '../lessonProgress';

describe('lessonProgress', () => {
  it('should mark lesson as complete', async () => {
    const t = convexTest(schema);

    const userId = await t.run(async (ctx) => {
      return await ctx.db.insert('users', {
        firstName: 'Test',
        lastName: 'User',
        email: 'test@example.com',
        isInstructor: false,
        isAdmin: false,
      });
    });

    // ... rest of test
  });
});
```

### Test Patterns

- Use `convex-test` for Convex backend tests
- Use `@testing-library/react` for component tests
- Mock external services (Stripe, Clerk)
- Test error cases and edge cases
- Follow patterns from `convex/__test__/lessonProgress.test.ts`

---

## Security Constraints

### Authentication

- **Always** verify `ctx.auth.getUserIdentity()` in mutations
- Check role-based permissions (`isInstructor`, `isAdmin`)
- Validate user owns resources before modifications

### Input Validation

- Use Zod schemas for all user input
- Validate in both frontend AND backend
- Sanitize HTML content (use DOMPurify)
- Never trust client-side validation alone

### Payment Security

- Never expose Stripe secret keys in client
- Validate webhook signatures (Stripe, Clerk)
- Store prices in cents (avoid floating point)
- Verify purchase before granting access

---

## Anti-Patterns (What NOT to Do)

### ❌ Don't Use useState for Server Data

```typescript
// ❌ BAD
const [courses, setCourses] = useState([]);
useEffect(() => {
  fetch('/api/courses').then(data => setCourses(data));
}, []);

// ✅ GOOD
const courses = useQuery(api.courses.list);
```

### ❌ Don't Create REST APIs

```typescript
// ❌ BAD - Creating REST endpoints
export async function GET(request: Request) {
  // Custom API route
}

// ✅ GOOD - Use Convex functions
export const list = query({
  handler: async (ctx) => {
    return await ctx.db.query('courses').collect();
  },
});
```

### ❌ Don't Use Default Exports

```typescript
// ❌ BAD
export default function CourseCard() {}

// ✅ GOOD
export function CourseCard() {}
```

### ❌ Don't Skip Error Handling

```typescript
// ❌ BAD
const course = await ctx.db.get(courseId);
await ctx.db.patch(course._id, { title: 'New' }); // course could be null!

// ✅ GOOD
const course = await ctx.db.get(courseId);
if (!course) throw new Error('Course not found');
await ctx.db.patch(course._id, { title: 'New' });
```

### ❌ Don't Ignore Loading States

```typescript
// ❌ BAD
const courses = useQuery(api.courses.list);
return <div>{courses.map(/* ... */)}</div>; // undefined crashes!

// ✅ GOOD
const courses = useQuery(api.courses.list);
if (courses === undefined) return <LoadingSkeleton />;
return <div>{courses.map(/* ... */)}</div>;
```

---

## Medical Aesthetics Domain Constraints

### Course Categories

- Botox & Neuromodulators
- Dermal Fillers
- Laser Treatments
- Chemical Peels
- Microneedling
- PDO Threads
- Medical Spa Operations
- Business & Marketing

### CE Credits

- Certificates must include CE credit information when applicable
- Instructors must submit accreditation details
- Admin approval required for CE credit courses

### Instructor Vetting

- License verification required (RN, MD, PA, NP)
- Insurance verification required
- Background check for platform integrity
- Manual admin approval process

---

## Current Sprint Context

**Sprint 2 (Weeks 3-4)**: Backend Development - Progress Tracking & Quiz System

**Active Epics:**
- EPIC-101: Lesson Progress Tracking (13 pts) - IN PROGRESS
- EPIC-102: Quiz Submission & Grading (25 pts) - IN PROGRESS

**Completed:**
- Sprint 1: Course Structure, Enrollments, Stripe Integration ✅

**See**: `_bmad-output/active-sprint.md` for detailed sprint information

---

## Development Workflow

### Before Starting Implementation

1. **Read this file** (`project-context.md`)
2. **Read the story file** (`docs/stories/story-XXX.md`)
3. **Read the story context** (`docs/stories/story-context-XXX.xml`)
4. **Check epic summary** (`_bmad-output/epic-summaries/epic-XXX.md`)
5. **Write tests FIRST** (TDD red-green-refactor)
6. **Implement** following patterns above
7. **Verify** all tests pass (100% coverage)

### Commands

```bash
# Development
yarn dev                  # Start Next.js + Convex

# Code Quality
yarn verify              # Format + Lint + TypeCheck + Test with coverage

# Testing
yarn test                # Run all tests
yarn test:coverage       # Coverage report

# Build
yarn build               # Production build
```

### Git Workflow

- Branch: `feature/[epic-number]-[description]`
- Commits: `feat: [description]` or `fix: [description]`
- Main branch: `master`

---

## Key Documentation References

### Within Project

- **PRD**: `docs/PRD.md` - Product requirements
- **Architecture**: `docs/ARCHITECTURE.md` - System architecture
- **Epics**: `docs/EPICS.md` - Epic definitions
- **DB Schema**: `docs/DB-SCHEMA.md` - Complete database schema
- **Stories**: `docs/stories/` - Implementation stories
- **Sprint Status**: `docs/sprint-status.yaml` - Current sprint tracking

### Standards (Legacy Droidz)

- **Backend Standards**: `droidz/standards/backend/`
- **Frontend Standards**: `droidz/standards/frontend/`
- **Global Standards**: `droidz/standards/global/`
- **Workflow**: `droidz/standards/RECOMMENDED_WORKFLOW.md`

### BMAD Documentation

- **Project Context**: `_bmad-output/project-context.md` (THIS FILE)
- **Active Sprint**: `_bmad-output/active-sprint.md`
- **Migration Notes**: `_bmad-output/migration-from-droidz.md`
- **Epic Summaries**: `_bmad-output/epic-summaries/`

---

## Critical Gotchas

### 1. Convex IDs are Type-Safe

```typescript
// ✅ GOOD
function getCourse(courseId: Id<'courses'>) {}

// ❌ BAD
function getCourse(courseId: string) {}
```

### 2. Handle Undefined from useQuery

```typescript
const data = useQuery(api.table.get);
// data is undefined during loading
// data is null if not found
// data is T if found
```

### 3. Mutations Return IDs

```typescript
const courseId = await ctx.db.insert('courses', { /* ... */ });
// courseId is Id<'courses'>, not the full object
```

### 4. Indexes Must Match Query Pattern

```typescript
// Schema has: .index('by_user_course', ['userId', 'courseId'])

// ✅ GOOD - Matches index
.withIndex('by_user_course', q =>
  q.eq('userId', userId).eq('courseId', courseId)
)

// ❌ BAD - Doesn't match index
.withIndex('by_user_course', q => q.eq('courseId', courseId))
```

### 5. Test Coverage is Mandatory

- NO exceptions for 100% backend coverage
- CI/CD will fail on coverage < 100%
- Write tests BEFORE implementation (TDD)

---

## Support & Questions

- **Codebase Documentation**: Check `docs/` folder first
- **Standards**: Reference `droidz/standards/` for patterns
- **Story Details**: Read story + context XML files
- **Epic Context**: Check `_bmad-output/epic-summaries/`

---

**Remember**: This file is your source of truth. When in doubt, follow these patterns. When conflicts arise, story requirements override, but implementation patterns from this file guide HOW to implement.

**Last Updated**: 2026-01-15
**Version**: 1.0 (BMAD Migration)
