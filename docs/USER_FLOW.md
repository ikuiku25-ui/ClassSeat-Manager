# ClassSeat Manager — User Flow

# 1. Core Principle

The application must support a natural classroom workflow.

Primary flow:

```text
Today's Lesson
 ↓
Open Lesson
 ↓
Seating Chart
 ↓
Attendance
 ↓
Evaluation
 ↓
Group Activity
 ↓
Seating Shuffle
 ↓
Lesson Completion
```

---

# 2. Daily Start

## Step 1

Teacher opens the application.

## Step 2

The home screen automatically displays today's date.

Example:

```text
2026年8月18日（火）
```

## Step 3

Today's timetable is displayed.

```text
1限 情報Ⅰ 1年1組
2限 情報Ⅰ 1年2組
3限 情報探究 合同
```

---

# 3. Start Lesson

Teacher selects the lesson.

The application opens the LessonSession.

Display:

* course
* class/participant information
* date
* period
* seating layout

---

# 4. Attendance Flow

Teacher sees the seating chart.

Students are displayed in their seats.

Teacher selects a student.

Attendance options appear:

```text
出席
欠席
遅刻
早退
公欠
忌引
その他
```

Teacher selects the status.

The change is saved automatically.

---

# 5. Attendance Confirmation

Teacher opens:

```text
出欠一覧
```

The system displays all participants.

Unconfirmed students are clearly marked.

Example:

```text
01 山田　出席
02 佐藤　欠席
03 鈴木　未確認
```

Teacher confirms all students.

If the teacher attempts to finish the lesson with unconfirmed attendance, display a warning.

---

# 6. Student Profile During Lesson

Teacher selects a student.

A side panel appears.

Display:

* current attendance
* today's evaluation
* student history
* teacher memo

Teacher can add a memo without leaving the lesson.

---

# 7. Evaluation Flow

Teacher opens the evaluation area.

The evaluation set assigned to the lesson is displayed.

Example:

```text
発言
課題への取り組み
グループ活動
振り返り
```

Teacher selects the relevant student and changes the required evaluation item.

The result saves automatically.

---

# 8. Group Activity Flow

Teacher selects:

```text
[グループ活動]
```

If groups have already been generated, display them.

If not:

```text
[グループを編成]
```

---

# 9. Generate Groups

Teacher selects:

```text
グループ数
```

or:

```text
1グループ人数
```

Optional conditions:

```text
前回同じグループを避ける
禁止ペア
希望ペア
```

Teacher clicks:

```text
[編成する]
```

---

# 10. Review Groups

The system displays:

```text
A班
01 山田
08 佐藤
15 鈴木
22 高橋

B班
02 高橋
06 伊藤
19 田中
```

Teacher reviews.

Teacher can:

* regenerate
* manually adjust
* confirm

---

# 11. Apply Group Layout

After confirmation:

```text
Groups
 ↓
Group Seating Layout
```

The seating chart updates.

Group colors and group names appear.

---

# 12. Seating Shuffle Flow

Teacher selects:

```text
[席替え]
```

Teacher opens the teacher-only configuration screen.

---

# 13. Configure Seating Conditions

Teacher can configure:

### Absolute conditions

```text
山田 × 佐藤
隣接禁止
```

### Preferences

```text
山田
前方優先
```

### History

```text
前回の席を避ける
前回の隣席を避ける
前回のグループを避ける
```

---

# 14. Generate Seating

Teacher clicks:

```text
[配置を生成]
```

The algorithm calculates candidate layouts.

The teacher sees:

* valid/invalid
* score
* constraint result
* warnings

---

# 15. Confirm Seating

If valid:

```text
[確定]
```

If invalid:

```text
[条件を調整]
```

Do not automatically accept a hard constraint violation.

---

# 16. Student Display

After teacher confirmation, switch to student display.

Display:

```text
🎉 席替えスタート！

3

2

1

あなたの新しい席は……

12番！
```

The display must not reveal teacher-only conditions.

---

# 17. Apply New Seats

After the display:

```text
[新しい席を確定]
```

The new assignment becomes the current seating arrangement.

The previous assignment remains in the historical lesson data.

---

# 18. Lesson Completion

Teacher selects:

```text
[授業終了]
```

The system checks:

* attendance completeness
* evaluation input
* data save state

If attendance is incomplete:

```text
⚠ 出欠未確認の生徒が3名います。
```

Teacher can:

```text
[確認して終了]
[そのまま終了]
[戻る]
```

The default should encourage confirmation.

---

# 19. After Lesson

LessonSession becomes:

```text
completed
```

Historical data remains available.

---

# 20. Attendance Review Flow

From navigation:

```text
出欠管理
```

Teacher selects:

* year
* class
* period/date range

System displays attendance summary.

---

# 21. Evaluation Review Flow

From navigation:

```text
評価管理
```

Teacher selects:

* course
* evaluation set
* date range
* student

System displays evaluation history.

---

# 22. Student Review Flow

From navigation:

```text
生徒
```

Teacher selects a student.

Student profile displays:

```text
基本情報
↓
出欠
↓
評価
↓
授業履歴
↓
教師メモ
```

---

# 23. Timetable Flow

Teacher opens:

```text
時間割
```

Teacher can:

* create entry
* edit entry
* delete entry
* activate/deactivate entry

The home screen uses timetable entries to generate today's lessons.

---

# 24. CSV Import Flow

Teacher selects:

```text
設定
 ↓
生徒
 ↓
CSVインポート
```

System:

1. Selects file.
2. Reads header.
3. Validates columns.
4. Validates duplicate numbers.
5. Shows preview.
6. Requests confirmation.
7. Imports data.

Invalid rows should be reported before import.

---

# 25. Backup Flow

Teacher selects:

```text
設定
 ↓
バックアップ
 ↓
[バックアップを作成]
```

Browser downloads the backup file.

---

# 26. Restore Flow

Teacher selects:

```text
設定
 ↓
バックアップ
 ↓
[復元]
```

System:

1. Selects backup.
2. Validates schema version.
3. Validates references.
4. Shows summary.
5. Requires confirmation.
6. Restores data.

Do not overwrite existing data without explicit confirmation.

---

# 27. Printing Flow

Teacher selects:

```text
通常席図
 ↓
[印刷]
```

or:

```text
グループ席図
 ↓
[印刷]
```

Print format:

A4 Landscape.

---

# 28. Error Recovery

If an error occurs:

* do not silently lose data
* show a teacher-friendly message
* preserve unsaved data where possible
* provide retry/recovery option

---

# 29. Daily Minimum Workflow

The minimum expected daily workflow is:

```text
Open App
 ↓
Today's Lesson
 ↓
Open Lesson
 ↓
Attendance
 ↓
Evaluation
 ↓
Lesson Complete
```

The teacher should not need to visit multiple settings screens during a normal lesson.

---

# 30. Advanced Workflow

For group activity:

```text
Today's Lesson
 ↓
Open Lesson
 ↓
Group Activity
 ↓
Generate Groups
 ↓
Review
 ↓
Apply Group Layout
 ↓
Activity
```

For seating shuffle:

```text
Lesson
 ↓
Seating Shuffle
 ↓
Configure Constraints
 ↓
Generate
 ↓
Review
 ↓
Student Display
 ↓
Confirm
```
