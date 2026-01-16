# 001. Convex as Backend

**Date**: 2025-10-01
**Status**: Accepted
**Deciders**: Product Team, Engineering Team

---

## Context

Doctrina LMS needed a backend solution that could:

- Handle real-time data updates (progress tracking, live sessions, notifications)
- Scale from MVP to production without infrastructure management
- Provide type-safety end-to-end (TypeScript)
- Support complex queries and relationships
- Minimize operational overhead
- Enable rapid development

Traditional options included:

- PostgreSQL/MySQL with REST API
- MongoDB with REST API
- Firebase Realtime Database/Firestore
- Supabase
- Custom GraphQL server

## Decision

We chose **Convex** as the backend platform for Doctrina LMS.

Convex is a serverless backend platform that provides:

- **Real-time database** with automatic subscriptions
- **End-to-end TypeScript** with generated types
- **Serverless functions** (queries, mutations, actions)
- **Built-in authentication** (integrates with Clerk)
- **ACID transactions** and relational data modeling
- **Zero infrastructure management**

## Rationale

### Why Convex Over Alternatives

#### vs. PostgreSQL + REST API

**Convex Advantages**:

- Real-time updates built-in (no WebSocket setup)
- No ORM needed (native TypeScript)
- Serverless (no database provisioning)
- Automatic type generation

**Trade-offs**:

- PostgreSQL more mature ecosystem
- PostgreSQL has more SQL tooling

**Decision**: Real-time is critical for LMS (progress updates, live sessions). Convex's built-in real-time is worth the trade-off.

#### vs. Firebase Firestore

**Convex Advantages**:

- Better TypeScript support
- Relational data modeling (indexes, joins)
- Transactional consistency (ACID)
- Better query performance for complex filters

**Trade-offs**:

- Firebase has larger ecosystem
- Firebase has more third-party integrations

**Decision**: Convex's relational model and TypeScript DX are better for complex LMS data (courses, modules, lessons, enrollments).

#### vs. Supabase

**Convex Advantages**:

- Better TypeScript DX (generated types)
- Simpler real-time subscriptions (no channels to manage)
- No SQL knowledge required
- Faster development velocity

**Trade-offs**:

- Supabase offers PostgreSQL (more familiar)
- Supabase has built-in auth (though we're using Clerk)

**Decision**: Convex's developer experience and real-time simplicity outweigh Supabase's PostgreSQL familiarity.

---

## Consequences

### Positive

1. **Fast Development**: No backend boilerplate, no API routes to write
2. **Real-Time by Default**: Progress updates, notifications, live sessions work seamlessly
3. **Type-Safe**: TypeScript types generated automatically from schema
4. **Scalable**: Serverless architecture scales automatically
5. **Simple Queries**: Convex queries are JavaScript/TypeScript (no SQL, no ORM)
6. **Testing**: `convex-test` library enables isolated backend testing

### Negative

1. **Vendor Lock-In**: Migrating away from Convex would be significant work
2. **Learning Curve**: Team must learn Convex-specific patterns (queries vs mutations, ctx.db API)
3. **Ecosystem Size**: Smaller ecosystem than PostgreSQL or Firebase
4. **Query Limitations**: No full-text search (must use external service like Algolia)
5. **Cost**: Pricing model may be expensive at large scale (though scalable)

---

## Implementation Patterns

### Query Pattern

```typescript
// convex/courses.ts
import { query } from './_generated/server';

export const list = query({
	handler: async ctx => {
		return await ctx.db.query('courses').collect();
	},
});
```

### Mutation Pattern

```typescript
// convex/courses.ts
import { mutation } from './_generated/server';
import { v } from 'convex/values';

export const create = mutation({
	args: {
		title: v.string(),
		description: v.string(),
	},
	handler: async (ctx, args) => {
		const identity = await ctx.auth.getUserIdentity();
		if (!identity) throw new Error('Unauthorized');

		return await ctx.db.insert('courses', args);
	},
});
```

### Real-Time in Frontend

```typescript
'use client';
import { useQuery, useMutation } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function CourseList() {
  // Real-time subscription - updates automatically
  const courses = useQuery(api.courses.list);

  if (courses === undefined) return <LoadingSkeleton />;

  return <div>{courses.map(/* ... */)}</div>;
}
```

---

## Alternatives Considered

### Alternative 1: PostgreSQL + Prisma + tRPC

**Pros**:

- Battle-tested database (PostgreSQL)
- Strong ORM (Prisma) with type generation
- Type-safe API (tRPC)
- Familiar SQL

**Cons**:

- Complex setup (database provisioning, migrations, API routes)
- Real-time requires WebSocket setup (Socket.io or similar)
- More operational overhead
- Slower development velocity

**Why Not**: Real-time complexity and operational overhead outweigh PostgreSQL's maturity.

### Alternative 2: Firebase Firestore

**Pros**:

- Real-time subscriptions built-in
- Serverless, scalable
- Large ecosystem
- Good documentation

**Cons**:

- NoSQL limitations (no complex queries, no joins)
- Weaker TypeScript support
- Denormalization required for performance
- Expensive at scale

**Why Not**: NoSQL model doesn't fit LMS relational data (courses, modules, lessons, enrollments).

### Alternative 3: Supabase (PostgreSQL + Realtime)

**Pros**:

- PostgreSQL (relational)
- Real-time subscriptions
- Built-in auth
- Familiar SQL

**Cons**:

- Complex real-time setup (channels)
- Requires SQL knowledge
- Less TypeScript-native than Convex

**Why Not**: Convex's TypeScript DX and simpler real-time model are better for rapid development.

---

## Migration Considerations

### If We Need to Migrate Away from Convex

**Likely Reasons**:

- Cost becomes prohibitive at scale
- Feature limitations (e.g., full-text search)
- Company policy requires self-hosted database

**Migration Path**:

1. **Schema Migration**: Convert Convex schema to SQL schema (PostgreSQL)
2. **Data Migration**: Export Convex data, transform, import to PostgreSQL
3. **Backend Migration**: Rewrite queries/mutations as REST API or GraphQL
4. **Frontend Migration**: Replace `useQuery/useMutation` with `fetch` or React Query
5. **Real-Time Migration**: Add WebSocket layer (Socket.io or Pusher)

**Estimated Effort**: 4-6 weeks for full migration

**Risk Mitigation**:

- Keep business logic in service layer (not in Convex functions)
- Use TypeScript types that can be shared across backends
- Document data model thoroughly

---

## References

- [Convex Documentation](https://docs.convex.dev/)
- [Convex vs Firebase Comparison](https://docs.convex.dev/guides/comparing-convex-and-firebase)
- [Convex React Integration](https://docs.convex.dev/client/react)
- [convex-test Library](https://www.npmjs.com/package/convex-test)
- Project File: `docs/ARCHITECTURE.md` (Backend Services Layer section)
- Project File: `docs/CONVEX.md` (Convex-specific patterns)

---

**Last Updated**: 2026-01-15
**Next Review**: After 6 months of production use
