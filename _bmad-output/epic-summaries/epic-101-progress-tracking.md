# EPIC-101: Lesson Progress Tracking System

**Priority**: P0 - CRITICAL
**Effort**: 13 story points
**Status**: ✅ COMPLETE
**Sprint**: Sprint 2 (Weeks 3-4) - COMPLETED

---

## Epic Overview

### Problem Statement

The learning interface (`/courses/[id]/learn`) has "Mark as Complete" buttons but no backend to persist completion. Progress bars are stuck at 0%. Certificates cannot be generated because there's no completion detection.

**User Impact**: Students cannot track their progress, instructors cannot measure engagement, and completion certificates cannot be issued.

### Solution Summary

Implement a complete lesson progress tracking system that:

1. Tracks lesson completions per user
2. Calculates course progress percentages
3. Triggers certificate generation at 100% completion
4. Enables "Continue Learning" functionality

---

## Technical Architecture

### Database Schema

#### lessonProgress Table

```typescript
lessonProgress: defineTable({
	userId: v.string(), // Clerk user ID
	lessonId: v.id('lessons'), // Foreign key to lessons
	completedAt: v.number(), // Unix timestamp (ms)
})
	.index('by_user_lesson', ['userId', 'lessonId']) // Uniqueness + lookup
	.index('by_user', ['userId']) // Get all user progress
	.index('by_lesson', ['lessonId']); // Get lesson completion stats
```

**Design Decisions**:

- **Simple schema**: Only essential fields (userId, lessonId, completedAt)
- **No "started" tracking**: Only track completion, not in-progress state
- **Immutable**: Once completed, record persists (no "uncomplete" in MVP)
- **Compound index**: `by_user_lesson` ensures one completion per user-lesson

### Backend Functions

#### 1. markComplete (Mutation)

**Purpose**: Mark a lesson as complete for the current user

**Flow**:

```
1. Verify user authentication
2. Get lesson details (moduleId, courseId)
3. Verify user is enrolled in course
4. Check if already marked complete (idempotent)
5. Insert lessonProgress record
6. Trigger progress recalculation for enrollment
7. Return progressId
```

**Authorization**:

- Must be authenticated
- Must be enrolled in the course

**Idempotency**: If already complete, returns existing record ID

#### 2. recalculateProgress (Mutation)

**Purpose**: Update enrollment progress percentage and trigger certificate generation

**Flow**:

```
1. Get enrollment record
2. Query all courseModules for this course
3. Collect all lessons across all modules
4. Count total lessons
5. Query lessonProgress records for this user
6. Count completed lessons
7. Calculate progressPercent = (completed / total) * 100
8. Update enrollment.progressPercent
9. If 100%, set enrollment.completedAt
10. If 100%, schedule certificate generation
11. Return { progressPercent, completedAt }
```

**Performance Considerations**:

- Multiple queries (modules → lessons → progress records)
- Potentially expensive for courses with many lessons
- Called on every lesson completion
- **Optimization**: Could denormalize lesson count per module

#### 3. getUserProgress (Query)

**Purpose**: Get user's progress for a course (for display)

**Returns**:

```typescript
{
  enrollmentId: Id<'enrollments'>,
  total: number,                    // Total lessons
  completed: number,                // Completed lessons
  percent: number,                  // enrollment.progressPercent
  completedLessonIds: Id<'lessons'>[]  // Array of completed IDs
}
```

**Use Cases**:

- Display progress bar on learning page
- Show which lessons are complete (checkmarks)
- Dashboard progress display

#### 4. getNextIncompleteLesson (Query)

**Purpose**: Find the next lesson the user hasn't completed

**Flow**:

```
1. Get all modules for course (sorted by order)
2. For each module:
   a. Get lessons (sorted by order)
   b. For each lesson:
      i. Check if user has completed it
      ii. If not, return this lesson ID
3. If all complete, return first lesson (for review)
```

**Use Cases**:

- "Continue Learning" button
- Auto-navigate to next incomplete lesson

---

## Integration Points

### 1. Enrollments Integration

**File**: `convex/enrollments.ts`

**Changes Needed**:

- Call `recalculateProgress()` from `markComplete()`
- Update `progressPercent` field in enrollments table
- Set `completedAt` timestamp at 100%

**Pattern**:

```typescript
await ctx.runMutation(api.lessonProgress.recalculateProgress, {
	enrollmentId: enrollment._id,
});
```

### 2. Certificates Integration

**File**: `convex/certificates.ts`

**Changes Needed** (Story 101.2):

- Certificate generation currently manual
- Should be triggered automatically at 100% completion
- Use `ctx.scheduler.runAfter()` for async generation

**Pattern**:

```typescript
if (progressPercent === 100) {
	await ctx.scheduler.runAfter(0, api.certificates.generate, {
		userId: enrollment.userId,
		courseId: enrollment.courseId,
	});
}
```

### 3. Frontend Integration

**Files**:

- `app/courses/[id]/learn/page.tsx` - Learning interface
- `app/dashboard/page.tsx` - Progress display
- `app/dashboard/progress/page.tsx` - Detailed progress

**Pattern**:

```typescript
'use client';
import { useMutation, useQuery } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function LessonView({ lessonId, courseId }) {
  const markComplete = useMutation(api.lessonProgress.markComplete);
  const progress = useQuery(api.lessonProgress.getUserProgress, { courseId });

  const handleComplete = async () => {
    await markComplete({ lessonId });
    // Progress updates automatically (real-time Convex)
  };

  return (
    <Button onClick={handleComplete} disabled={progress?.completedLessonIds.includes(lessonId)}>
      {progress?.completedLessonIds.includes(lessonId) ? 'Completed ✓' : 'Mark Complete'}
    </Button>
  );
}
```

---

## Stories Breakdown

### Story 101.1: Create lessonProgress Schema & Mutations (8 pts) - ✅ COMPLETE

**File**: `docs/stories/story-101.1.md`
**Status**: ✅ DONE
**Owner**: Dev agent (Amelia)

**Tasks**: ✅ All Complete

1. ✅ Create `convex/lessonProgress.ts`
2. ✅ Implement `markComplete()` mutation
3. ✅ Implement `recalculateProgress()` mutation
4. ✅ Implement `getUserProgress()` query
5. ✅ Implement `getNextIncompleteLesson()` query
6. ✅ Write comprehensive tests (100% coverage)
7. ✅ Update schema.ts if needed

**Acceptance Criteria**: ✅ All Met

- ✅ All functions implemented
- ✅ 100% test coverage
- ✅ Real-time progress updates
- ✅ Idempotent markComplete

### Story 101.2: Refactor Certificate Triggers (5 pts) - ✅ COMPLETE

**File**: `docs/stories/story-101.2.md`
**Status**: ✅ DONE
**Owner**: Dev agent (Amelia)
**Depends On**: Story 101.1 (complete)

**Tasks**: ✅ All Complete

1. ✅ Modify `convex/certificates.ts`
2. ✅ Add automatic certificate generation trigger
3. ✅ Remove manual certificate creation from UI
4. ✅ Test certificate generation at 100% completion
5. ✅ Update tests

**Acceptance Criteria**: ✅ All Met

- ✅ Certificate generated automatically at 100%
- ✅ No manual certificate creation needed
- ✅ 100% test coverage

---

## Testing Strategy

### Unit Tests (Vitest + convex-test)

**File**: `convex/__test__/lessonProgress.test.ts`

**Test Scenarios**:

1. **markComplete()**:
   - ✅ Authenticated user can mark lesson complete
   - ✅ Unauthenticated user cannot mark complete
   - ✅ User not enrolled cannot mark complete
   - ✅ Marking already completed lesson is idempotent
   - ✅ Progress recalculation is triggered

2. **recalculateProgress()**:
   - ✅ Correctly calculates 0% (no lessons complete)
   - ✅ Correctly calculates 50% (half complete)
   - ✅ Correctly calculates 100% (all complete)
   - ✅ Updates enrollment.progressPercent
   - ✅ Sets enrollment.completedAt at 100%
   - ✅ Triggers certificate generation at 100%

3. **getUserProgress()**:
   - ✅ Returns correct total lesson count
   - ✅ Returns correct completed lesson count
   - ✅ Returns completed lesson IDs
   - ✅ Returns null for unauthenticated user
   - ✅ Returns null for user not enrolled

4. **getNextIncompleteLesson()**:
   - ✅ Returns first lesson if none complete
   - ✅ Returns next lesson after last completed
   - ✅ Returns first lesson if all complete
   - ✅ Respects module and lesson order

**Coverage Target**: 100% (mandatory)

### Integration Tests

1. **Full Progress Flow**:
   - Create user → Enroll → Complete all lessons → Verify certificate generated

2. **Real-Time Updates**:
   - Mark lesson complete → Verify progress bar updates in real-time

3. **Edge Cases**:
   - Course with 1 lesson
   - Course with 100+ lessons
   - Multiple users completing same course

---

## Performance Considerations

### Query Optimization

**Concern**: `recalculateProgress()` makes multiple queries

**Current Approach**:

```typescript
// Get modules
const modules = await ctx.db.query('courseModules')...
// For each module, get lessons
for (const module of modules) {
  const lessons = await ctx.db.query('lessons')...
}
```

**Performance**: O(M + L) where M = modules, L = lessons

- Acceptable for MVP (courses have ~10-50 lessons)
- Consider caching for courses with 100+ lessons

**Future Optimization**:

- Denormalize `lessonCount` on course/module
- Cache progress calculations (invalidate on lesson complete)
- Background job for batch recalculation

### Index Usage

**by_user_lesson Index**:

- Used for uniqueness check
- Used for "already complete" queries
- Efficient compound index

**by_user Index**:

- Used for getUserProgress()
- Efficient for "get all user completions"

---

## Security Considerations

### Authorization

- ✅ All mutations verify authentication
- ✅ Enrollment check prevents unauthorized progress marking
- ✅ Queries return null for unauthenticated users

### Data Validation

- ✅ lessonId must be valid (foreign key)
- ✅ courseId verified through lesson → module → course chain
- ✅ userId comes from auth identity (trusted)

### Edge Cases

- **Race condition**: Multiple simultaneous markComplete() calls
  - **Solution**: Index uniqueness prevents duplicates
- **Orphaned progress**: User unenrolls but progress remains
  - **Solution**: Keep progress for historical data
- **Deleted lessons**: Lesson deleted but progress exists
  - **Solution**: Soft-delete lessons, keep progress records

---

## Dependencies

### Upstream Dependencies

- ✅ `lessons` table exists
- ✅ `courseModules` table exists
- ✅ `enrollments` table exists
- ✅ `certificates` table exists

### Downstream Dependencies

- **EPIC-102 (Quizzes)**: May want to block progress until quiz passed
- **EPIC-103 (Reviews)**: May want to require reviews for certificate
- **EPIC-109 (Mock Replacement)**: Progress dashboard needs real data

---

## Rollout Plan

### Phase 1: Backend Implementation (Week 3)

1. Implement `convex/lessonProgress.ts`
2. Write and pass all tests (100% coverage)
3. Deploy to Convex backend

### Phase 2: Frontend Integration (Week 3)

1. Update learning page to call markComplete()
2. Update progress bars to use getUserProgress()
3. Update "Continue Learning" to use getNextIncompleteLesson()
4. Test real-time updates

### Phase 3: Certificate Integration (Week 4)

1. Refactor certificate trigger (Story 101.2)
2. Test end-to-end flow
3. Verify certificates generated at 100%

---

## Success Metrics

### Functional Metrics

- [ ] 100% of lesson completions persisted
- [ ] Progress bars show accurate percentages
- [ ] Certificates generated within 1 second of 100% completion
- [ ] "Continue Learning" navigates correctly

### Performance Metrics

- [ ] markComplete() completes in < 500ms
- [ ] getUserProgress() completes in < 200ms
- [ ] recalculateProgress() completes in < 1000ms (for 50 lesson course)

### Quality Metrics

- [ ] 100% test coverage
- [ ] Zero TypeScript errors
- [ ] Zero ESLint warnings
- [ ] All acceptance criteria met

---

## Known Limitations (MVP)

1. **No "uncomplete"**: Once marked complete, cannot undo (acceptable for MVP)
2. **No partial completion**: Lesson is either 0% or 100% (no tracking of video progress)
3. **No time tracking**: Only track completion, not time spent (future enhancement)
4. **No prerequisites**: Cannot block later lessons until earlier ones complete (future)
5. **No quiz blocking**: Progress tracking independent of quiz results (addressed in EPIC-102)

---

## Future Enhancements (Post-MVP)

1. **Video Progress Tracking**: Track how much of video watched (%)
2. **Time Spent**: Track duration spent on each lesson
3. **Prerequisites**: Block lessons until earlier ones complete
4. **Quiz Prerequisites**: Require quiz pass before marking complete
5. **Batch Recalculation**: Background job for progress repair
6. **Progress Analytics**: Instructor dashboard showing drop-off points
7. **Gamification**: Badges, streaks, milestones

---

## References

### Documentation

- **Epic Definition**: `docs/EPICS.md` (EPIC-101 section)
- **Story 101.1**: `docs/stories/story-101.1.md`
- **Story 101.2**: `docs/stories/story-101.2.md`
- **Story Context**: `docs/stories/story-context-101.1.xml`
- **DB Schema**: `docs/DB-SCHEMA.md` (lessonProgress table)

### Related Files

- `convex/schema.ts` - lessonProgress table definition
- `convex/enrollments.ts` - progressPercent field
- `convex/certificates.ts` - generation trigger
- `app/courses/[id]/learn/page.tsx` - UI integration

---

**Last Updated**: 2026-01-15
**Epic Owner**: Dev team
**Next Review**: After Story 101.1 completion
