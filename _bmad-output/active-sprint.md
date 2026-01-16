# What Needs to Be Done - Current Work Queue

**Last Updated**: 2026-01-15
**Status Source**: See `docs/sprint-status.yaml` for official tracking

---

## Purpose

This document guides BMAD agents on **what work remains** and **what to implement next**. For official status tracking, see `docs/sprint-status.yaml`.

---

## ⚠️ Incomplete Work - Priorities

### Priority 1: Complete EPIC-102 (Quiz System)

**Remaining**: 4 of 5 stories (15 of 25 points)

#### Story 102.2: Quiz Submission & Grading Backend (6 pts)
**Status**: DRAFTED (ready for dev)
**File**: `docs/stories/story-102.2.md`
**What to Build**:
- `submit()` mutation - Grade quiz, enforce maxAttempts
- `getBestAttempt()` query - Get highest score
- `getAttemptResults()` query - Get detailed results
- 100% test coverage

**Why This Matters**: Students can't take quizzes until this exists

#### Story 102.3: Quiz Management Backend (4 pts)
**Status**: BACKLOG
**What to Build**:
- Schema: Add maxAttempts, deleted, deletedAt fields
- `update()` mutation - Update quiz settings
- `remove()` mutation - Soft-delete quiz
- `restore()` mutation - Un-delete quiz

#### Story 102.4a: Quiz Creation UI Enhancements (2 pts)
**Status**: BACKLOG
**What to Build**:
- Add explanation textarea to question form
- Add quiz settings (passingScore, maxAttempts)
- Update QuizQuestion interface

#### Story 102.4b: Quiz Taking/Results UI (3 pts)
**Status**: BACKLOG
**What to Build**:
- Quiz taking interface component
- Results display with explanations
- Attempt tracking ("Attempt X of Y")
- Retake quiz button

---

### Priority 2: EPIC-103 (Course Reviews & Ratings)

**Remaining**: 2 stories (8 points total)

#### Story 103.1: Create Course Reviews Backend (5 pts)
**Status**: DRAFTED
**What to Build**:
- `create()` mutation - Submit review (enrolled users only)
- `list()` query - Get reviews for course
- `hide()` mutation - Admin moderation
- Average rating calculation

#### Story 103.2: Update Course Ratings UI (3 pts)
**Status**: BACKLOG
**What to Build**:
- Update courses.list() to include real ratings
- Update course card display

---

### Priority 3: Remaining Cleanup

#### Story 109.3: Post-MVP Mock Replacement
**Status**: DRAFTED
**What to Build**:
- Replace community page mock data (requires EPIC-106 backend)

---

## Next Story to Implement

**Recommended**: `story-102.2` - Quiz Submission & Grading Backend

**Why**:
- Blocks student quiz-taking functionality
- Story is DRAFTED (ready for dev)
- 6 points, completable in 1-2 days
- High user impact

---

## Epic Summaries for Context

Before implementing, read:
- `epic-summaries/epic-102-quiz-system.md` - Full technical context
- `docs/stories/story-102.2.md` - Story details
- `docs/stories/story-context-102.2.xml` - Implementation context (if exists)

---

## Backlog Overview

### Goal

Complete the core learning experience by enabling students to:
1. Mark lessons as complete
2. Track their progress through courses
3. Take quizzes and receive instant feedback
4. Earn certificates upon 100% completion

### Why This Sprint Matters

Without progress tracking and quizzes, the platform has no way to:
- Measure student engagement
- Determine course completion
- Issue certificates
- Provide meaningful analytics to instructors

These features are **P0 (Critical)** and block the MVP launch.

---

## Active Epics

### EPIC-101: Lesson Progress Tracking System

**Priority**: P0 - CRITICAL
**Effort**: 13 story points
**Status**: NOT STARTED

#### Problem

The `/courses/[id]/learn` page has "Mark as Complete" buttons but no backend to persist completion. Progress bars show 0%. Certificates can't be generated because completion detection doesn't work.

#### What Needs to be Built

1. **Backend Functions** (`convex/lessonProgress.ts`):
   - `markComplete()` - Mark lesson as complete
   - `unmarkComplete()` - Undo completion (if needed)
   - `getUserProgress()` - Get user's progress for a course
   - `getNextIncompleteLesson()` - For "Continue Learning" feature
   - `recalculateProgress()` - Update enrollment progress percentage

2. **Integration Points**:
   - Update `enrollments.progressPercent` when lessons completed
   - Trigger certificate generation at 100% completion
   - Real-time progress updates in UI

#### Stories

1. **Story 101.1**: Create lessonProgress Schema & Mutations (8 pts)
   - File: `docs/stories/story-101.1.md`
   - Status: READY FOR DEV
   - Next Action: Implement `convex/lessonProgress.ts`

2. **Story 101.2**: Refactor Certificate Triggers (5 pts)
   - File: `docs/stories/story-101.2.md`
   - Status: DRAFTED
   - Depends on: Story 101.1

#### Acceptance Criteria

- [ ] Student can mark lesson as complete
- [ ] Progress updates in real-time (optimistic UI)
- [ ] Progress percentage calculated correctly
- [ ] Certificate generated at 100% completion
- [ ] "Continue Learning" navigates to next incomplete lesson
- [ ] Progress persists across sessions
- [ ] 100% test coverage for all functions

#### Files to Create/Modify

- **Create**: `convex/lessonProgress.ts`
- **Create**: `convex/__test__/lessonProgress.test.ts`
- **Modify**: `convex/certificates.ts` (trigger logic)
- **Modify**: `convex/enrollments.ts` (progress update)

---

### EPIC-102: Quiz Submission & Grading System

**Priority**: P0 - CRITICAL
**Effort**: 25 story points (revised from 21)
**Status**: PARTIALLY COMPLETE

#### Problem

The course wizard has an AI quiz generator component, but there's no backend to save quiz questions or grade submissions. Students can't take quizzes or receive feedback.

#### What Needs to be Built

1. **Quiz Creation** (Story 102.1 - COMPLETED ✅):
   - Create quiz with title and passing score
   - Add multiple choice questions with explanations
   - Retrieve quiz for display

2. **Quiz Submission & Grading** (Story 102.2 - BACKLOG):
   - Submit quiz answers
   - Auto-grade submissions
   - Track attempts (with maxAttempts enforcement)
   - Show results with explanations

3. **Quiz Management** (Story 102.3 - BACKLOG):
   - Update quiz details (title, passingScore, maxAttempts)
   - Soft-delete quizzes (preserve student attempts)
   - Restore deleted quizzes

4. **UI Enhancements** (Stories 102.4a, 102.4b - BACKLOG):
   - Quiz taking interface
   - Results display with explanations for ALL questions
   - Attempt tracking UI
   - Retake quiz functionality

#### Stories

1. **Story 102.1**: Create Quiz System (10 pts) - ✅ **COMPLETED**
   - File: `docs/stories/story-102.1.md`
   - Status: DONE
   - Implemented: `convex/quizzes.ts` (create, addQuestions, getQuiz)

2. **Story 102.2**: Quiz Submission & Grading Backend (6 pts)
   - File: `docs/stories/story-102.2.md`
   - Status: DRAFTED (BACKLOG)
   - Next Action: Implement submit(), getBestAttempt(), getAttemptResults()

3. **Story 102.3**: Quiz Management Backend (4 pts)
   - Status: BACKLOG
   - Next Action: Add maxAttempts, soft-delete, update mutations

4. **Story 102.4a**: Quiz Creation UI Enhancements (2 pts)
   - Status: BACKLOG
   - Next Action: Add explanation fields to wizard

5. **Story 102.4b**: Quiz Taking/Results UI (3 pts)
   - Status: BACKLOG
   - Next Action: Build quiz interface in learning page

#### Acceptance Criteria

- [x] Instructor can create quiz from wizard (Story 102.1 complete)
- [x] Instructor can add questions (AI-generated or manual) (Story 102.1 complete)
- [ ] Student can load quiz and see questions
- [ ] Student can submit answers
- [ ] Quiz graded instantly with feedback
- [ ] Attempts tracked and enforced (maxAttempts)
- [ ] Best score tracked
- [ ] Results show explanations for ALL questions

#### Files to Create/Modify

- **Existing**: `convex/quizzes.ts` (extend with submission logic)
- **Create**: `convex/__test__/quizzes.test.ts` (comprehensive tests)
- **Modify**: `components/course-wizard/ai-quiz-generator.tsx` (add explanation field)
- **Create**: `components/quiz/QuizTakingInterface.tsx`
- **Create**: `components/quiz/QuizResults.tsx`

---

## Sprint Backlog (Ordered by Priority)

### Week 3 Focus

1. **Story 101.1**: Create lessonProgress Schema & Mutations (8 pts)
   - **Owner**: Dev agent (Amelia)
   - **Status**: READY FOR DEV
   - **Priority**: P0
   - **Blocking**: Certificate generation, progress tracking

2. **Story 102.2**: Quiz Submission & Grading Backend (6 pts)
   - **Owner**: Dev agent (Amelia)
   - **Status**: DRAFTED
   - **Priority**: P0
   - **Depends on**: Story 102.1 (complete)

### Week 4 Focus

3. **Story 101.2**: Refactor Certificate Triggers (5 pts)
   - **Owner**: Dev agent (Amelia)
   - **Status**: DRAFTED
   - **Priority**: P0
   - **Depends on**: Story 101.1

4. **Story 102.3**: Quiz Management Backend (4 pts)
   - **Owner**: Dev agent (Amelia)
   - **Status**: BACKLOG
   - **Priority**: P1

**Total Sprint Points**: 23 points (achievable in 2 weeks)

---

## Dependencies & Blockers

### External Dependencies

- ✅ Convex backend operational
- ✅ Clerk authentication configured
- ✅ Story 102.1 completed (quiz creation)

### Internal Dependencies

- Story 101.2 depends on Story 101.1 (progress tracking must exist before certificate triggers)
- Stories 102.4a/102.4b depend on Stories 102.2/102.3 (backend must exist before UI)

### Current Blockers

**None** - All stories are unblocked and ready for implementation.

---

## Definition of Done

A story is considered "done" when:

1. ✅ **Code Implemented**: All tasks/subtasks in story file completed
2. ✅ **Tests Written**: 100% test coverage for backend, 85% for frontend
3. ✅ **Tests Passing**: All tests green (yarn test)
4. ✅ **Type-Safe**: No TypeScript errors (yarn typescript)
5. ✅ **Lint Clean**: Zero ESLint warnings (yarn lint)
6. ✅ **Formatted**: Code formatted (yarn formatting)
7. ✅ **Acceptance Criteria Met**: All AC from story satisfied
8. ✅ **Story Context Updated**: Story file marked as done

---

## Sprint Metrics

### Velocity Tracking

- **Sprint 1 Velocity**: ~25 points (Course Structure, Enrollments, Stripe)
- **Sprint 2 Planned**: 23 points
- **Sprint 2 Completed**: 0 points (as of 2026-01-15)
- **Sprint 2 Remaining**: 23 points

### Burn-down

- **Week 3 Target**: Complete 13 points (Stories 101.1, 102.2)
- **Week 4 Target**: Complete 10 points (Stories 101.2, 102.3)

---

## Testing Strategy for Sprint 2

### Backend Testing (100% Coverage Required)

1. **convex/lessonProgress.ts**:
   - Test markComplete() with valid/invalid auth
   - Test progress calculation logic
   - Test certificate trigger at 100%
   - Test edge cases (no lessons, all complete, partial)

2. **convex/quizzes.ts** (extend existing):
   - Test submit() with correct/incorrect answers
   - Test maxAttempts enforcement
   - Test getBestAttempt() logic
   - Test soft-delete behavior

### Integration Testing

- Test full flow: enroll → complete lessons → trigger certificate
- Test full flow: take quiz → get results → retake quiz → max attempts

### Manual Testing Checklist

- [ ] Mark all lessons complete, verify certificate generated
- [ ] Progress bar updates in real-time
- [ ] "Continue Learning" button works correctly
- [ ] Quiz submission works with instant feedback
- [ ] Quiz attempts tracked correctly
- [ ] Max attempts prevents further submissions

---

## Sprint Ceremonies

### Daily Standup (Virtual)

**Format**: Update in sprint chat daily
- What did I complete yesterday?
- What am I working on today?
- Any blockers?

### Sprint Review (End of Week 4)

**Goals**:
- Demo progress tracking functionality
- Demo quiz submission and grading
- Review sprint metrics
- Showcase certificate generation

### Sprint Retrospective (End of Week 4)

**Topics**:
- What went well?
- What could be improved?
- Action items for Sprint 3

---

## Sprint 3 Preview (Weeks 5-6)

### Planned Epics

1. **EPIC-102**: Complete remaining quiz stories (4a, 4b - UI work)
2. **EPIC-103**: Course Reviews & Ratings (8 pts)
3. **EPIC-104**: Instructor Verification (partial - 13 pts)

**Goal**: Reviews, instructor verification, quiz UI polishing

---

## Key Resources

### Story Files

- `docs/stories/story-101.1.md` - Lesson progress schema/mutations
- `docs/stories/story-101.2.md` - Certificate trigger refactor
- `docs/stories/story-102.2.md` - Quiz submission/grading
- `docs/stories/story-context-101.1.xml` - Story 101.1 context
- `docs/stories/story-context-101.2.xml` - Story 101.2 context
- `docs/stories/story-context-102.1.xml` - Story 102.1 context

### Epic Documentation

- `docs/EPICS.md` - Full epic definitions
- `_bmad-output/epic-summaries/epic-101.md` - EPIC-101 tech summary
- `_bmad-output/epic-summaries/epic-102.md` - EPIC-102 tech summary

### Sprint Tracking

- `docs/sprint-status.yaml` - Official sprint status YAML
- `docs/bmm-workflow-status.md` - BMM workflow state
- `_bmad-output/active-sprint.md` - THIS FILE (living document)

---

## Commands for This Sprint

```bash
# Start development
yarn dev

# Run tests for specific files
yarn test convex/__test__/lessonProgress.test.ts
yarn test convex/__test__/quizzes.test.ts

# Check test coverage
yarn test:coverage

# Verify everything before commit
yarn verify

# Type check
yarn typescript

# Lint
yarn lint
```

---

## Notes for Dev Agent (Amelia)

### Implementation Order

1. **Start with Story 101.1** (lessonProgress)
   - Create `convex/lessonProgress.ts`
   - Write tests FIRST (TDD)
   - Implement markComplete, getUserProgress, recalculateProgress
   - Ensure 100% coverage

2. **Then Story 102.2** (quiz submission)
   - Extend `convex/quizzes.ts`
   - Add submit(), getBestAttempt(), getAttemptResults()
   - Test with multiple attempts
   - Ensure 100% coverage

3. **Then Story 101.2** (certificate triggers)
   - Modify `convex/certificates.ts`
   - Integrate with lessonProgress
   - Test certificate generation on 100% completion

### Red-Green-Refactor Cycle

1. **Red**: Write failing test
2. **Green**: Write minimal code to pass test
3. **Refactor**: Clean up while keeping tests green

### Story Context XML

Each story has a companion XML file with:
- File paths to existing code
- Implementation context
- Edge cases to handle
- Test scenarios

**Always read story context XML before implementing!**

---

**Last Updated**: 2026-01-15
**Next Update**: After Story 101.1 completion
