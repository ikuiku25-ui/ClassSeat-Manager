# ClassSeat Manager — Data Model

## 1. Data Modeling Principles

### Principle 1

出席番号を主キーにしない。

すべての主要エンティティに内部IDを持たせる。

### Principle 2

現在の状態と過去の履歴を分離する。

### Principle 3

SeatとStudentを固定的に結びつけない。

### Principle 4

ClassとCourseを分離する。

### Principle 5

LessonSessionを授業履歴の中心とする。

### Principle 6

バックアップ形式にはschemaVersionを持たせる。

---

# 2. Entity Relationship

```text
SchoolYear
    │
    ├── Class
    │     │
    │     └── Student
    │
    ├── Course
    │     │
    │     └── CourseEnrollment
    │              │
    │              └── Student
    │
    ├── TimetableEntry
    │
    └── LessonSession
           │
           ├── LessonParticipant
           │       └── Student
           │
           ├── SeatingLayout
           │       │
           │       ├── Seat
           │       │      └── SeatAssignment
           │       │
           │       └── LayoutObject
           │
           ├── Group
           │       └── GroupMember
           │
           ├── AttendanceRecord
           │
           ├── LessonEvaluationSet
           │
           └── TeacherMemo
```

---

# 3. school_years

| Field      | Type     | Description  |
| ---------- | -------- | ------------ |
| id         | UUID     | Primary key  |
| year       | integer  | 2026         |
| name       | string   | 2026年度       |
| is_active  | boolean  | Current year |
| created_at | datetime | Created      |
| updated_at | datetime | Updated      |

Constraints:

* year unique
* active year normally one

---

# 4. classes

| Field          | Type     | Description  |
| -------------- | -------- | ------------ |
| id             | UUID     | Primary key  |
| school_year_id | UUID     | School year  |
| grade          | integer  | Grade        |
| name           | string   | Class name   |
| display_name   | string   | Display name |
| created_at     | datetime | Created      |
| updated_at     | datetime | Updated      |

Unique constraint:

```text
school_year_id + grade + name
```

---

# 5. students

| Field             | Type     | Description       |
| ----------------- | -------- | ----------------- |
| id                | UUID     | Primary key       |
| class_id          | UUID     | Current class     |
| attendance_number | integer  | Attendance number |
| name              | string   | Student name      |
| is_active         | boolean  | Active            |
| created_at        | datetime | Created           |
| updated_at        | datetime | Updated           |

Unique constraint:

```text
class_id + attendance_number
```

Important:

attendance_number is not the primary key.

---

# 6. courses

| Field          | Type     | Description |
| -------------- | -------- | ----------- |
| id             | UUID     | Primary key |
| school_year_id | UUID     | School year |
| name           | string   | Course name |
| description    | string   | Optional    |
| created_at     | datetime | Created     |
| updated_at     | datetime | Updated     |

---

# 7. course_enrollments

Connects students and courses.

| Field      | Type     | Description |
| ---------- | -------- | ----------- |
| id         | UUID     | Primary key |
| course_id  | UUID     | Course      |
| student_id | UUID     | Student     |
| created_at | datetime | Created     |
| ended_at   | datetime | Optional    |

Unique constraint:

```text
course_id + student_id
```

---

# 8. timetable_entries

| Field       | Type     | Description   |
| ----------- | -------- | ------------- |
| id          | UUID     | Primary key   |
| course_id   | UUID     | Course        |
| day_of_week | integer  | 0-6           |
| period      | integer  | Period number |
| is_active   | boolean  | Active        |
| created_at  | datetime | Created       |
| updated_at  | datetime | Updated       |

---

# 9. lesson_sessions

The most important historical entity.

| Field      | Type     | Description                        |
| ---------- | -------- | ---------------------------------- |
| id         | UUID     | Primary key                        |
| course_id  | UUID     | Course                             |
| date       | date     | Lesson date                        |
| period     | integer  | Period                             |
| status     | enum     | planned/active/completed/cancelled |
| created_at | datetime | Created                            |
| updated_at | datetime | Updated                            |

Recommended uniqueness:

```text
course_id + date + period
```

---

# 10. lesson_participants

| Field             | Type | Description |
| ----------------- | ---- | ----------- |
| id                | UUID | Primary key |
| lesson_session_id | UUID | Lesson      |
| student_id        | UUID | Student     |

Unique constraint:

```text
lesson_session_id + student_id
```

This table is essential for combined classes.

---

# 11. seating_layouts

| Field             | Type     | Description  |
| ----------------- | -------- | ------------ |
| id                | UUID     | Primary key  |
| lesson_session_id | UUID     | Lesson       |
| layout_type       | enum     | normal/group |
| name              | string   | Layout name  |
| created_at        | datetime | Created      |
| updated_at        | datetime | Updated      |

---

# 12. seats

| Field       | Type     | Description  |
| ----------- | -------- | ------------ |
| id          | UUID     | Primary key  |
| layout_id   | UUID     | Layout       |
| seat_number | integer  | Seat number  |
| x           | number   | X coordinate |
| y           | number   | Y coordinate |
| width       | number   | Width        |
| height      | number   | Height       |
| rotation    | number   | Rotation     |
| created_at  | datetime | Created      |
| updated_at  | datetime | Updated      |

---

# 13. seat_assignments

Represents who occupies a seat for a particular layout.

| Field       | Type     | Description     |
| ----------- | -------- | --------------- |
| id          | UUID     | Primary key     |
| seat_id     | UUID     | Seat            |
| student_id  | UUID     | Student         |
| assigned_at | datetime | Assignment time |

Constraints:

A student cannot be assigned to multiple seats in the same layout.

---

# 14. layout_objects

| Field       | Type   | Description   |
| ----------- | ------ | ------------- |
| id          | UUID   | Primary key   |
| layout_id   | UUID   | Layout        |
| object_type | enum   | Object type   |
| x           | number | X             |
| y           | number | Y             |
| width       | number | Width         |
| height      | number | Height        |
| rotation    | number | Rotation      |
| text        | string | Optional text |

Supported object types:

* blackboard
* whiteboard
* teacher_desk
* teacher_pc
* text

---

# 15. groups

| Field             | Type     | Description |
| ----------------- | -------- | ----------- |
| id                | UUID     | Primary key |
| lesson_session_id | UUID     | Lesson      |
| name              | string   | Group name  |
| color_index       | integer  | 0-19        |
| created_at        | datetime | Created     |
| updated_at        | datetime | Updated     |

---

# 16. group_members

| Field      | Type | Description |
| ---------- | ---- | ----------- |
| id         | UUID | Primary key |
| group_id   | UUID | Group       |
| student_id | UUID | Student     |

Unique constraint:

```text
group_id + student_id
```

A student should belong to at most one active group within the same group-generation state.

---

# 17. attendance_records

| Field             | Type     | Description      |
| ----------------- | -------- | ---------------- |
| id                | UUID     | Primary key      |
| lesson_session_id | UUID     | Lesson           |
| student_id        | UUID     | Student          |
| status            | enum     | Attendance state |
| note              | string   | Optional         |
| created_at        | datetime | Created          |
| updated_at        | datetime | Updated          |

Status:

```text
present
absent
late
early_leave
official_absence
bereavement
other
```

---

# 18. evaluation_sets

| Field       | Type     | Description    |
| ----------- | -------- | -------------- |
| id          | UUID     | Primary key    |
| course_id   | UUID     | Course         |
| name        | string   | Evaluation set |
| description | string   | Optional       |
| created_at  | datetime | Created        |
| updated_at  | datetime | Updated        |

---

# 19. evaluation_items

| Field             | Type     | Description     |
| ----------------- | -------- | --------------- |
| id                | UUID     | Primary key     |
| evaluation_set_id | UUID     | Evaluation set  |
| name              | string   | Item name       |
| type              | enum     | Evaluation type |
| default_value     | string   | Optional        |
| sort_order        | integer  | Display order   |
| created_at        | datetime | Created         |
| updated_at        | datetime | Updated         |

Types:

```text
checkbox
scale
numeric
select
```

---

# 20. evaluation_options

| Field              | Type    | Description   |
| ------------------ | ------- | ------------- |
| id                 | UUID    | Primary key   |
| evaluation_item_id | UUID    | Item          |
| label              | string  | Display label |
| value              | string  | Stored value  |
| sort_order         | integer | Order         |

---

# 21. lesson_evaluation_sets

Stores which evaluation set was used for a specific lesson.

| Field             | Type | Description    |
| ----------------- | ---- | -------------- |
| id                | UUID | Primary key    |
| lesson_session_id | UUID | Lesson         |
| evaluation_set_id | UUID | Evaluation set |

Historical integrity must be preserved.

If evaluation definitions are changed later, historical records must remain interpretable.

---

# 22. evaluation_records

| Field              | Type     | Description  |
| ------------------ | -------- | ------------ |
| id                 | UUID     | Primary key  |
| lesson_session_id  | UUID     | Lesson       |
| student_id         | UUID     | Student      |
| evaluation_item_id | UUID     | Item         |
| value              | string   | Stored value |
| created_at         | datetime | Created      |
| updated_at         | datetime | Updated      |

Unique constraint:

```text
lesson_session_id + student_id + evaluation_item_id
```

---

# 23. teacher_memos

| Field             | Type     | Description     |
| ----------------- | -------- | --------------- |
| id                | UUID     | Primary key     |
| student_id        | UUID     | Student         |
| lesson_session_id | UUID     | Optional lesson |
| content           | string   | Memo            |
| created_at        | datetime | Created         |
| updated_at        | datetime | Updated         |

---

# 24. seating_constraints

| Field        | Type    | Description     |
| ------------ | ------- | --------------- |
| id           | UUID    | Primary key     |
| course_id    | UUID    | Course          |
| type         | enum    | Constraint type |
| student_a_id | UUID    | Optional        |
| student_b_id | UUID    | Optional        |
| priority     | integer | Priority        |
| value        | string  | Optional        |
| enabled      | boolean | Active          |

Types:

```text
separate
together
front_preference
back_preference
avoid_previous_seat
avoid_previous_neighbor
avoid_previous_group
```

---

# 25. grouping_constraints

Same conceptual structure as seating constraints.

Supported types:

```text
separate
together
avoid_previous_group
preferred_group
```

---

# 26. Backup Structure

Backup file:

```json
{
  "schemaVersion": "1.0.0",
  "appVersion": "0.1.0",
  "createdAt": "2026-08-18T00:00:00Z",
  "schoolYear": "2026年度",
  "data": {
    "schoolYears": [],
    "classes": [],
    "students": [],
    "courses": [],
    "courseEnrollments": [],
    "timetableEntries": [],
    "lessonSessions": [],
    "lessonParticipants": [],
    "seatingLayouts": [],
    "seats": [],
    "seatAssignments": [],
    "layoutObjects": [],
    "groups": [],
    "groupMembers": [],
    "attendanceRecords": [],
    "evaluationSets": [],
    "evaluationItems": [],
    "evaluationOptions": [],
    "lessonEvaluationSets": [],
    "evaluationRecords": [],
    "teacherMemos": [],
    "seatingConstraints": [],
    "groupingConstraints": []
  }
}
```

---

# 27. Data Integrity Rules

The implementation must prevent:

1. Duplicate Student IDs.
2. Duplicate students within the same class and attendance number.
3. Same student assigned to multiple seats in the same layout.
4. Student assigned to a group that does not belong to the lesson.
5. Attendance record for a non-participant.
6. Evaluation record for a non-participant.
7. Invalid evaluation item references.
8. Invalid course references.
9. Broken foreign-key relationships.
10. Invalid backup schema versions.

---

# 28. Historical Data Rules

Historical LessonSession data must be treated as immutable after completion except for explicit correction operations.

Changing:

* current seating
* current group rules
* current evaluation definitions
* current class configuration

must not rewrite past lesson records.

If a correction to a historical record is allowed, it must be explicit and auditable.

---

# 29. Storage Architecture

The application must use a local persistent database.

Do not use localStorage as the primary database.

Recommended candidates:

* IndexedDB
* SQLite WASM

The final choice must be documented in ARCHITECTURE.md.

All application data access should go through a repository/data-access layer rather than directly accessing the database from UI components.
