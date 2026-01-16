# 005. useState-Less Pattern (Avoid useState for Server Data)

**Date**: 2025-10-15
**Status**: Accepted
**Deciders**: Engineering Team

---

## Context

When integrating Convex (real-time database) with React, there are two patterns for managing server data:

### Pattern A: Traditional useState + useEffect

```typescript
'use client';
import { useState, useEffect } from 'react';

function CourseList() {
  const [courses, setCourses] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/courses')
      .then(res => res.json())
      .then(data => {
        setCourses(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;
  return <div>{courses.map(/* ... */)}</div>;
}
```

### Pattern B: Convex Hooks (useState-Less)

```typescript
'use client';
import { useQuery } from 'convex/react';
import { api } from '@/convex/_generated/api';

function CourseList() {
  const courses = useQuery(api.courses.list);

  if (courses === undefined) return <div>Loading...</div>;
  return <div>{courses.map(/* ... */)}</div>;
}
```

## Decision

We adopt **Pattern B: useState-Less** for all server data in Doctrina LMS.

**Rule**: **Never use useState/useEffect to manage data from Convex.** Always use Convex hooks (`useQuery`, `useMutation`).

## Rationale

### Why useState-Less is Better

#### 1. Real-Time Updates Automatic

**With useState**:

- Data is stale immediately after fetch
- Must manually poll or setup WebSocket
- Complexity increases

**With Convex Hooks**:

- Data updates automatically when changed
- No polling, no WebSocket setup
- Real-time by default

**Example**: Student marks lesson complete → Progress bar updates for instructor in real-time (no refresh needed)

#### 2. Less Code, Fewer Bugs

**With useState**:

- Manage loading state manually
- Manage error state manually
- Manage stale data manually
- Risk of memory leaks (unmounted component fetches)

**With Convex Hooks**:

- Loading state handled by `undefined`
- Errors thrown automatically
- Stale data impossible
- Cleanup handled by Convex

**Result**: ~50% less code, fewer bugs

#### 3. Type Safety

**With useState**:

```typescript
const [courses, setCourses] = useState<Course[]>([]); // Must type manually
```

**With Convex Hooks**:

```typescript
const courses = useQuery(api.courses.list); // Type inferred from Convex schema
```

**Result**: Automatic type inference, no manual typing

#### 4. Optimistic Updates

**With useState**:

- Must manually implement optimistic UI
- Complex rollback logic on error

**With Convex**:

- Optimistic updates built-in via `useOptimistic` (coming in Convex)
- Automatic rollback on error

---

## Consequences

### Positive

1. **Real-Time by Default**: All data updates automatically
2. **Less Code**: No useState, no useEffect for data fetching
3. **Type-Safe**: Types inferred from Convex schema
4. **Better UX**: Users see updates instantly
5. **Fewer Bugs**: No stale data, no race conditions

### Negative

1. **Convex Dependency**: Pattern tied to Convex (migration would require rewrite)
2. **Learning Curve**: Team must unlearn useState for server data
3. **Third-Party Integration**: External APIs still need useState/useEffect
4. **Testing**: Must use convex-test for testing (not plain Jest)

---

## Implementation Patterns

### ✅ GOOD - Convex Hooks

```typescript
'use client';
import { useQuery, useMutation } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function CourseCard({ courseId }) {
  const course = useQuery(api.courses.get, { courseId });
  const enroll = useMutation(api.enrollments.create);

  if (course === undefined) return <LoadingSkeleton />;
  if (course === null) return <NotFound />;

  return (
    <div>
      <h2>{course.title}</h2>
      <button onClick={() => enroll({ courseId })}>
        Enroll
      </button>
    </div>
  );
}
```

### ❌ BAD - useState + useEffect

```typescript
'use client';
import { useState, useEffect } from 'react';

export function CourseCard({ courseId }) {
  const [course, setCourse] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/courses/${courseId}`)
      .then(res => res.json())
      .then(data => {
        setCourse(data);
        setLoading(false);
      });
  }, [courseId]);

  if (loading) return <LoadingSkeleton />;
  if (!course) return <NotFound />;

  return (
    <div>
      <h2>{course.title}</h2>
      {/* No real-time updates, stale data */}
    </div>
  );
}
```

### Exceptions: When useState is OK

✅ **Use useState for**:

- **UI state** (modals open/closed, form inputs, local filters)
- **Non-Convex data** (third-party APIs, localStorage)
- **Derived state** (computed from props/Convex data)

```typescript
'use client';
import { useState } from 'react';
import { useQuery } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function CourseList() {
  // ✅ OK - UI state (modal)
  const [modalOpen, setModalOpen] = useState(false);

  // ✅ OK - Convex data (real-time)
  const courses = useQuery(api.courses.list);

  // ✅ OK - Derived state (filtered)
  const [filter, setFilter] = useState('');
  const filtered = courses?.filter(c => c.title.includes(filter));

  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      {filtered?.map(course => <CourseCard key={course._id} course={course} />)}
      <Modal open={modalOpen} onClose={() => setModalOpen(false)} />
    </div>
  );
}
```

---

## Handling Loading States

### Pattern 1: Inline Check

```typescript
const courses = useQuery(api.courses.list);

if (courses === undefined) return <LoadingSkeleton />;

return <div>{courses.map(/* ... */)}</div>;
```

### Pattern 2: Optional Chaining

```typescript
const courses = useQuery(api.courses.list);

return (
  <div>
    {courses?.map(course => (
      <CourseCard key={course._id} course={course} />
    )) || <LoadingSkeleton />}
  </div>
);
```

### Pattern 3: Suspense (Future)

```typescript
// Future: Convex + React Suspense
<Suspense fallback={<LoadingSkeleton />}>
  <CourseList />
</Suspense>
```

---

## Alternatives Considered

### Alternative 1: React Query + REST API

**Pros**:

- Well-known library
- Works with any backend
- Good caching

**Cons**:

- More setup than Convex hooks
- No real-time (must add WebSocket)
- More code than Convex

**Why Not**: Convex hooks provide real-time + less code

### Alternative 2: SWR + REST API

**Pros**:

- Simple API
- Good caching
- Works with any backend

**Cons**:

- No real-time
- Must manage loading/error states
- More code than Convex

**Why Not**: Convex hooks provide real-time + automatic state management

### Alternative 3: Redux/Zustand + Convex

**Pros**:

- Centralized state management
- Can combine Convex data with other state

**Cons**:

- Overengineering (Convex already manages server state)
- More boilerplate
- Duplicates Convex's caching

**Why Not**: Convex hooks are sufficient for server state. Use React Context for global UI state if needed.

---

## Migration Considerations

### If We Migrate Away from Convex

**Scenario**: Switch from Convex to PostgreSQL + REST API

**Migration Path**:

1. Replace `useQuery` with React Query or SWR
2. Replace `useMutation` with async functions
3. Add loading/error state management
4. Add WebSocket or polling for real-time (if needed)

**Code Change**:

```typescript
// Before (Convex)
const courses = useQuery(api.courses.list);

// After (React Query + REST)
const { data: courses, isLoading } = useQuery('courses', () => fetch('/api/courses').then(res => res.json()));
```

**Estimated Effort**: 1-2 weeks to replace all Convex hooks

---

## Key Guidelines

### DOs ✅

- **DO** use `useQuery` for reading Convex data
- **DO** use `useMutation` for writing Convex data
- **DO** use `useState` for UI state (modals, form inputs)
- **DO** handle `undefined` state (loading)
- **DO** handle `null` state (not found)

### DON'Ts ❌

- **DON'T** use `useState` to store Convex data
- **DON'T** use `useEffect` to fetch Convex data
- **DON'T** manually poll for data updates
- **DON'T** implement your own caching layer
- **DON'T** use Redux/Zustand for Convex data

---

## References

- [Convex React Integration](https://docs.convex.dev/client/react)
- [useQuery Hook](https://docs.convex.dev/api/modules/react#usequery)
- [useMutation Hook](https://docs.convex.dev/api/modules/react#usemutation)
- Project File: `.claude/standards/react-convex.md` (React + Convex patterns)
- Project File: `_bmad-output/project-context.md` (Anti-patterns section)

---

**Last Updated**: 2026-01-15
**Next Review**: After React 19 stable release
