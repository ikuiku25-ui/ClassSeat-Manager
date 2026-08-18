# ClassSeat Manager — Algorithm Specification

# 1. Purpose

This document defines algorithms for:

* random grouping
* constrained seating shuffle
* seat adjacency
* historical avoidance
* constraint scoring

Algorithms must be independent from UI components.

---

# 2. General Architecture

```text
UI
 ↓
Application Service
 ↓
Algorithm Engine
 ↓
Constraint Evaluator
 ↓
Repository
```

The algorithm engine must not directly manipulate UI state.

---

# 3. Constraint Types

## 3.1 Hard Constraint

Must never be violated.

Examples:

* Student A and Student B must not be adjacent.
* Student A and Student B must be separated by a minimum distance.
* Student A must be assigned to an allowed seat.

If a candidate violates a hard constraint, discard the candidate.

---

## 3.2 Soft Constraint

A preference.

Examples:

* Prefer front seats.
* Avoid previous seat.
* Avoid previous neighbors.
* Avoid previous group members.

Soft constraints contribute to a score.

---

# 4. Constraint Priority

Recommended priority:

```text
1000 = absolute
800  = strong
500  = normal
200  = weak
```

The actual numerical values can be changed, but the relative ordering must be preserved.

---

# 5. Seat Adjacency

Adjacency must be based on the seating layout, not only raw screen coordinates.

The system should support:

* left
* right
* front
* back
* diagonal

as configurable neighbor types.

---

# 6. Distance

Basic Euclidean distance:

```text
distance(A,B) =
sqrt(
  (Ax-Bx)^2 +
  (Ay-By)^2
)
```

However, the algorithm should allow a future layout-aware distance model.

---

# 7. Seating Shuffle Inputs

```text
students
availableSeats
previousAssignment
previousGroups
seatingConstraints
layout
randomSeed
```

---

# 8. Seating Shuffle Output

```text
assignment
score
hardConstraintViolations
softConstraintBreakdown
randomSeed
```

---

# 9. Seating Shuffle Algorithm

## Step 1

Get active lesson participants.

## Step 2

Remove students whose attendance status excludes them from seating.

Default excluded statuses:

* absent
* official_absence
* bereavement

Default included statuses:

* present
* late
* early_leave

---

## Step 3

Get available seats.

If:

```text
students > availableSeats
```

return an error.

Do not silently remove students.

If:

```text
availableSeats > students
```

allow empty seats.

---

## Step 4

Apply fixed seats first.

Students with fixed-seat constraints are assigned before random placement.

---

## Step 5

Generate candidate assignments.

Use seeded randomization.

The algorithm should produce multiple candidates.

---

# 10. Candidate Validation

For each candidate:

1. Check duplicate student assignment.
2. Check duplicate seat assignment.
3. Check hard constraints.
4. Calculate soft constraint score.
5. Record result.

---

# 11. Hard Constraint Evaluation

Example:

```text
if adjacent(A,B) && constraint.type == "separate":
    reject candidate
```

Minimum distance:

```text
if distance(A,B) < constraint.minimumDistance:
    reject candidate
```

---

# 12. Soft Constraint Evaluation

Example:

Previous seat avoidance:

```text
if currentSeat == previousSeat:
    penalty
```

Previous neighbor avoidance:

```text
if currentNeighbor == previousNeighbor:
    penalty
```

Front preference:

```text
if student prefers front:
    higher score for front seats
```

---

# 13. Score Example

```text
score =
  previousSeatScore
+ previousNeighborScore
+ previousGroupScore
+ frontPreferenceScore
+ backPreferenceScore
+ togetherBonus
```

Hard constraints are not scored.

They are pass/fail.

---

# 14. Candidate Search

MVP:

```text
generate N candidates
evaluate candidates
select highest scoring valid candidate
```

N should be configurable.

Example:

```text
N = 500
```

The exact value should be benchmarked.

Do not assume 500 is optimal without testing.

---

# 15. Local Improvement

Optional improvement phase:

```text
candidate
 ↓
swap Student A / Student B
 ↓
evaluate
 ↓
score improved?
 ├─ yes → keep
 └─ no → revert
```

Repeat until:

* no improvement
* maximum iterations reached

---

# 16. Impossible Constraints

If no candidate satisfies all hard constraints:

Do not automatically produce an invalid result.

Return:

```text
success = false

violatedConstraints = [...]
```

UI should explain the issue.

---

# 17. Grouping Inputs

```text
students
groupCount
groupSize
previousGroups
groupingConstraints
randomSeed
```

---

# 18. Grouping Participant Filtering

Default excluded:

* absent
* official_absence
* bereavement

Included:

* present
* late
* early_leave

---

# 19. Group Count Calculation

If group size is specified:

```text
groupCount = ceil(studentCount / groupSize)
```

Then distribute students as evenly as possible.

Example:

37 students / 4:

```text
4 × 8
5 × 1
```

Avoid unnecessary group size differences.

---

# 20. Random Group Generation

Initial algorithm:

1. Shuffle students using seeded RNG.
2. Create groups.
3. Distribute students evenly.
4. Evaluate constraints.
5. Repeat.
6. Select best valid result.

---

# 21. Previous Group Penalty

If two students were in the same previous group:

```text
penalty += previousGroupPenalty
```

If they have been together multiple times:

```text
penalty increases
```

This should encourage rotation over time.

---

# 22. Forbidden Pair

Hard constraint.

Example:

```text
A and B cannot be in the same group.
```

Any candidate containing both is rejected.

---

# 23. Preferred Pair

Can be:

* Hard constraint
* Soft constraint

depending on configuration.

Default should be Soft.

---

# 24. Group Size Balance

Group sizes should differ by at most one where possible.

Example:

```text
33 students
8 groups

5 groups × 4
3 groups × 5
```

is preferred over uneven distribution.

---

# 25. Random Seed

Every generation should have a random seed.

Store the seed with the result where practical.

This allows reproducibility.

---

# 26. Group Result

Result:

```text
groups = [
  {
    name: "A班",
    colorIndex: 0,
    students: [...]
  },
  ...
]
```

---

# 27. Group-to-Seating Integration

After group generation:

```text
Groups
 ↓
Group Layout
 ↓
Seats
 ↓
Student Assignment
```

The algorithm must not directly manipulate DOM elements.

---

# 28. Algorithm Logging

For debugging, the engine should be able to return:

* random seed
* number of candidates
* valid candidate count
* best score
* hard constraint status
* soft constraint breakdown

Production UI does not need to display all details.

---

# 29. Performance

The algorithm must remain responsive for approximately 20–50 students.

If computation becomes expensive:

* limit candidate count
* use efficient constraint evaluation
* avoid unnecessary UI re-renders
* move heavy calculations to a Web Worker if necessary

Do not prematurely introduce Web Workers unless profiling justifies it.

---

# 30. Determinism

Given:

```text
same input
+
same constraints
+
same random seed
```

the algorithm should produce the same result.

This is important for testing and debugging.

---

# 31. Testing Requirements

Test:

* 20 students
* 30 students
* 40 students
* 50 students
* fewer seats than students
* more seats than students
* duplicate constraints
* contradictory constraints
* fixed seats
* previous-seat avoidance
* previous-group avoidance
* forbidden pairs
* preferred pairs
* absent students
* combined classes

---

# 32. Future Extensions

Potential future algorithms:

* gender balancing
* achievement balancing
* behavior-aware grouping
* multiple teacher constraints
* optimization using simulated annealing
* constraint programming
* advanced seating fairness

These are not MVP requirements.

Do not implement them unless explicitly requested.
