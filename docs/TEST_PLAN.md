# ClassSeat Manager — Test Plan

# 1. Purpose

This document defines the verification strategy for ClassSeat Manager.

Testing must focus not only on whether features work, but whether:

* historical data remains correct
* offline operation works
* combined classes work
* seating algorithms respect constraints
* classroom workflows remain fast
* backup/restore is reliable

---

# 2. Test Levels

Use:

1. Unit tests
2. Integration tests
3. Database/repository tests
4. Algorithm tests
5. UI tests
6. End-to-end tests
7. Manual classroom usability tests

---

# 3. Foundation Tests

## TC-001 Project Build

Expected:

```text
npm run build
```

succeeds.

---

## TC-002 Type Check

No TypeScript errors.

---

## TC-003 Lint

No blocking lint errors.

---

# 4. Database Tests

## TC-004 Student ID

Create two students with the same attendance number in different classes.

Expected:

Both are allowed.

---

## TC-005 Duplicate Student

Create two students with the same attendance number in the same class.

Expected:

Rejected or clearly warned.

---

## TC-006 Student Persistence

Create a student.

Close browser.

Reopen.

Expected:

Student remains.

---

# 5. Year Tests

## TC-007 Multiple Years

Create:

* 2026
* 2027

Expected:

Data remains separated.

---

## TC-008 Year Switching

Switch from 2026 to 2027.

Expected:

2026 data is not modified.

---

# 6. Combined Class Tests

## TC-009 Combined Course

Create:

* 1年1組01
* 1年2組01

Add both to one course.

Expected:

Both are displayed separately.

---

## TC-010 Attendance Number Collision

Two students both have attendance number 01.

Expected:

No data collision.

---

# 7. Lesson Session Tests

## TC-011 Create Lesson

Create a lesson.

Expected:

LessonSession exists.

---

## TC-012 Participants

Add students from multiple classes.

Expected:

All participants appear.

---

# 8. Seating Tests

## TC-013 Seat Count

Create:

* 20 seats
* 30 seats
* 40 seats
* 50 seats

Expected:

All work.

---

## TC-014 Too Few Seats

40 students, 35 seats.

Expected:

Warning/error.

Do not silently remove students.

---

## TC-015 Extra Seats

35 students, 40 seats.

Expected:

Five empty seats allowed.

---

## TC-016 Drag Seat

Move a seat.

Expected:

New position persists.

---

## TC-017 Resize Seat

Resize a seat.

Expected:

New dimensions persist.

---

## TC-018 Student Assignment

Assign student to seat.

Expected:

Student appears correctly.

---

## TC-019 Duplicate Assignment

Attempt to assign one student to two seats.

Expected:

Prevented.

---

# 9. Historical Seating Tests

## TC-020 Seat History

Lesson A:

Student A → Seat 5.

Lesson B:

Student A → Seat 18.

Expected:

Lesson A still shows Seat 5.

---

## TC-021 Current Layout Change

Modify current seating.

Expected:

Past completed lesson is unchanged.

---

# 10. Group Tests

## TC-022 Create Groups

Create 8 groups.

Expected:

All groups exist.

---

## TC-023 Group Colors

Use all 20 color indexes.

Expected:

All are available.

---

## TC-024 Group Membership

Assign students.

Expected:

Member list is correct.

---

## TC-025 Group Size Balance

37 students, 4-person groups.

Expected:

Group sizes differ by at most one where possible.

---

## TC-026 Absent Students

Mark two students absent.

Generate groups.

Expected:

Absent students are excluded.

---

# 11. Group Constraint Tests

## TC-027 Forbidden Pair

A and B cannot be together.

Expected:

No valid result contains both.

---

## TC-028 Preferred Pair

A and B prefer same group.

Expected:

Algorithm favors this where possible.

---

## TC-029 Previous Group Avoidance

Students who were previously together should receive a penalty.

Expected:

New groups generally rotate membership.

---

## TC-030 Contradictory Group Constraints

Same pair:

* together required
* separate required

Expected:

System reports conflict.

---

# 12. Seating Shuffle Tests

## TC-031 Random Shuffle

Same class, same constraints.

Different random seeds.

Expected:

Potentially different valid results.

---

## TC-032 Deterministic Seed

Same input and same seed.

Expected:

Same result.

---

## TC-033 Hard Separation

A and B must not be adjacent.

Expected:

Never adjacent.

---

## TC-034 Front Preference

A prefers front.

Expected:

Front seats receive higher score.

---

## TC-035 Previous Seat Avoidance

A was previously in Seat 5.

Expected:

Seat 5 receives penalty.

---

## TC-036 Previous Neighbor Avoidance

A and B were neighbors.

Expected:

New adjacency receives penalty.

---

## TC-037 Impossible Seating

Create impossible hard constraints.

Expected:

No invalid assignment is silently accepted.

---

# 13. Attendance Tests

## TC-038 Present

Set student to present.

Expected:

Status saved.

---

## TC-039 Absent

Set student to absent.

Expected:

Seat becomes gray and shows "欠席".

---

## TC-040 Late

Set late.

Expected:

"遅刻" displayed.

---

## TC-041 Official Absence

Set official absence.

Expected:

Correct status.

---

## TC-042 Attendance List

Modify status from list.

Expected:

Seating chart updates.

---

## TC-043 Seating Chart Attendance

Modify status from seating chart.

Expected:

Attendance list updates.

---

# 14. Evaluation Tests

## TC-044 Create Evaluation Set

Create evaluation set.

Expected:

Set appears.

---

## TC-045 Create Evaluation Item

Add item.

Expected:

Item appears in lesson.

---

## TC-046 Checkbox Evaluation

Mark student.

Expected:

Record saved.

---

## TC-047 Scale Evaluation

Set ◎○△.

Expected:

Correct value stored.

---

## TC-048 Numeric Evaluation

Enter number.

Expected:

Validation works.

---

## TC-049 Evaluation History

Change current evaluation definition.

Expected:

Historical evaluation remains interpretable.

---

# 15. Student Profile Tests

## TC-050 Student Summary

Open student profile.

Expected:

Attendance summary appears.

---

## TC-051 Evaluation Summary

Expected:

Evaluation summary appears.

---

## TC-052 Lesson History

Expected:

Historical lessons appear chronologically.

---

## TC-053 Teacher Memo

Create memo.

Close/reopen.

Expected:

Memo remains.

---

# 16. Timetable Tests

## TC-054 Timetable Entry

Create Monday 1st period.

Expected:

Entry appears.

---

## TC-055 Today's Lesson

Set date.

Expected:

Correct lessons appear.

---

# 17. Printing Tests

## TC-056 Normal Seating Print

Print normal layout.

Expected:

A4 landscape.

---

## TC-057 Group Seating Print

Print group layout.

Expected:

A4 landscape.

---

## TC-058 Print Content

Expected:

* year
* course
* class
* date
* period
* seating chart

are displayed.

---

# 18. CSV Tests

## TC-059 Student Import

Import valid CSV.

Expected:

All valid students imported.

---

## TC-060 Invalid CSV

Missing required column.

Expected:

Clear error.

No partial import unless explicitly supported.

---

## TC-061 Duplicate CSV

Duplicate attendance number.

Expected:

Detected before import.

---

## TC-062 CSV Export

Export students.

Expected:

Correct rows and columns.

---

## TC-063 Attendance Export

Expected:

Correct historical attendance.

---

## TC-064 Evaluation Export

Expected:

Correct evaluation data.

---

# 19. Backup Tests

## TC-065 Full Backup

Create backup.

Expected:

All entities included.

---

## TC-066 Restore

Delete local data.

Restore backup.

Expected:

All data restored.

---

## TC-067 Backup Version

Attempt incompatible backup.

Expected:

Restore rejected with clear message.

---

## TC-068 Broken Reference

Corrupt a backup reference.

Expected:

Validation catches it before restoration.

---

# 20. Offline Tests

## TC-069 Network Disabled

Disable network.

Open application.

Expected:

Application works.

---

## TC-070 Offline Attendance

Record attendance offline.

Expected:

Saved.

---

## TC-071 Offline Evaluation

Record evaluation offline.

Expected:

Saved.

---

## TC-072 Offline Seating

Modify seating offline.

Expected:

Saved.

---

# 21. Data Safety Tests

## TC-073 Browser Restart

Close browser.

Reopen.

Expected:

Data persists.

---

## TC-074 Historical Integrity

Change current configuration.

Expected:

Past lesson data remains unchanged.

---

## TC-075 Unexpected Reload

Reload during lesson.

Expected:

Previously saved actions remain.

---

# 22. UX Tests

## TC-076 Attendance Speed

Teacher can select student and update attendance in a small number of interactions.

Target:

Two main interactions where practical.

---

## TC-077 Evaluation Speed

Teacher can select student and enter evaluation without leaving lesson screen.

---

## TC-078 Student Profile

Student profile opens without losing lesson context.

---

# 23. Performance Tests

Test with:

* 20 students
* 30 students
* 40 students
* 50 students

Measure:

* lesson loading
* seating rendering
* group generation
* seating shuffle

The UI should remain responsive.

---

# 24. Regression Tests

Every Phase must run:

* typecheck
* lint
* unit tests
* integration tests
* build

No Phase should knowingly break a previous Phase.

---

# 25. Critical End-to-End Scenario

The following scenario must pass before MVP release.

```text
Create 2026年度
 ↓
Create 1年1組
 ↓
Import 40 students
 ↓
Create 情報Ⅰ
 ↓
Create timetable
 ↓
Open today's lesson
 ↓
Create 40-seat layout
 ↓
Assign students
 ↓
Save layout
 ↓
Print A4
 ↓
Mark 2 students absent
 ↓
Create evaluation set
 ↓
Evaluate several students
 ↓
Open student profile
 ↓
Create teacher memo
 ↓
Generate groups
 ↓
Exclude absent students
 ↓
Apply group layout
 ↓
Generate constrained seating shuffle
 ↓
Show student-facing shuffle screen
 ↓
Confirm new seats
 ↓
Complete lesson
 ↓
Export CSV
 ↓
Create backup
 ↓
Delete local test data
 ↓
Restore backup
 ↓
Verify all data
```

Expected:

All data remains correct.

---

# 26. Release Criteria

MVP cannot be considered complete if any of the following remain unresolved:

* data loss
* historical corruption
* duplicate student identity
* combined-class collision
* hard seating constraint violation
* hard grouping constraint violation
* backup restoration failure
* offline failure
* A4 printing failure
* critical TypeScript errors
* critical runtime errors

---

# 27. Manual Classroom Trial

Before actual school deployment, run a simulated classroom trial.

Use fictional student data.

Simulate:

* 40 students
* 1 absent
* 1 late
* normal seating
* group activity
* random grouping
* seating shuffle
* evaluation
* teacher memo
* printing
* lesson completion

Only after this trial should actual school data be considered.
