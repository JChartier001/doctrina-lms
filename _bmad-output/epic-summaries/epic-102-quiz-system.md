# EPIC-102: Quiz Submission & Grading System

**Priority**: P0 - CRITICAL
**Effort**: 25 story points (revised from 21)
**Status**: 🟡 PARTIALLY COMPLETE (1 of 5 stories done - 20%)
**Sprint**: Sprint 2-3 (Weeks 3-6) - IN PROGRESS

---

## Epic Overview

### Problem Statement

The course wizard includes an AI quiz generator component (`components/course-wizard/ai-quiz-generator.tsx`), but there's no backend to save quiz questions or grade submissions. Students cannot take quizzes or receive instant feedback on their performance.

**User Impact**: Instructors cannot assess student learning, students have no way to validate knowledge, and course completion lacks meaningful assessment.

### Solution Summary

Implement a complete quiz system that:

1. Allows instructors to create quizzes with multiple-choice questions
2. Enables students to take quizzes and submit answers
3. Auto-grades submissions and provides instant feedback with explanations
4. Tracks attempts and enforces attempt limits
5. Records best scores for certification/analytics

---

## Technical Architecture

### Database Schema

#### quizzes Table

```typescript
quizzes: defineTable({
	courseId: v.id('courses'),
	moduleId: v.optional(v.id('courseModules')), // Optional: quiz at module level
	title: v.string(),
	passingScore: v.number(), // Default: 80%
	maxAttempts: v.number(), // Default: 3 (Story 102.3)
	deleted: v.boolean(), // Soft delete (Story 102.3)
	deletedAt: v.optional(v.number()), // Soft delete timestamp
	createdAt: v.number(),
})
	.index('by_course', ['courseId'])
	.index('by_module', ['moduleId']);
```

**Design Decisions**:

- **moduleId optional**: Quiz can belong to module OR course
- **Soft delete**: Preserve quiz data when deleted (student attempts remain accessible)
- **maxAttempts**: Limit retakes (prevents answer fishing)
- **passingScore**: Customizable per quiz (default 80%)

#### quizQuestions Table

```typescript
quizQuestions: defineTable({
	quizId: v.id('quizzes'),
	question: v.string(),
	options: v.array(v.string()), // e.g., ["Option A", "Option B", ...]
	correctAnswer: v.number(), // Index of correct option (0-based)
	explanation: v.optional(v.string()), // Why this is correct (Story 102.3)
	order: v.number(), // Display order
}).index('by_quiz', ['quizId']);
```

**Design Decisions**:

- **Multiple choice only**: MVP supports only multiple choice (no essay, no fill-in-blank)
- **options array**: Flexible number of choices (2-6 typical)
- **correctAnswer as index**: Simple integer validation
- **explanation**: Optional feedback for learning (Story 102.4a adds UI)

#### quizAttempts Table

```typescript
quizAttempts: defineTable({
	userId: v.string(), // Clerk user ID
	quizId: v.id('quizzes'),
	answers: v.array(v.number()), // Array of selected option indices
	score: v.number(), // Percentage (0-100)
	passed: v.boolean(), // score >= passingScore
	submittedAt: v.number(), // Unix timestamp (ms)
})
	.index('by_user_quiz', ['userId', 'quizId']) // Get user attempts for quiz
	.index('by_user', ['userId']); // Get all user attempts
```

**Design Decisions**:

- **Store answers array**: Enables review of incorrect answers
- **Passed boolean**: Denormalized for quick filtering
- **No "in-progress"**: Quiz must be submitted atomically (no partial saves in MVP)

---

## Backend Functions

### Story 102.1: Quiz Creation (COMPLETED ✅)

#### 1. create (Mutation)

**Purpose**: Create a new quiz for a course/module

**Authorization**: Instructor must own the course

**Pattern**:

```typescript
export const create = mutation({
	args: {
		courseId: v.id('courses'),
		moduleId: v.optional(v.id('courseModules')),
		title: v.string(),
		passingScore: v.number(), // Default: 80
	},
	handler: async (ctx, args) => {
		// Verify instructor owns course
		const identity = await ctx.auth.getUserIdentity();
		const course = await ctx.db.get(args.courseId);

		if (course.instructorId !== identity?.subject) {
			throw new Error('Not authorized');
		}

		return await ctx.db.insert('quizzes', {
			...args,
			createdAt: Date.now(),
		});
	},
});
```

#### 2. addQuestions (Mutation)

**Purpose**: Add multiple questions to a quiz (bulk operation)

**Authorization**: Instructor must own the course

**Pattern**:

```typescript
export const addQuestions = mutation({
	args: {
		quizId: v.id('quizzes'),
		questions: v.array(
			v.object({
				question: v.string(),
				options: v.array(v.string()),
				correctAnswer: v.number(),
				explanation: v.optional(v.string()),
			}),
		),
	},
	handler: async (ctx, { quizId, questions }) => {
		// Verify ownership
		const quiz = await ctx.db.get(quizId);
		const course = await ctx.db.get(quiz.courseId);
		const identity = await ctx.auth.getUserIdentity();

		if (course.instructorId !== identity?.subject) {
			throw new Error('Not authorized');
		}

		// Insert all questions
		const questionIds = [];
		for (let i = 0; i < questions.length; i++) {
			const id = await ctx.db.insert('quizQuestions', {
				quizId,
				...questions[i],
				order: i,
			});
			questionIds.push(id);
		}

		return questionIds;
	},
});
```

#### 3. getQuiz (Query)

**Purpose**: Retrieve quiz with questions for display

**Security**: Does NOT return correctAnswer to students (only to instructors)

**Pattern**:

```typescript
export const getQuiz = query({
	args: { quizId: v.id('quizzes') },
	handler: async (ctx, { quizId }) => {
		const quiz = await ctx.db.get(quizId);
		if (!quiz) return null;

		const questions = await ctx.db
			.query('quizQuestions')
			.withIndex('by_quiz', q => q.eq('quizId', quizId))
			.collect();

		return {
			...quiz,
			questions: questions
				.sort((a, b) => a.order - b.order)
				.map(q => ({
					id: q._id,
					question: q.question,
					options: q.options,
					// correctAnswer NOT included (security)
				})),
		};
	},
});
```

---

### Story 102.2: Quiz Submission & Grading (BACKLOG)

#### 4. submit (Mutation)

**Purpose**: Submit quiz answers and receive instant grading

**Flow**:

```
1. Verify user authentication
2. Get quiz and questions
3. Check maxAttempts enforcement
4. Grade each answer (compare with correctAnswer)
5. Calculate score percentage
6. Determine pass/fail (score >= passingScore)
7. Insert quizAttempt record
8. Return results with explanations for ALL questions
```

**Pattern**:

```typescript
export const submit = mutation({
	args: {
		quizId: v.id('quizzes'),
		answers: v.array(v.number()), // Array of selected option indices
	},
	handler: async (ctx, { quizId, answers }) => {
		const identity = await ctx.auth.getUserIdentity();
		if (!identity) throw new Error('Unauthorized');

		const quiz = await ctx.db.get(quizId);

		// Check maxAttempts (if field exists)
		const attempts = await ctx.db
			.query('quizAttempts')
			.withIndex('by_user_quiz', q => q.eq('userId', identity.subject).eq('quizId', quizId))
			.collect();

		if (quiz.maxAttempts && attempts.length >= quiz.maxAttempts) {
			throw new Error(`Maximum attempts (${quiz.maxAttempts}) reached`);
		}

		const questions = await ctx.db
			.query('quizQuestions')
			.withIndex('by_quiz', q => q.eq('quizId', quizId))
			.collect();

		const sortedQuestions = questions.sort((a, b) => a.order - b.order);

		// Grade quiz
		let correctCount = 0;
		const results = sortedQuestions.map((q, i) => {
			const isCorrect = q.correctAnswer === answers[i];
			if (isCorrect) correctCount++;

			return {
				questionId: q._id,
				question: q.question,
				options: q.options,
				selectedAnswer: answers[i],
				correctAnswer: q.correctAnswer,
				isCorrect,
				explanation: q.explanation,
			};
		});

		const score = Math.round((correctCount / questions.length) * 100);
		const passed = score >= quiz.passingScore;

		// Save attempt
		const attemptId = await ctx.db.insert('quizAttempts', {
			userId: identity.subject,
			quizId,
			answers,
			score,
			passed,
			submittedAt: Date.now(),
		});

		return {
			attemptId,
			score,
			passed,
			results, // Includes explanations for ALL questions
			passingScore: quiz.passingScore,
			attemptsUsed: attempts.length + 1,
			attemptsRemaining: quiz.maxAttempts ? quiz.maxAttempts - attempts.length - 1 : null,
		};
	},
});
```

#### 5. getBestAttempt (Query)

**Purpose**: Get user's best quiz attempt (highest score)

**Use Cases**:

- Display best score on quiz card
- Show "View Best Attempt" link
- Analytics

**Pattern**:

```typescript
export const getBestAttempt = query({
	args: { quizId: v.id('quizzes') },
	handler: async (ctx, { quizId }) => {
		const identity = await ctx.auth.getUserIdentity();
		if (!identity) return null;

		const attempts = await ctx.db
			.query('quizAttempts')
			.withIndex('by_user_quiz', q => q.eq('userId', identity.subject).eq('quizId', quizId))
			.collect();

		if (attempts.length === 0) return null;

		// Return attempt with highest score
		return attempts.reduce((best, current) => (current.score > best.score ? current : best));
	},
});
```

#### 6. getAttemptResults (Query)

**Purpose**: Retrieve detailed results for a specific attempt

**Pattern**:

```typescript
export const getAttemptResults = query({
	args: { attemptId: v.id('quizAttempts') },
	handler: async (ctx, { attemptId }) => {
		const attempt = await ctx.db.get(attemptId);
		if (!attempt) return null;

		const identity = await ctx.auth.getUserIdentity();

		// Verify user owns this attempt OR is instructor
		if (attempt.userId !== identity?.subject) {
			const quiz = await ctx.db.get(attempt.quizId);
			const course = await ctx.db.get(quiz.courseId);

			if (course.instructorId !== identity?.subject) {
				throw new Error('Not authorized');
			}
		}

		const quiz = await ctx.db.get(attempt.quizId);
		const questions = await ctx.db
			.query('quizQuestions')
			.withIndex('by_quiz', q => q.eq('quizId', attempt.quizId))
			.collect();

		const sortedQuestions = questions.sort((a, b) => a.order - b.order);

		const results = sortedQuestions.map((q, i) => ({
			question: q.question,
			options: q.options,
			selectedAnswer: attempt.answers[i],
			correctAnswer: q.correctAnswer,
			isCorrect: q.correctAnswer === attempt.answers[i],
			explanation: q.explanation,
		}));

		return {
			score: attempt.score,
			passed: attempt.passed,
			submittedAt: attempt.submittedAt,
			passingScore: quiz.passingScore,
			results,
		};
	},
});
```

---

### Story 102.3: Quiz Management (BACKLOG)

#### 7. update (Mutation)

**Purpose**: Update quiz details (title, passingScore, maxAttempts)

**Authorization**: Instructor must own course

#### 8. remove (Mutation - Soft Delete)

**Purpose**: Soft-delete quiz (set deleted=true, preserve attempts)

**Why Soft Delete**: Student attempts must remain accessible even after quiz deleted

#### 9. restore (Mutation)

**Purpose**: Un-delete a soft-deleted quiz

---

## Integration Points

### 1. Course Wizard Integration

**File**: `components/course-wizard/ai-quiz-generator.tsx`

**Changes Needed** (Story 102.4a):

- Add `explanation` textarea field to question form
- Add quiz settings (passingScore slider, maxAttempts input)
- Update QuizQuestion interface to include `explanation?: string`

**Current Flow**:

```
1. User clicks "Generate Quiz" in wizard
2. AI generates questions (title, options, correctAnswer)
3. User reviews/edits questions
4. Wizard calls api.quizzes.create() then api.quizzes.addQuestions()
```

**Enhanced Flow** (Story 102.4a):

```
1. User sets quiz settings (passingScore, maxAttempts)
2. AI generates questions WITH explanations
3. User edits questions/explanations
4. Wizard saves with enhanced data
```

### 2. Learning Page Integration

**File**: `app/courses/[id]/learn/page.tsx`

**Changes Needed** (Story 102.4b):

- Build quiz taking interface component
- Build results display component
- Handle attempt tracking UI
- Show "Retake Quiz" / "Max Attempts Reached" states

**Pattern**:

```typescript
'use client';
import { useQuery, useMutation } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function QuizInterface({ quizId }) {
  const quiz = useQuery(api.quizzes.getQuiz, { quizId });
  const submit = useMutation(api.quizzes.submit);
  const bestAttempt = useQuery(api.quizzes.getBestAttempt, { quizId });

  const [answers, setAnswers] = useState<number[]>([]);
  const [results, setResults] = useState(null);

  const handleSubmit = async () => {
    const result = await submit({ quizId, answers });
    setResults(result);
  };

  if (results) {
    return <QuizResults {...results} />;
  }

  return (
    <div>
      <QuizProgress
        attemptsUsed={bestAttempt ? 1 : 0}
        maxAttempts={quiz.maxAttempts}
      />

      {quiz.questions.map((q, i) => (
        <QuizQuestion
          key={q.id}
          question={q}
          selectedAnswer={answers[i]}
          onAnswerSelect={(answer) => {
            const newAnswers = [...answers];
            newAnswers[i] = answer;
            setAnswers(newAnswers);
          }}
        />
      ))}

      <Button onClick={handleSubmit}>Submit Quiz</Button>
    </div>
  );
}
```

---

## Stories Breakdown

### Story 102.1: Create Quiz System (10 pts) - ✅ COMPLETED

**Status**: DONE
**Implemented**: `convex/quizzes.ts` (create, addQuestions, getQuiz)
**Tests**: Comprehensive test coverage added

### Story 102.2: Quiz Submission & Grading Backend (6 pts)

**Status**: DRAFTED (BACKLOG)
**Files to Create/Modify**:

- Extend `convex/quizzes.ts`
- Add `submit()`, `getBestAttempt()`, `getAttemptResults()`
- Write comprehensive tests

**Acceptance Criteria**:

- [ ] submit() grades quiz correctly
- [ ] maxAttempts enforced (if present)
- [ ] Results include explanations for ALL questions
- [ ] Best attempt tracked
- [ ] 100% test coverage

### Story 102.3: Quiz Management Backend (4 pts)

**Status**: BACKLOG
**Files to Modify**:

- `convex/schema.ts` (add maxAttempts, deleted, deletedAt fields)
- `convex/quizzes.ts` (add update, remove, restore mutations)

**Acceptance Criteria**:

- [ ] update() modifies quiz details
- [ ] remove() soft-deletes quiz
- [ ] restore() un-deletes quiz
- [ ] Deleted quizzes excluded from queries by default
- [ ] Student attempts work with soft-deleted quizzes
- [ ] 100% test coverage

### Story 102.4a: Quiz Creation UI Enhancements (2 pts)

**Status**: BACKLOG
**Files to Modify**:

- `components/course-wizard/ai-quiz-generator.tsx`
- QuizQuestion interface

**Acceptance Criteria**:

- [ ] Explanation textarea added to question form
- [ ] Quiz settings form (passingScore, maxAttempts)
- [ ] AI generates explanations
- [ ] 85% test coverage

### Story 102.4b: Quiz Taking/Results UI (3 pts)

**Status**: BACKLOG
**Files to Create**:

- `components/quiz/QuizTakingInterface.tsx`
- `components/quiz/QuizResults.tsx`
- `components/quiz/QuizProgress.tsx`

**Acceptance Criteria**:

- [ ] Quiz taking interface complete
- [ ] Results show score, pass/fail, explanations for ALL questions
- [ ] Attempt tracking UI ("Attempt X of Y")
- [ ] "Retake Quiz" button (disabled at max attempts)
- [ ] "View Best Attempt" link (if multiple attempts)
- [ ] 85% test coverage

---

## Testing Strategy

### Unit Tests (Vitest + convex-test)

**File**: `convex/__test__/quizzes.test.ts`

**Test Scenarios (Story 102.1 - COMPLETE)**:

- ✅ create() - authorized instructor
- ✅ create() - unauthorized user fails
- ✅ addQuestions() - bulk insert
- ✅ getQuiz() - returns quiz without correctAnswer for students

**Test Scenarios (Story 102.2 - TODO)**:

- [ ] submit() - correct answers get 100%
- [ ] submit() - incorrect answers get 0%
- [ ] submit() - mixed answers calculate correctly
- [ ] submit() - enforces maxAttempts
- [ ] submit() - unauthenticated user fails
- [ ] getBestAttempt() - returns highest score
- [ ] getAttemptResults() - returns full results
- [ ] getAttemptResults() - authorization check

**Test Scenarios (Story 102.3 - TODO)**:

- [ ] update() - modifies quiz details
- [ ] remove() - soft-deletes quiz
- [ ] restore() - un-deletes quiz
- [ ] Queries exclude deleted quizzes
- [ ] Student attempts work with deleted quizzes

**Coverage Target**: 100% (mandatory)

---

## Security Considerations

### Authorization

- ✅ Only instructors can create/modify quizzes
- ✅ Only enrolled students can take quizzes
- ✅ Only student OR instructor can view attempt results
- ✅ correctAnswer hidden from students in getQuiz()

### Validation

- ✅ answers array length must match questions length
- ✅ each answer must be valid index (0 to options.length - 1)
- ✅ maxAttempts enforced server-side (not client-side)

### Data Integrity

- **Soft delete**: Preserves data when quiz deleted
- **Immutable attempts**: Cannot modify submitted attempts
- **Atomic submission**: All answers submitted together (no partial saves)

---

## Performance Considerations

### Grading Performance

**Concern**: Grading happens synchronously in submit()

**Current Approach**: O(N) where N = number of questions

- Acceptable for MVP (quizzes have ~10-20 questions)
- Sub-100ms for typical quiz

**Future Optimization**:

- Cache quiz questions (invalidate on update)
- Batch grade multiple quizzes
- Background job for complex grading (essay questions)

### Query Optimization

**by_quiz Index**: Efficient for getQuiz()
**by_user_quiz Index**: Efficient for getBestAttempt(), maxAttempts check

---

## Dependencies

### Upstream Dependencies

- ✅ `courses` table exists
- ✅ `courseModules` table exists
- ✅ `enrollments` table exists

### Downstream Dependencies

- **EPIC-101 (Progress)**: May want to require quiz pass before marking lesson complete
- **EPIC-103 (Reviews)**: Quiz performance may affect review prompts
- **EPIC-107 (Analytics)**: Quiz scores feed instructor analytics

---

## Rollout Plan

### Phase 1: Backend Core (Week 3) - Story 102.2

1. Implement submit(), getBestAttempt(), getAttemptResults()
2. Write comprehensive tests
3. Deploy to Convex

### Phase 2: Backend Management (Week 4) - Story 102.3

1. Add maxAttempts, soft-delete fields to schema
2. Implement update(), remove(), restore()
3. Write tests

### Phase 3: UI Enhancements (Week 5) - Stories 102.4a, 102.4b

1. Update quiz wizard with explanation fields
2. Build quiz taking interface
3. Build results display component

---

## Success Metrics

### Functional Metrics

- [ ] 100% of quiz submissions graded correctly
- [ ] Instant feedback (< 1 second)
- [ ] Attempt limits enforced
- [ ] Explanations shown for ALL questions

### Quality Metrics

- [ ] 100% backend test coverage
- [ ] 85% frontend test coverage
- [ ] Zero TypeScript errors
- [ ] Zero ESLint warnings

---

## Known Limitations (MVP)

1. **Multiple choice only**: No essay, fill-in-blank, or matching questions
2. **No question banks**: Cannot randomize questions from a pool
3. **No time limits**: Quizzes are not timed
4. **No partial credit**: Questions are right or wrong (no partial points)
5. **No adaptive quizzes**: Difficulty does not adjust based on performance

---

## Future Enhancements (Post-MVP)

1. **Question Types**: Essay, fill-in-blank, matching, true/false
2. **Question Banks**: Randomize questions from pool
3. **Time Limits**: Timed quizzes with countdown
4. **Partial Credit**: Award points for partially correct answers
5. **Adaptive Quizzes**: Adjust difficulty based on performance
6. **Quiz Analytics**: Track difficult questions, average scores
7. **Certification Quizzes**: Proctored quizzes for CE credits

---

## References

### Documentation

- **Epic Definition**: `docs/EPICS.md` (EPIC-102 section)
- **Stories**: `docs/stories/story-102.1.md`, `story-102.2.md`
- **DB Schema**: `docs/DB-SCHEMA.md` (quizzes tables)

### Related Files

- `convex/quizzes.ts` - Backend functions
- `convex/__test__/quizzes.test.ts` - Tests
- `components/course-wizard/ai-quiz-generator.tsx` - Quiz creation UI
- `app/courses/[id]/learn/page.tsx` - Quiz taking UI

---

**Last Updated**: 2026-01-15
**Epic Owner**: Dev team
**Next Review**: After Story 102.2 completion
