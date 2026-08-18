# ClassSeat Manager Development Rules

## Project

ClassSeat Manager is an offline-first classroom management
application for high school teachers.

The system manages:

- students
- classes
- courses
- lesson sessions
- seating layouts
- groups
- attendance
- evaluations
- teacher memos
- timetables
- printing
- CSV
- backup/restore

## Source of Truth

The following documents are the source of truth:

- docs/MASTER_SPEC.md
- docs/DATA_MODEL.md
- docs/UI_SPEC.md
- docs/ALGORITHM.md
- docs/USER_FLOW.md
- docs/TEST_PLAN.md
- issues/MVP_ISSUES.md

Always read the relevant documents before implementation.

## Critical Architecture Rules

1. Never use attendance number as a primary key.
2. Never permanently bind a student to a seat.
3. Historical lesson data must remain immutable.
4. Class and course must remain separate concepts.
5. LessonSession is the core historical unit.
6. Offline operation is mandatory.
7. Do not rely on external APIs.
8. Do not use localStorage as the primary database.
9. Seating constraints must be separated from the UI.
10. Grouping algorithms must be separated from the UI.

## Development Rules

Do not implement the entire application at once.

Work phase by phase.

Before implementing a major feature:

1. Inspect the existing code.
2. Read the relevant specification.
3. Identify dependencies.
4. Propose the implementation plan.
5. Implement.
6. Run tests.
7. Run type checking.
8. Run lint.
9. Run build.
10. Summarize changes.

Do not silently change the architecture.

If a requirement conflicts with the specification,
stop and explain the conflict.

## UX Priority

The application is used during live classes.

Minimize clicks and interruptions.

The primary workflow is:

Today's Lesson
→ Lesson Session
→ Seating Chart
→ Student
→ Attendance / Evaluation

## History

Never modify historical lesson records when current settings change.

## Git

Make small, meaningful commits.

Do not create one giant commit for the entire project.
