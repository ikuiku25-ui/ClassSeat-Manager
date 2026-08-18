# ClassSeat Manager

## 詳細設計仕様書

Version: 1.0
Target: Claude CodeによるMVP開発
Platform: オフラインWebアプリ / Windows PC
Primary User: 高等学校教員

---

# 0. 設計全体像

ClassSeat Managerは、以下の構造を基本とする。

```text
年度
 ├─ クラス
 │   └─ 生徒
 │
 ├─ 授業
 │   └─ 授業参加者
 │
 ├─ 時間割
 │
 └─ 授業セッション
      │
      ├─ 席レイアウト
      │    └─ 座席配置
      │
      ├─ グループ
      │    └─ グループメンバー
      │
      ├─ 出欠
      │
      ├─ 評価
      │
      └─ 教師メモ
```

最重要原則：

> 「現在の状態」と「過去の授業履歴」を混同しない。

---

# 1. ER図

## 1.1 概念ER図

```text
┌──────────────┐
│ SchoolYear   │
│──────────────│
│ id           │
│ year         │
│ name         │
└──────┬───────┘
       │
       ├───────────────────────┐
       │                       │
       ▼                       ▼
┌──────────────┐        ┌──────────────┐
│ Class        │        │ Course       │
│──────────────│        │──────────────│
│ id           │        │ id           │
│ schoolYearId │        │ name         │
│ grade        │        │ description  │
│ name         │        └──────┬───────┘
└──────┬───────┘               │
       │                       │
       ▼                       ▼
┌──────────────┐        ┌──────────────────┐
│ Student      │        │ CourseEnrollment │
│──────────────│        │──────────────────│
│ id           │◄───────│ studentId        │
│ classId      │        │ courseId         │
│ number       │        │ enrolledFrom     │
│ name         │        │ enrolledTo       │
└──────┬───────┘        └────────┬─────────┘
       │                          │
       │                          ▼
       │                  ┌──────────────┐
       │                  │ LessonSession│
       │                  │──────────────│
       └─────────────────►│ courseId     │
                          │ date         │
                          │ period       │
                          └──────┬───────┘
                                 │
          ┌──────────────────────┼────────────────────────┐
          │                      │                        │
          ▼                      ▼                        ▼
┌─────────────────┐    ┌─────────────────┐      ┌─────────────────┐
│ SeatingLayout   │    │ Attendance      │      │ Group           │
│─────────────────│    │─────────────────│      │─────────────────│
│ id              │    │ id              │      │ id              │
│ sessionId       │    │ sessionId       │      │ sessionId       │
│ type            │    │ studentId       │      │ name            │
└────────┬────────┘    │ status          │      │ colorIndex      │
         │             └─────────────────┘      └────────┬────────┘
         ▼                                               │
┌─────────────────┐                                      ▼
│ Seat            │                              ┌─────────────────┐
│─────────────────│                              │ GroupMember     │
│ id              │                              │─────────────────│
│ layoutId        │                              │ groupId         │
│ seatNumber      │                              │ studentId       │
│ x               │                              └─────────────────┘
│ y               │
│ width           │
│ height          │
└────────┬────────┘
         │
         ▼
┌─────────────────────┐
│ SeatAssignment      │
│─────────────────────│
│ id                  │
│ seatId              │
│ studentId           │
│ assignedAt          │
└─────────────────────┘

LessonSession
      │
      ├──────────────► EvaluationRecord
      │                    │
      │                    ▼
      │              EvaluationItem
      │
      └──────────────► TeacherMemo
```

---

# 2. DBテーブル定義

DBはSQL的な構造を想定する。

実装方式がIndexedDBの場合も、以下の論理モデルを維持する。

---

## 2.1 school_years

年度を管理。

| Field      | Type     | Required | Description |
| ---------- | -------- | -------: | ----------- |
| id         | UUID     |      Yes | 主キー         |
| year       | integer  |      Yes | 2026など      |
| name       | string   |      Yes | 2026年度      |
| is_active  | boolean  |      Yes | 現在年度        |
| created_at | datetime |      Yes | 作成日時        |
| updated_at | datetime |      Yes | 更新日時        |

制約：

* yearは重複不可
* is_active=trueは原則1件

---

# 2.2 classes

| Field          | Type     | Required | Description |
| -------------- | -------- | -------: | ----------- |
| id             | UUID     |      Yes | 主キー         |
| school_year_id | UUID     |      Yes | 年度          |
| grade          | integer  |      Yes | 学年          |
| name           | string   |      Yes | 1組など        |
| display_name   | string   |      Yes | 1年1組        |
| created_at     | datetime |      Yes |             |
| updated_at     | datetime |      Yes |             |

---

# 2.3 students

| Field             | Type     | Required | Description |
| ----------------- | -------- | -------: | ----------- |
| id                | UUID     |      Yes | 主キー         |
| class_id          | UUID     |      Yes | 所属クラス       |
| attendance_number | integer  |      Yes | 出席番号        |
| name              | string   |      Yes | 氏名          |
| created_at        | datetime |      Yes |             |
| updated_at        | datetime |      Yes |             |
| is_active         | boolean  |      Yes | 在籍状態        |

重要：

attendance_numberを主キーにしない。

合同授業では異なるクラスに同じ出席番号が存在するため。

---

# 2.4 courses

授業そのもの。

| Field          | Type     | Required | Description |
| -------------- | -------- | -------: | ----------- |
| id             | UUID     |      Yes | 主キー         |
| school_year_id | UUID     |      Yes | 年度          |
| name           | string   |      Yes | 情報Ⅰなど       |
| description    | string   |       No | 説明          |
| created_at     | datetime |      Yes |             |
| updated_at     | datetime |      Yes |             |

---

# 2.5 course_enrollments

授業への生徒参加情報。

| Field      | Type     | Required | Description |
| ---------- | -------- | -------: | ----------- |
| id         | UUID     |      Yes |             |
| course_id  | UUID     |      Yes |             |
| student_id | UUID     |      Yes |             |
| created_at | datetime |      Yes |             |
| ended_at   | datetime |       No |             |

これにより、

「1年1組の生徒だが、情報探究では合同授業に参加」

を表現できる。

---

# 2.6 timetable_entries

時間割。

| Field         | Type    | Required | Description |
| ------------- | ------- | -------: | ----------- |
| id            | UUID    |      Yes |             |
| course_id     | UUID    |      Yes |             |
| day_of_week   | integer |      Yes |             |
| period        | integer |      Yes |             |
| display_order | integer |      Yes |             |
| is_active     | boolean |      Yes |             |

将来的に特定日変更を追加できる構造にする。

---

# 2.7 lesson_sessions

最重要テーブル。

「実際に行われた1回の授業」を表す。

| Field      | Type     | Required | Description |
| ---------- | -------- | -------: | ----------- |
| id         | UUID     |      Yes |             |
| course_id  | UUID     |      Yes |             |
| date       | date     |      Yes |             |
| period     | integer  |      Yes |             |
| status     | enum     |      Yes |             |
| created_at | datetime |      Yes |             |
| updated_at | datetime |      Yes |             |

status例：

* planned
* active
* completed
* cancelled

---

# 2.8 lesson_participants

その授業に実際に参加する生徒。

| Field             | Type | Required | Description |
| ----------------- | ---- | -------: | ----------- |
| id                | UUID |      Yes |             |
| lesson_session_id | UUID |      Yes |             |
| student_id        | UUID |      Yes |             |

合同授業を処理する重要テーブル。

---

# 2.9 seating_layouts

| Field             | Type     | Required | Description |
| ----------------- | -------- | -------: | ----------- |
| id                | UUID     |      Yes |             |
| lesson_session_id | UUID     |      Yes |             |
| layout_type       | enum     |      Yes |             |
| name              | string   |      Yes |             |
| created_at        | datetime |      Yes |             |
| updated_at        | datetime |      Yes |             |

layout_type：

* normal
* group

授業ごとの履歴を保持する。

---

# 2.10 seats

| Field       | Type     | Required | Description |
| ----------- | -------- | -------: | ----------- |
| id          | UUID     |      Yes |             |
| layout_id   | UUID     |      Yes |             |
| seat_number | integer  |      Yes |             |
| x           | number   |      Yes |             |
| y           | number   |      Yes |             |
| width       | number   |      Yes |             |
| height      | number   |      Yes |             |
| rotation    | number   |      Yes |             |
| created_at  | datetime |      Yes |             |
| updated_at  | datetime |      Yes |             |

20～50席。

---

# 2.11 layout_objects

座席以外のオブジェクト。

| Field       | Type   | Required | Description |
| ----------- | ------ | -------: | ----------- |
| id          | UUID   |      Yes |             |
| layout_id   | UUID   |      Yes |             |
| object_type | enum   |      Yes |             |
| x           | number |      Yes |             |
| y           | number |      Yes |             |
| width       | number |      Yes |             |
| height      | number |      Yes |             |
| text        | string |       No |             |

object_type：

* blackboard
* whiteboard
* teacher_desk
* teacher_pc
* text

---

# 2.12 seat_assignments

| Field       | Type     | Required | Description |
| ----------- | -------- | -------: | ----------- |
| id          | UUID     |      Yes |             |
| seat_id     | UUID     |      Yes |             |
| student_id  | UUID     |      Yes |             |
| assigned_at | datetime |      Yes |             |

重要：

同一レイアウト内で同じstudent_idを複数座席に割り当てない。

---

# 2.13 groups

| Field             | Type     | Required | Description |
| ----------------- | -------- | -------: | ----------- |
| id                | UUID     |      Yes |             |
| lesson_session_id | UUID     |      Yes |             |
| name              | string   |      Yes |             |
| color_index       | integer  |      Yes |             |
| created_at        | datetime |      Yes |             |

color_indexは0～19。

---

# 2.14 group_members

| Field      | Type | Required | Description |
| ---------- | ---- | -------: | ----------- |
| id         | UUID |      Yes |             |
| group_id   | UUID |      Yes |             |
| student_id | UUID |      Yes |             |

---

# 2.15 attendance_records

| Field             | Type     | Required | Description |
| ----------------- | -------- | -------: | ----------- |
| id                | UUID     |      Yes |             |
| lesson_session_id | UUID     |      Yes |             |
| student_id        | UUID     |      Yes |             |
| status            | enum     |      Yes |             |
| note              | string   |       No |             |
| updated_at        | datetime |      Yes |             |

status：

* present
* absent
* late
* early_leave
* official_absence
* bereavement
* other

---

# 2.16 evaluation_sets

授業ごとの評価セット。

| Field       | Type   | Required |
| ----------- | ------ | -------: |
| id          | UUID   |      Yes |
| course_id   | UUID   |      Yes |
| name        | string |      Yes |
| description | string |       No |

---

# 2.17 evaluation_items

| Field             | Type    | Required |
| ----------------- | ------- | -------: |
| id                | UUID    |      Yes |
| evaluation_set_id | UUID    |      Yes |
| name              | string  |      Yes |
| type              | enum    |      Yes |
| default_value     | string  |       No |
| sort_order        | integer |      Yes |

type：

* checkbox
* scale
* numeric
* select

---

# 2.18 evaluation_options

select/scale用。

| Field              | Type    | Required |
| ------------------ | ------- | -------: |
| id                 | UUID    |      Yes |
| evaluation_item_id | UUID    |      Yes |
| label              | string  |      Yes |
| value              | string  |      Yes |
| sort_order         | integer |      Yes |

---

# 2.19 lesson_evaluation_sets

授業セッションで実際に使用した評価セット。

| Field             | Type | Required |
| ----------------- | ---- | -------: |
| id                | UUID |      Yes |
| lesson_session_id | UUID |      Yes |
| evaluation_set_id | UUID |      Yes |

履歴保護のため、必要に応じて評価項目のスナップショットを保持する。

---

# 2.20 evaluation_records

| Field              | Type     | Required |
| ------------------ | -------- | -------: |
| id                 | UUID     |      Yes |
| lesson_session_id  | UUID     |      Yes |
| student_id         | UUID     |      Yes |
| evaluation_item_id | UUID     |      Yes |
| value              | string   |      Yes |
| created_at         | datetime |      Yes |
| updated_at         | datetime |      Yes |

---

# 2.21 teacher_memos

| Field             | Type     | Required |
| ----------------- | -------- | -------: |
| id                | UUID     |      Yes |
| student_id        | UUID     |      Yes |
| lesson_session_id | UUID     |       No |
| content           | string   |      Yes |
| created_at        | datetime |      Yes |
| updated_at        | datetime |      Yes |

lesson_session_idをNULLにすることで、授業に紐づかない一般メモも将来可能。

---

# 2.22 seating_constraints

席替え条件。

| Field        | Type    | Required |
| ------------ | ------- | -------: |
| id           | UUID    |      Yes |
| course_id    | UUID    |      Yes |
| type         | enum    |      Yes |
| student_a_id | UUID    |       No |
| student_b_id | UUID    |       No |
| priority     | integer |      Yes |
| value        | string  |       No |
| enabled      | boolean |      Yes |

type例：

* separate
* together
* front_preference
* back_preference
* avoid_previous_seat
* avoid_previous_neighbor
* avoid_previous_group

---

# 2.23 grouping_constraints

グループ編成条件。

基本構造はseating_constraintsと同様。

---

# 2.24 backup_metadata

| Field          | Type     | Required |
| -------------- | -------- | -------: |
| schema_version | string   |      Yes |
| app_version    | string   |      Yes |
| created_at     | datetime |      Yes |
| school_year    | string   |      Yes |

---

# 3. 画面遷移図

## 3.1 全体

```text
                    ┌─────────────┐
                    │   起動画面   │
                    └──────┬──────┘
                           ▼
                    ┌─────────────┐
                    │ 今日の授業   │
                    └──────┬──────┘
                           │
             ┌─────────────┼──────────────┐
             ▼             ▼              ▼
       ┌──────────┐  ┌──────────┐  ┌──────────┐
       │ 通常席図 │  │グループ席│  │  出欠一覧 │
       └────┬─────┘  └────┬─────┘  └──────────┘
            │              │
            └──────┬───────┘
                   ▼
             ┌────────────┐
             │ 生徒詳細   │
             └─────┬──────┘
                   │
       ┌───────────┼─────────────┐
       ▼           ▼             ▼
    出欠入力     評価入力      生徒カルテ
                                 │
                                 ▼
                              教師メモ
```

---

# 3.2 メインナビゲーション

```text
ホーム
├─ 今日の授業
├─ 席図・グループ
├─ 出欠管理
├─ 評価管理
├─ 生徒
├─ 時間割
└─ 設定
```

---

# 3.3 席図エディタ

```text
席図管理
 │
 ├─ 通常授業
 │    └─ 編集
 │         ├─ 座席追加
 │         ├─ 座席削除
 │         ├─ ドラッグ
 │         ├─ 生徒割当
 │         └─ 印刷
 │
 └─ グループ活動
      └─ 編集
           ├─ グループ編成
           ├─ 座席配置
           └─ 印刷
```

---

# 3.4 席替え

```text
席図
 ↓
席替え
 ↓
条件設定
 ↓
条件検証
 ↓
配置生成
 ↓
結果確認
 ↓
┌───────────────┐
│ 条件を満たした │
└──────┬────────┘
       ↓
 生徒向け演出
       ↓
 結果確定
```

条件を満たせない場合：

```text
配置生成
 ↓
条件違反
 ↓
警告
 ↓
条件調整
または
最善配置を教師が承認
```

---

# 4. 席替えアルゴリズム詳細

## 4.1 基本方針

単純ランダムは禁止。

「制約付きランダム配置」とする。

入力：

```text
Students
Seats
PreviousAssignment
Constraints
```

出力：

```text
CandidateAssignment
Score
ConstraintViolations
```

---

# 4.2 制約分類

## Hard Constraint

絶対条件。

例：

* AとBを隣接させない
* AとBを一定距離以上離す
* 指定席から動かさない

違反した候補は不採用。

---

## Soft Constraint

希望条件。

例：

* 前方優先
* 前回と違う席
* 前回同じグループを避ける
* 特定生徒との距離を広げる

スコアで評価。

---

# 4.3 距離

座席間距離は基本的に、

```text
distance = sqrt(
  (seatA.x - seatB.x)^2 +
  (seatA.y - seatB.y)^2
)
```

を利用。

ただし「隣席」は単純なユークリッド距離だけでなく、座席配置上の近傍関係として判定できるようにする。

---

# 4.4 隣接判定

以下を候補として検討。

* 左
* 右
* 前
* 後
* 斜め

設定によって、

「左右だけ」

「前後左右」

「周囲8方向」

を切り替えられる設計にする。

---

# 4.5 候補生成

基本フロー：

```text
1. 現在の参加者を取得
2. 欠席者を除外
3. 使用可能座席を取得
4. 固定席を確定
5. 残り生徒をシャッフル
6. 仮配置
7. Hard Constraintを検証
8. Soft Constraintを採点
9. 候補を保存
10. 複数回生成
11. 最良候補を選択
```

---

# 4.6 スコア

例：

```text
totalScore =
  previousSeatScore
+ neighborScore
+ frontPreferenceScore
+ groupHistoryScore
```

Hard Constraint違反は候補除外。

Soft Constraintは点数化。

---

# 4.7 ローカル改善

初期配置生成後、

```text
生徒Aと生徒Bを交換
 ↓
スコア改善？
 ↓
YES → 採用
NO → 元に戻す
```

を繰り返す方式を採用可能。

MVPでは単純なランダム試行＋スコアリングから開始し、性能・品質を確認して改善アルゴリズムを追加する。

---

# 4.8 結果の保存

席替えを確定したら、

* 使用した条件
* 生成日時
* 結果
* スコア
* 条件違反
* 実行者

を履歴として保存可能な構造にする。

過去結果を後から再計算して変更しない。

---

# 4.9 生徒表示

教師が「開始」を押した後、

```text
3
2
1
```

などの演出。

最終的に、

```text
01 山田
↓
12番席
```

を表示。

教師用条件は絶対に表示しない。

---

# 5. グループ編成アルゴリズム詳細

## 5.1 入力

* 対象生徒
* 欠席者
* グループ数
* 1グループ人数
* 前回グループ
* グループ制約

---

# 5.2 欠席者

欠席者は原則除外。

ただし、

「公欠・忌引は除外」

「遅刻者は含める」

など、出欠状態ごとの扱いを設定可能にする。

MVPでは、

* 欠席
* 公欠
* 忌引

を除外。

* 遅刻
* 早退

は参加対象とする。

---

# 5.3 人数計算

例：

37人、4人グループ。

```text
37 ÷ 4
= 9グループ + 1人
```

自動的に、

```text
4人 × 8
5人 × 1
```

などへ調整。

極端に人数差が出ないようにする。

---

# 5.4 前回グループ回避

前回同じグループだった組み合わせにペナルティを設定。

例：

```text
A-B：前回同じ
→ -10
```

複数回同じならさらにペナルティ。

---

# 5.5 禁止ペア

Hard Constraint。

```text
A × B
```

同一グループ禁止。

---

# 5.6 同一グループ希望

Soft ConstraintまたはHard Constraintとして設定可能。

---

# 5.7 ランダム性

毎回完全に同じ結果にならないよう乱数seedを利用する。

ただし、

「同じ条件＋同じseed」

なら同じ結果を再現できる設計にする。

---

# 5.8 グループスコア

例：

```text
score =
  previousGroupPenalty
+ forbiddenPairPenalty
+ preferredPairBonus
+ sizeBalanceScore
```

Hard Constraint違反は不採用。

---

# 5.9 グループ結果

確定後、

```text
A班
01 山田
08 佐藤
15 鈴木

B班
02 高橋
06 伊藤
19 田中
```

を表示。

そのままグループ活動レイアウトに反映できること。

---

# 6. 授業中の操作フロー

## 6.1 理想的な授業開始

```text
アプリ起動
 ↓
今日の授業
 ↓
3限 情報Ⅰ 1年1組
 ↓
授業開始
 ↓
通常席図表示
```

教師が追加入力する情報を最小化する。

---

# 6.2 出欠

席図を表示。

生徒をタップ。

```text
出席
欠席
遅刻
早退
公欠
忌引
```

を選択。

理想操作：

```text
生徒タップ
 ↓
状態タップ
```

2操作以内。

---

# 6.3 欠席者確認

授業開始後、

```text
[出欠一覧]
```

を開く。

欠席・遅刻などを一覧確認。

未入力生徒も表示。

授業開始時点で、

「出欠未確認」

を分かりやすくする。

---

# 6.4 授業中の評価

生徒をタップ。

```text
今日の評価
├─ 発言
├─ 課題
├─ グループ活動
└─ 振り返り
```

該当項目をタップ。

評価後、自動保存。

明示的な「保存」ボタンを毎回押さなくてよい。

---

# 6.5 グループ活動

```text
通常席図
 ↓
グループ編成
 ↓
条件設定
 ↓
自動編成
 ↓
教師確認
 ↓
グループ活動レイアウト
 ↓
プロジェクター表示
```

---

# 6.6 席替え

```text
席替え
 ↓
教師が条件設定
 ↓
自動生成
 ↓
教師確認
 ↓
生徒向け演出
 ↓
新席発表
 ↓
結果確定
```

---

# 6.7 生徒カルテ

授業中に生徒をタップ。

```text
生徒詳細
 ↓
カルテ
```

表示：

* 出欠
* 評価
* 最近の履歴
* 教師メモ

授業画面を離れず、モーダルまたはサイドパネルで表示する。

---

# 6.8 授業終了

授業終了時に、

```text
[授業終了]
```

を押す。

未入力の出欠がある場合、

```text
⚠ 出欠未入力：3名
```

と警告。

教師が確認後、completedへ変更。

---

# 7. MVP Issue一覧

GitHub Issueとして管理する。

---

## EPIC-001 プロジェクト基盤

### ISSUE-001

プロジェクト初期化

* TypeScript
* React
* Vite
* lint
* test
* build

Acceptance Criteria：

* npm install成功
* npm run dev成功
* npm run build成功

---

### ISSUE-002

ローカルDB基盤

* DB初期化
* migration/version管理
* repository layer

Acceptance Criteria：

* ブラウザ再起動後もデータ保持

---

### ISSUE-003

共通ID・日時・エラー処理

* UUID
* createdAt
* updatedAt
* schemaVersion

---

# EPIC-002 年度・クラス・生徒

### ISSUE-004

年度管理

### ISSUE-005

クラス管理

### ISSUE-006

生徒手入力

### ISSUE-007

生徒CSVインポート

### ISSUE-008

生徒一覧UI

Acceptance Criteria：

* 出席番号
* 氏名
* クラス
* 編集
* 削除

---

# EPIC-003 授業・時間割

### ISSUE-009

授業登録

### ISSUE-010

合同授業

### ISSUE-011

授業参加者管理

### ISSUE-012

時間割登録

### ISSUE-013

今日の授業

Acceptance Criteria：

今日の日付に応じて授業が表示される。

---

# EPIC-004 席図

### ISSUE-014

席図データモデル

### ISSUE-015

座席追加・削除

### ISSUE-016

ドラッグ配置

### ISSUE-017

座席サイズ変更

### ISSUE-018

生徒割り当て

### ISSUE-019

席番号表示

### ISSUE-020

黒板等オブジェクト

### ISSUE-021

グループ色

### ISSUE-022

通常レイアウト

### ISSUE-023

グループレイアウト

---

# EPIC-005 印刷

### ISSUE-024

A4横印刷CSS

### ISSUE-025

通常席図印刷

### ISSUE-026

グループ席図印刷

Acceptance Criteria：

実際のA4横紙に収まる。

---

# EPIC-006 出欠

### ISSUE-027

出欠データモデル

### ISSUE-028

席図から出欠

### ISSUE-029

出欠一覧

### ISSUE-030

欠席表示

### ISSUE-031

出欠集計

### ISSUE-032

出欠履歴

---

# EPIC-007 評価

### ISSUE-033

評価セット

### ISSUE-034

評価項目

### ISSUE-035

評価尺度

### ISSUE-036

授業への評価セット割当

### ISSUE-037

席図から評価入力

### ISSUE-038

評価履歴

### ISSUE-039

評価集計

---

# EPIC-008 生徒カルテ

### ISSUE-040

生徒詳細モーダル

### ISSUE-041

出欠サマリー

### ISSUE-042

評価サマリー

### ISSUE-043

授業履歴

### ISSUE-044

教師メモ

---

# EPIC-009 グループ編成

### ISSUE-045

グループ作成

### ISSUE-046

20色管理

### ISSUE-047

グループメンバー表示

### ISSUE-048

ランダムグループ生成

### ISSUE-049

欠席者除外

### ISSUE-050

前回グループ履歴

### ISSUE-051

グループ制約

### ISSUE-052

グループ結果を席図へ反映

---

# EPIC-010 席替え

### ISSUE-053

席替え条件モデル

### ISSUE-054

Hard Constraint

### ISSUE-055

Soft Constraint

### ISSUE-056

前回席回避

### ISSUE-057

隣接判定

### ISSUE-058

候補生成

### ISSUE-059

スコアリング

### ISSUE-060

制約違反検出

### ISSUE-061

最良配置選択

### ISSUE-062

席替え結果保存

### ISSUE-063

生徒向け演出画面

---

# EPIC-011 CSV・バックアップ

### ISSUE-064

生徒CSV出力

### ISSUE-065

出欠CSV出力

### ISSUE-066

評価CSV出力

### ISSUE-067

授業履歴CSV出力

### ISSUE-068

グループ履歴CSV出力

### ISSUE-069

席配置履歴CSV出力

### ISSUE-070

全データバックアップ

### ISSUE-071

バックアップ復元

### ISSUE-072

復元前検証

---

# EPIC-012 QA

### ISSUE-073

単体テスト

### ISSUE-074

DB整合性テスト

### ISSUE-075

合同授業テスト

### ISSUE-076

席替え制約テスト

### ISSUE-077

グループ編成テスト

### ISSUE-078

出欠テスト

### ISSUE-079

評価テスト

### ISSUE-080

バックアップ復元テスト

### ISSUE-081

A4印刷テスト

### ISSUE-082

ブラウザ再起動テスト

### ISSUE-083

大量データテスト

---

# 8. MVP開発順序

優先順位は以下。

```text
P0
基盤
 ↓
年度・クラス・生徒
 ↓
授業・時間割
 ↓
今日の授業

P1
席図
 ↓
通常／グループレイアウト
 ↓
印刷

P2
出欠
 ↓
集計

P3
評価
 ↓
生徒カルテ
 ↓
教師メモ

P4
グループ編成
 ↓
席替え

P5
CSV
 ↓
バックアップ
 ↓
復元

P6
QA
```

---

# 9. MVP完成判定

以下の実際のシナリオを通過したらMVP完成とする。

```text
2026年度を作成
 ↓
1年1組を作成
 ↓
40人をCSV登録
 ↓
情報Ⅰを登録
 ↓
時間割登録
 ↓
今日の授業を表示
 ↓
授業開始
 ↓
40人を席図へ配置
 ↓
席図を保存
 ↓
A4横印刷
 ↓
欠席者2人を登録
 ↓
評価項目を3つ設定
 ↓
数名を評価
 ↓
生徒カルテ確認
 ↓
グループ編成
 ↓
グループ席図へ切替
 ↓
グループ席図を印刷
 ↓
席替え条件設定
 ↓
条件付きランダム席替え
 ↓
生徒向け演出
 ↓
新席確定
 ↓
授業終了
 ↓
出欠・評価履歴確認
 ↓
CSV出力
 ↓
バックアップ
 ↓
データ削除後に復元
 ↓
過去の授業履歴が正常に復元
```

---

# 10. 最重要QAシナリオ

以下は必須。

## ケース1：合同授業

1年1組01番と1年2組01番が同じ授業に存在しても混同しない。

---

## ケース2：席替え

8/18の山田が5番席。

8/25の山田が18番席。

8/18の履歴を表示すると5番席のままである。

---

## ケース3：評価項目変更

8/18に「発言」という評価項目を使用。

後日、評価項目名を変更。

8/18の履歴が壊れない。

---

## ケース4：欠席者

欠席者がグループ編成から除外される。

---

## ケース5：条件矛盾

「AとBを隣接禁止」と「AとBを隣接必須」を同時指定。

システムが無理に配置せず、矛盾を報告する。

---

## ケース6：座席不足

生徒40人、座席35。

警告を表示し、確定を防止する。

---

## ケース7：バックアップ

バックアップ→削除→復元。

全データが復元される。

---

# 11. 実装時の禁止事項

以下は禁止。

* localStorageだけで全データを管理する
* 出席番号を主キーにする
* 生徒と座席を固定的に紐づける
* 現在の設定から過去履歴を再構築する
* Math.random()だけで席替えする
* 制約違反を黙って無視する
* 外部APIを必須にする
* インターネット接続を前提にする
* いきなり全機能を実装する
* 大量のダミーデータを本番DBへ混入させる

---

# 12. 設計上の最終原則

ClassSeat Managerの中心は、

「席図」

ではなく、

**「授業セッション」**

である。

授業セッションに、

* 誰が参加したか
* どこに座ったか
* 出欠はどうだったか
* どの評価を受けたか
* どのグループだったか
* 教師が何を記録したか

を紐づける。

これにより、現在だけではなく過去の授業を再現できる。

この原則をDB、UI、API、サービス層、テストのすべてに適用する。
