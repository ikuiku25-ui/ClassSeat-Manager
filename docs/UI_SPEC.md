# ClassSeat Manager — UI Specification

## 1. UX Philosophy

The application is used during live classroom teaching.

Therefore:

1. Minimize clicks.
2. Minimize typing.
3. Avoid unnecessary dialogs.
4. Auto-save frequent classroom actions.
5. Make student status immediately visible.
6. Keep the seating chart central.
7. Do not interrupt the lesson with complex UI.

Primary workflow:

```text
今日の授業
→ 授業開始
→ 席図
→ 生徒
→ 出欠 / 評価
```

---

# 2. Global Navigation

Main navigation:

```text
🏠 今日の授業
🪑 席図・グループ
📝 出欠管理
📊 評価管理
👤 生徒
🕐 時間割
⚙ 設定
```

On desktop, use a persistent sidebar or top navigation.

---

# 3. Home Screen

## 3.1 Today's Lesson

Display:

```text
2026年8月18日（火）

1限　情報Ⅰ　　　1年1組
2限　情報Ⅰ　　　1年2組
3限　情報探究　　合同
4限　情報Ⅰ　　　1年5組
```

Each lesson is clickable.

---

## 3.2 Lesson Status

Possible states:

* 未開始
* 授業中
* 完了
* 休講

Display status clearly.

---

# 4. Lesson Screen

The Lesson Screen is the main working screen.

Header:

```text
情報Ⅰ
1年1組
3限
2026/08/18
```

Navigation tabs:

```text
[通常授業]
[グループ活動]
[出欠一覧]
[評価]
```

Main area:

```text
┌─────────────────────────────┐
│                             │
│          Seating Chart       │
│                             │
└─────────────────────────────┘
```

---

# 5. Seating Chart

Seats are displayed as movable cards.

Each seat can show:

```text
01
山田 太郎
```

or:

```text
01
山田 太郎
A班
```

Attendance status can be displayed.

Example:

```text
┌──────────┐
│ 01       │
│ 山田太郎 │
│ A班      │
└──────────┘
```

---

# 6. Attendance Display

Absent:

* gray seat
* "欠席" label

Late:

* "遅刻"

Early leave:

* "早退"

Official absence:

* "公欠"

Do not rely on color alone.

---

# 7. Student Interaction

Clicking/tapping a student opens a side panel or modal.

Example:

```text
┌────────────────────────┐
│ 01 山田 太郎            │
├────────────────────────┤
│ 出欠                    │
│ ○ 出席                  │
│ ○ 欠席                  │
│ ○ 遅刻                  │
│ ○ 早退                  │
│ ○ 公欠                  │
├────────────────────────┤
│ 今日の評価              │
│ □ 発言                  │
│ □ 課題                  │
│ □ グループ活動          │
├────────────────────────┤
│ [生徒カルテ]            │
└────────────────────────┘
```

---

# 8. Attendance List

Display all lesson participants.

Columns:

```text
番号
氏名
所属クラス
出欠
```

Status can be edited directly.

Provide filters:

* 全員
* 未確認
* 欠席
* 遅刻
* 早退
* 公欠

---

# 9. Evaluation Panel

Display the evaluation items configured for the current lesson.

Example:

```text
今日の評価

発言
[未評価]

課題への取り組み
[未評価]

グループ活動
[未評価]
```

The teacher should be able to mark only relevant students.

---

# 10. Evaluation Input Types

## Checkbox

```text
☐ 発言
```

## Scale

```text
◎ ○ △
```

## Numeric

```text
[ 3 ]
```

## Select

```text
[提出 ▼]
```

---

# 11. Evaluation Bulk Interaction

Because the normal state is assumed to be unchanged/unmarked, the UI should support quick marking of individual students.

Optional future feature:

Select multiple students and apply an evaluation simultaneously.

---

# 12. Student Profile

Open as side panel or modal.

Display:

```text
山田 太郎
出席番号：01

【出欠】
出席 35
欠席 2
遅刻 1
早退 0
公欠 1

【評価】
発言 12回
課題 28回
活動 15回

【最近の授業】
8/18 出席
8/12 出席
8/05 遅刻

【教師メモ】
8/18
グループ活動で積極的。
```

---

# 13. Teacher Memo

Allow adding a memo without leaving the current screen.

Example:

```text
[メモを追加]

________________________

[保存]
```

Auto-save is preferred where practical.

---

# 14. Seating Layout Editor

The editor contains:

```text
┌───────────────────────────────┐
│ Toolbar                       │
│ [座席] [黒板] [WB] [教卓] [PC]│
├───────────────────────────────┤
│                               │
│          Canvas               │
│                               │
└───────────────────────────────┘
```

Objects can be dragged.

---

# 15. Seat Management

Support:

* Add seat
* Delete seat
* Move seat
* Resize seat
* Assign student
* Change group color

Seat count must remain between 20 and 50 for normal configured layouts.

---

# 16. Group Layout

Display group name and color.

Example:

```text
      A班
┌────┐ ┌────┐
│山田│ │佐藤│
└────┘ └────┘

┌────┐ ┌────┐
│鈴木│ │田中│
└────┘ └────┘
```

---

# 17. Group Management Screen

Display:

```text
A班
4人
01 山田
08 佐藤
15 鈴木
22 高橋
```

Actions:

* Rename group
* Change color
* Add/remove member
* Randomize groups
* Apply groups to seating layout

---

# 18. Random Group Dialog

Inputs:

```text
グループ数 [ 8 ]

または

1グループ人数 [ 4 ]
```

Options:

```text
☑ 前回同じグループを避ける
☑ 禁止ペアを守る
☑ 希望ペアを考慮する
```

Buttons:

```text
[編成する]
```

---

# 19. Seating Shuffle Screen

Teacher view:

```text
席替え設定

【絶対条件】
山田 × 佐藤
[隣接禁止]

【希望】
山田
[前方優先]

☑ 前回と同じ席を避ける
☑ 前回近かった生徒を避ける

[配置を生成]
```

---

# 20. Seating Shuffle Result

Before applying:

```text
生成結果

条件達成率：98%

Hard Constraint
✓ すべて達成

Soft Constraint
✓ 前回席回避
△ 前方希望 1名

[再生成]
[確定]
```

If a hard constraint cannot be satisfied:

```text
⚠ 条件を満たす配置が存在しません。

[条件を確認]
[戻る]
```

Do not silently accept a violation.

---

# 21. Student Display Mode

The teacher can switch to a student-facing display.

Example:

```text
        席替え！

          3
          2
          1

       あなたの席は……

          12番！
```

Do not show:

* constraints
* student relationships
* scores
* algorithm details
* teacher settings

---

# 22. Seating Chart Printing

A4 landscape.

Print layout:

```text
2026年度
情報Ⅰ　1年1組
2026年8月18日　3限

              黒板

[01 山田] [02 佐藤] [03 鈴木]

[04 高橋] [05 田中] [06 伊藤]
```

The browser should use dedicated print CSS.

---

# 23. Normal / Group Printing

Provide separate actions:

```text
[通常席図を印刷]
[グループ席図を印刷]
```

Do not combine them into one printout.

---

# 24. Timetable Screen

Display weekly timetable.

Example:

```text
       月    火    水    木    金
1限   情報Ⅰ 情報Ⅰ
2限         情報Ⅰ
3限   探究       情報Ⅰ
```

Clicking a timetable entry allows editing.

---

# 25. Student Management Screen

Display:

```text
1年1組

01 山田太郎
02 佐藤花子
03 鈴木一郎
```

Actions:

* Add student
* Edit student
* Remove/deactivate student
* CSV import
* CSV export

---

# 26. Attendance Management Screen

Provide:

* student filter
* class filter
* date range
* status filter

Example:

```text
生徒      出席 欠席 遅刻 早退 公欠

山田       35   2    1    0    1
佐藤       38   0    0    0    0
```

---

# 27. Evaluation Management Screen

Display:

```text
評価セット

情報Ⅰ 通常授業
 ├─ 発言
 ├─ 課題
 ├─ グループ活動
 └─ 振り返り

プログラミング
 ├─ コード理解
 ├─ デバッグ
 └─ 課題達成
```

Actions:

* Add set
* Edit set
* Add item
* Edit item
* Reorder item
* Configure scale

---

# 28. Settings

Settings sections:

```text
年度
クラス
授業
生徒
時間割
評価
席替え条件
データ
バックアップ
```

---

# 29. Responsive Design

Desktop-first.

Minimum usable width should support common Windows laptop/desktop screens.

Avoid hard-coded dimensions that prevent future tablet support.

---

# 30. UX Requirements

The teacher should be able to:

1. Open today's lesson.
2. See the seating chart.
3. Select a student.
4. Change attendance.
5. Enter evaluation.
6. Open student profile.

without leaving the lesson screen.

---

# 31. Auto Save

Attendance and evaluation changes should be saved automatically.

The UI should provide subtle save status where appropriate.

Example:

```text
✓ 保存済み
```

Avoid requiring a save button after every individual action.

---

# 32. Error Messages

Messages must be teacher-friendly.

Bad:

```text
ConstraintError: null reference
```

Good:

```text
この席替え条件をすべて満たす配置を作成できませんでした。
「山田さんと佐藤さんを離す」という条件を確認してください。
```

---

# 33. Accessibility

At minimum:

* keyboard navigation for important controls
* visible focus state
* color not being the sole status indicator
* readable font sizes
* sufficient contrast
* clear status labels
