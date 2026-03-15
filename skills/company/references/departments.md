# 部署別テンプレート集

生産技術版 Company プラグインの各部署が使用する CLAUDE.md・_template.md・_index.md の定義集。

## 共通ルール

- **ファイル命名規則**
  - イベント型: `{YYYY-MM-DD}_{種別}_{タイトル}.md`
  - エンティティ型: `{ID}_{名称}.md` または `{名前}.md`
  - メタファイル: `_index.md` と `_template.md` のみ
- **種別 ENUM**: `memo` | `meeting` | `report` | `decision` | `task` | `incident`
- **frontmatter 必須フィールド**: `site`（`sanda` | `himeji` | `both` | `unknown`）
- **SSoT 原則**: 情報のコピーは作らない。リンクで参照する
- **requests ステータス**: `open` → `waiting` → `closed`

---

## 1. inbox（共通）

### inbox/_template.md

```markdown
---
captured_at: YYYY-MM-DD HH:MM
site: unknown
status: pending
---

{1行メモ}
```

### inbox/_index.md

```markdown
# Inbox

未処理のメモ一覧。整理時に各部署へ振り分ける。

| ファイル | キャプチャ日時 | status |
|---------|-------------|--------|
```

---

## 2. 秘書室（secretary）

### secretary/CLAUDE.md

```markdown
# 秘書室

## 役割

リーダーの常駐窓口。何でも相談に乗り、壁打ち・アイデア記録・会議メモ・定型業務リマインドを担当する。

## 口調・キャラクター

- 丁寧だが堅すぎない。「承知しました」「いいですね！」「なるほど」
- 主体的に提案する。「ついでにこれも整理しておきましょうか？」
- 過去のメモや決定事項を参照して文脈を持った対話をする
- リーダーを支える姿勢。「判断待ちが溜まっています。優先順位をつけましょうか？」

## ルール

- ユーザーからの入力はまず秘書が受け取る
- 秘書で完結するもの（壁打ち、アイデア、会議メモ、雑談）は直接対応
- 部署の作業が必要と判断したら意図ベースで振り分けを提案（L2）
- 壁打ちの結論が出たら ideas/ にアイデア抽出を提案
- 会議メモからアクションアイテムを抽出し、振り分けを提案

## フォルダ構成

- `ideas/` - アイデアDB（永続、タグ付き）
- `meetings/` - 会議メモ（永続）
- `brainstorm/` - 壁打ちログ（永続）
- `routines/` - 定型業務リマインド定義
```

### secretary/ideas/_index.md

```markdown
# アイデアDB

壁打ちやひらめきから生まれたアイデアの一覧。

| アイデア | タグ | 作成日 | ステータス |
|---------|------|--------|----------|
```

### secretary/ideas/_template.md

```markdown
---
title: "{アイデアタイトル}"
tags:
  - "{タグ1}"
  - "{タグ2}"
site: unknown
created: YYYY-MM-DD
source: brainstorm  # brainstorm | meeting | flash
---

## アイデア概要

{アイデアの要点を1〜3文で}

## 背景・きっかけ

{どの壁打ち・会議・ひらめきから生まれたか。ソースファイルへのリンク}

## 次のステップ

- [ ] {具体的なアクション}
```

### secretary/meetings/_index.md

```markdown
# 会議メモ一覧

| 会議名 | 日付 | 参加者 | アクション数 |
|--------|------|--------|------------|
```

### secretary/meetings/_template.md

```markdown
---
title: "{会議名}"
attendees:
  - "{参加者1}"
  - "{参加者2}"
site: unknown
created: YYYY-MM-DD
---

## 議題・要点

- {議題1}: {要点}
- {議題2}: {要点}

## アクションアイテム

- [ ] {アクション} → 振り分け先: {部署/担当}
- [ ] {アクション} → 振り分け先: {部署/担当}

## 決定事項

- {決定事項1}
- {決定事項2}

## 次回予定

- 日時: {YYYY-MM-DD HH:MM}
- 議題（予定）: {次回の議題}
```

### secretary/brainstorm/_index.md

```markdown
# 壁打ちログ一覧

| テーマ | 日付 | 結論 | アイデア化 |
|--------|------|------|----------|
```

### secretary/routines/_schedule.md

```yaml
routines:
  - name: inbox整理
    frequency: every_session
    trigger: session_start
    action: inbox/のpendingファイル一覧を表示し、振り分け提案

  - name: 週次レビュー
    frequency: weekly
    day: friday
    action: reviews/weekly/に週次レビュー生成

  - name: 月次報告ドラフト
    frequency: monthly
    day: last_friday
    action: reviews/monthly/に月次レビュー生成

  - name: リンク整合性チェック
    frequency: weekly
    day: friday
    action: 全Markdownファイルのリンク切れを検出・報告
```

---

## 3. DXグループ管理（dx-group）

### dx-group/CLAUDE.md

```markdown
# DXグループ管理

## 役割

チームメンバーの管理、スキルマップ、アサイン判断支援、引き継ぎノートを担当する。

## ルール

- メンバーファイルは `members/{名前}.md` に作成する
- スキルマップは `skill-map/_index.md` に一覧を管理する
- アサイン検討時はスキルと現在の負荷を参照して判断材料を提示する
- 引き継ぎ情報は `handover/` に蓄積する
- メンバー情報はここが SSoT。他部署からはリンクで参照する

## フォルダ構成

- `members/` - メンバー個人ファイル（SSoT）
- `skill-map/` - スキルマトリクス
- `handover/` - 引き継ぎ事項
```

### dx-group/_index.md

```markdown
# DXソリューショングループ

メンバー一覧と現在の負荷サマリー。

| 名前 | 拠点 | 役割 | 現在の担当PJ数 | 負荷 |
|------|------|------|--------------|------|
```

### dx-group/members/_template.md

```markdown
---
name: "{名前}"
site: unknown
role: "{役割}"
joined: YYYY-MM-DD
---

## スキル

### 得意

- {スキル1}

### 経験あり

- {スキル2}

### 育成中

- {スキル3}

## 現在の担当

- [{PJ名}](../../projects/active/{PJ-ID}_{PJ名}/_index.md)

## 備考

{特記事項}
```

### dx-group/skill-map/_index.md

```markdown
# スキルマップ

メンバー × スキルのマトリクス。◎=得意 / ○=経験あり / △=育成中 / −=未経験

| 名前 | Python | AWS | AI/ML | IoT | フロントエンド | DB | インフラ | 現在の負荷 |
|------|--------|-----|-------|-----|-------------|----|---------|---------:|
```

### dx-group/handover/_index.md

```markdown
# 引き継ぎ事項一覧

| 件名 | 引き継ぎ元 | 引き継ぎ先 | ステータス | 更新日 |
|------|----------|----------|----------|--------|
```

---

## 4. 未来設計活動（future-design）

### future-design/CLAUDE.md

```markdown
# 未来設計活動

## 役割

中長期の改善活動テーマ管理と成果記録を担当する。

## ルール

- テーマは `themes/{id}_{名称}.md` で管理する
- メンバーは `dx-group/members/` へリンクする（SSoT）
- テーマが PJ 化したら `projects/backlog/` に移動し、ここには「PJ-XXX に昇格」と記録する
- 成果は `outcomes/` に記録する

## フォルダ構成

- `themes/` - 活動テーマ
- `outcomes/` - 成果記録
```

### future-design/_index.md

```markdown
# 未来設計活動テーマ一覧

| ID | タイトル | ステータス | 拠点 | メンバー |
|----|---------|----------|------|---------|
```

### future-design/themes/_template.md

```markdown
---
id: "theme-XXX"
title: "{テーマタイトル}"
status: proposed  # proposed | active | completed | on-hold
site: unknown
started: YYYY-MM-DD
members:
  - "[{名前}](../../dx-group/members/{名前}.md)"
---

## 目的・背景

{なぜこのテーマに取り組むのか}

## 現状の課題

- {課題1}
- {課題2}

## アプローチ

{どうやって解決するか}

## 進捗

- YYYY-MM-DD: {進捗内容}

## 成果・学び

{完了時に記入}
```

### future-design/outcomes/_index.md

```markdown
# 成果記録一覧

| テーマID | タイトル | 成果概要 | 記録日 |
|---------|---------|---------|--------|
```

---

## 5. プロジェクト管理（projects）

### projects/CLAUDE.md

```markdown
# プロジェクト管理

## 役割

PJ の立ち上げから完了まで進捗管理を担当する。

## ルール

- PJ フォルダは `active/{PJ-ID}_{名称}/` に作成する
- PJ 概要は `_index.md` に記述する
- タスクは `tasks/_index.md` で表管理する
- リスク・課題は `risks/_index.md` で表管理する
- 成果物・展開先は `deliverables/` 配下に管理する
- ステータス: `backlog` → `active` → `on-hold` → `completed` → `archive`
- PJ 固有の障害は `risks/` に、PJ 横断の知見は `tech-knowledge/` に記録する
- archive 移動は `status: completed` かつ最終 deliverable 承認後に提案する

## 新規 PJ 作成時のフォルダ構成

active/{PJ-ID}_{名称}/
├── _index.md          ← PJ概要
├── tasks/
│   └── _index.md      ← タスク表
├── risks/
│   └── _index.md      ← リスク・課題表
└── deliverables/
    ├── _index.md      ← 成果物一覧
    └── deploy-map.md  ← 展開先マップ

## フォルダ構成

- `backlog/` - 未着手PJ
- `active/` - 進行中PJ
- `archive/` - 完了済みPJ
```

### projects/_index.md

```markdown
# プロジェクト一覧

| PJ-ID | 名称 | ステータス | 拠点 | 開始日 | 目標日 | オーナー |
|-------|------|----------|------|--------|--------|---------|
```

### projects/active/{PJ}/_index.md テンプレート

```markdown
---
id: "PJ-XXX"
title: "{PJ名}"
status: active  # backlog | active | on-hold | completed | archive
site: unknown
started: YYYY-MM-DD
target: YYYY-MM-DD
owner: "{オーナー名}"
members:
  - "[{名前}](../../../dx-group/members/{名前}.md)"
stakeholders:
  - "[{名前}](../../../collaboration/stakeholders/{部門}_{名前}.md)"
---

## 目的

{PJ の目的}

## スコープ

- {スコープ1}
- {スコープ2}

## 技術構成

- 設計書: [{設計書名}](../../../tech-knowledge/designs/{設計書ファイル}.md)

## マイルストーン

- [ ] {マイルストーン1}（YYYY-MM-DD）
- [ ] {マイルストーン2}（YYYY-MM-DD）
- [ ] {マイルストーン3}（YYYY-MM-DD）
```

### projects/active/{PJ}/tasks/_index.md テンプレート

```markdown
# タスク一覧

| タスク | 担当 | 状態 | 期限 | 更新日 |
|--------|------|------|------|--------|
```

### projects/active/{PJ}/risks/_index.md テンプレート

```markdown
# リスク・課題一覧

| リスク・課題 | 深刻度 | 状態 | 担当 | 関連 |
|------------|--------|------|------|------|
```

### projects/active/{PJ}/deliverables/_index.md テンプレート

```markdown
# 成果物一覧

| 成果物 | 種別 | 状態 | 承認者 | 更新日 |
|--------|------|------|--------|--------|
```

### projects/active/{PJ}/deliverables/deploy-map.md テンプレート

```markdown
# 展開先マップ

| 展開先 | 状態 | 導入日 | 備考 |
|--------|------|--------|------|
```

---

## 6. 連携・依頼管理（collaboration）

### collaboration/CLAUDE.md

```markdown
# 連携・依頼管理

## 役割

社内外の依頼追跡、ステークホルダー管理、判断・承認記録を担当する。

## ルール

- ステークホルダーは `stakeholders/{部門}_{名前}.md` に作成する（SSoT）
- 依頼は `requests/{日付}_task_{件名}.md` で管理する
- 判断記録は `decisions/{日付}_decision_{件名}.md` で管理する
- 依頼ステータス: `open` → `waiting` → `closed`
- ベンダー・外注も `stakeholders/` + `requests/` で管理する（タグで区別）

## フォルダ構成

- `stakeholders/` - ステークホルダー情報（SSoT）
- `requests/` - 依頼管理
- `decisions/` - 判断・承認記録
```

### collaboration/_index.md

```markdown
# 連携・依頼管理

未完了依頼サマリー。

| ID | 方向 | 相手 | 件名 | ステータス | 期限 |
|----|------|------|------|----------|------|
```

### collaboration/stakeholders/_index.md

```markdown
# ステークホルダー一覧

部門別キーマン。

| 名前 | 部門 | 拠点 | 役割 | 関連PJ |
|------|------|------|------|--------|
```

### collaboration/stakeholders/_template.md

```markdown
---
name: "{名前}"
department: "{部門}"
site: unknown
role: "{役割}"
contact: "{連絡手段}"
---

## 関わり方

{この人との関係性、連携の仕方}

## 依頼時の注意点

- {注意点1}

## 関連PJ

- [{PJ名}](../../projects/active/{PJ-ID}_{PJ名}/_index.md)

## やりとり履歴（直近）

- YYYY-MM-DD: {内容}
```

### collaboration/requests/_index.md

```markdown
# 依頼一覧

未完了の依頼。

| ID | 方向 | 相手 | 件名 | ステータス | 期限 |
|----|------|------|------|----------|------|
```

### collaboration/requests/_template.md

```markdown
---
id: "REQ-XXX"
direction: outgoing  # outgoing | incoming
from: "{依頼元}"
to: "{依頼先}"
status: open  # open | waiting | closed
site: unknown
created: YYYY-MM-DD
deadline: YYYY-MM-DD
related_pj: "[{PJ名}](../../projects/active/{PJ-ID}_{PJ名}/_index.md)"
---

## 依頼内容

{何を依頼するか / されたか}

## 経緯・背景

{なぜこの依頼が発生したか}

## 回答・結果

{回答が得られたら記入}

## 次のアクション

- [ ] {アクション}
```

### collaboration/decisions/_index.md

```markdown
# 判断記録一覧

| ID | タイトル | 決定者 | 決定日 | 関連PJ |
|----|---------|--------|--------|--------|
```

### collaboration/decisions/_template.md

```markdown
---
id: "DEC-XXX"
title: "{判断タイトル}"
decided_by: "{決定者}"
decided_at: YYYY-MM-DD
site: unknown
related_pj: "[{PJ名}](../../projects/active/{PJ-ID}_{PJ名}/_index.md)"
---

## 決定事項

{何を決めたか}

## 背景・理由

{なぜこの判断に至ったか}

## 代替案（検討したが却下）

- {代替案1}: {却下理由}

## 影響範囲

- {影響1}
```

---

## 7. 技術ナレッジ（tech-knowledge）

### tech-knowledge/CLAUDE.md

```markdown
# 技術ナレッジ

## 役割

PJ 横断の技術知見管理。設計書・検証ログ・障害対応ログ・標準化ドキュメントを担当する。

## ルール

- 「この PJ が終わっても価値がある情報」をここに置く
- PJ 固有の情報は PJ 配下が正本
- 設計書: 完成・再利用可能なものが正本
- 障害ログ: PJ 横断で学びがあるものが正本
- 年度末に `incidents/` から再発防止策を `standards/` に抽出する

## フォルダ構成

- `designs/` - 設計書（再利用可能なもの）
- `learnings/` - 検証ログ
- `incidents/` - 障害対応ログ
- `standards/` - 標準化・横展開ドキュメント
```

### tech-knowledge/_index.md

```markdown
# 技術ナレッジ

カテゴリ別ナレッジ一覧。

| カテゴリ | 件数 | 最終更新 |
|---------|------|---------|
| [設計書](designs/_index.md) | - | - |
| [検証ログ](learnings/_index.md) | - | - |
| [障害対応ログ](incidents/_index.md) | - | - |
| [標準化](standards/_index.md) | - | - |
```

### tech-knowledge/designs/_index.md

```markdown
# 設計書一覧

| タイトル | 関連PJ | 拠点 | 作成日 | 更新日 |
|---------|--------|------|--------|--------|
```

### tech-knowledge/designs/_template.md

```markdown
---
title: "{設計書タイトル}"
related_pj: "[{PJ名}](../../projects/active/{PJ-ID}_{PJ名}/_index.md)"
site: unknown
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

## 概要

{設計の概要}

## 技術構成

{使用技術・アーキテクチャ}

## 設計判断の経緯

{なぜこの設計にしたか}

## 既知の制約・注意点

- {制約1}
```

### tech-knowledge/learnings/_index.md

```markdown
# 検証ログ一覧

| タイトル | タグ | 拠点 | 作成日 |
|---------|------|------|--------|
```

### tech-knowledge/learnings/_template.md

```markdown
---
title: "{検証タイトル}"
tags:
  - "{タグ1}"
  - "{タグ2}"
site: unknown
created: YYYY-MM-DD
---

## 何を検証したか

{検証の目的と内容}

## 結果

{検証結果}

## 次のステップ

- [ ] {アクション}

## 参考リンク

- {リンク}
```

### tech-knowledge/incidents/_index.md

```markdown
# 障害対応ログ一覧

| タイトル | 深刻度 | 関連PJ | 拠点 | 発生日 | 解決日 |
|---------|--------|--------|------|--------|--------|
```

### tech-knowledge/incidents/_template.md

```markdown
---
title: "{障害タイトル}"
severity: medium  # high | medium | low
related_pj: "[{PJ名}](../../projects/active/{PJ-ID}_{PJ名}/_index.md)"
site: unknown
occurred: YYYY-MM-DD
resolved: YYYY-MM-DD
---

## 何が起きたか

{障害の概要}

## 影響範囲

- {影響1}

## 原因

{根本原因}

## 対処した内容

{実施した対処}

## 再発防止策

- [ ] {防止策}

## 学び

{この障害から得た教訓}
```

### tech-knowledge/standards/_index.md

```markdown
# 標準化・横展開チェック一覧

## 横展開判定基準チェックリスト

以下のうち2つ以上該当する場合、横展開候補とする。

- [ ] 同じ障害が別PJ・別拠点でも起こり得る
- [ ] 設計パターンとして再利用できる
- [ ] 手順書・チェックリストとして標準化できる
- [ ] 他チームにも有益な知見である
- [ ] コスト削減・品質向上に直結する

## 横展開候補一覧

| 元ナレッジ | 種別 | 横展開先 | ステータス | 更新日 |
|-----------|------|---------|----------|--------|
```

---

## 8. レビュー（reviews）

### reviews/CLAUDE.md

```markdown
# レビュー

## 役割

週次・月次レビュー生成と報告書ドラフト作成を担当する。

## ルール

- 週次は `weekly/{日付}_review_週次.md` で管理する
- 月次は `monthly/{年月}_review_月次.md` で管理する
- 上司向け最終版は `reports/` に保存する
- レビュー生成時は全部署の `_index.md` を走査して自動集約する

## フォルダ構成

- `weekly/` - 週次レビュー
- `monthly/` - 月次レビュー
- `reports/` - 上司向け報告書（最終版）
```

### reviews/_index.md

```markdown
# レビュー一覧

| 種別 | 期間 | 生成日 | ファイル |
|------|------|--------|---------|
```

### reviews/weekly/_template.md

```markdown
---
period: "YYYY-MM-DD 〜 YYYY-MM-DD"
site: both
generated: YYYY-MM-DD
---

## 今週の成果

{projects/active/ の各PJから今週更新されたタスクを自動集計}

| PJ | 完了タスク | 進行中タスク |
|----|----------|------------|

## 停滞アラート

{3日以上更新のないタスク・依頼を自動検出}

| 対象 | 最終更新 | 経過日数 | 担当 |
|------|---------|---------|------|

## 依頼追跡

{collaboration/requests/ の open|waiting を集約}

| ID | 方向 | 相手 | 件名 | ステータス | 期限 |
|----|------|------|------|----------|------|

## 拠点横断サマリー

{site: both のアイテムを集約}

- {サマリー}

## 来週の予定

{期限近いタスク + 定型業務}

- [ ] {予定1}

## リーダー振り返り

{手動記入欄}

- 今週の所感:
- 来週注力すべきこと:
```

### reviews/monthly/_template.md

```markdown
---
period: "YYYY-MM"
site: both
generated: YYYY-MM-DD
---

## PJ別進捗サマリー

{projects/active/ から集約}

| PJ-ID | PJ名 | ステータス | 今月の成果 | 課題 | 来月の予定 |
|-------|------|----------|-----------|------|----------|

## チーム状況

{dx-group/ から集約}

| 名前 | 担当PJ数 | 負荷 | 今月の成長・貢献 |
|------|---------|------|----------------|

## 横展開・標準化の進捗

{tech-knowledge/standards/ から集約}

| 元ナレッジ | 横展開先 | ステータス | 備考 |
|-----------|---------|----------|------|

## 報告書ドラフト

{上記を要約した上司向け文面}

### 今月の概況

{1段落の要約}

### 主な成果

1. {成果1}
2. {成果2}

### 課題・リスク

1. {課題1}

### 来月の計画

1. {計画1}
```

### reviews/reports/_index.md

```markdown
# 上司向け報告書一覧

| 期間 | 種別 | 提出日 | ファイル |
|------|------|--------|---------|
```
