# GitHub × CLAUDE.md × SDD チーム開発ベストプラクティス (2026年3月版)

> 対象読者: Claude Code を使って複数人でプロダクト開発をしているチーム

---

## 目次

1. [SDD（スペック駆動開発）とは何か](#1-sddスペック駆動開発とは何か)
2. [CLAUDE.md の役割と育て方](#2-claudemd-の役割と育て方)
3. [リポジトリ構成のベストプラクティス](#3-リポジトリ構成のベストプラクティス)
4. [GitHub フローとの統合](#4-github-フローとの統合)
5. [CLAUDE.md をチームで育てるプロセス](#5-claudemd-をチームで育てるプロセス)
6. [具体例：新機能開発の一連の流れ](#6-具体例新機能開発の一連の流れ)
7. [よくある失敗パターンと対策](#7-よくある失敗パターンと対策)
8. [チェックリスト](#8-チェックリスト)

---

## 1. SDD（スペック駆動開発）とは何か

### 概念

**SDD (Spec-Driven Development)** とは、コードを書く前に「仕様書（スペック）」を構造化されたMarkdownで書き、その仕様書を元にAI（Claude）が実装を行う開発手法です。

```
従来の開発:  要件定義 → 実装 → テスト → ドキュメント（後付け）
SDD:        要件定義 → スペック文書化 → AI実装 → テスト → CLAUDE.md更新
```

### SDD の3層構造

```
┌─────────────────────────────────┐
│  CLAUDE.md                      │  ← プロジェクト全体の文脈・規約
│  (プロジェクト知識の蓄積)          │
├─────────────────────────────────┤
│  specs/                         │  ← 機能ごとの仕様書
│  (What を定義する)               │
├─────────────────────────────────┤
│  実装コード                       │  ← AI + 人間が協働で書く
│  (How を実現する)                 │
└─────────────────────────────────┘
```

### なぜ SDD が有効か（2026年時点）

- **Claude は長期記憶を持たない**: セッションをまたぐと文脈が消える。スペックに書いておけば毎回渡せる
- **チームで文脈を共有できる**: 新メンバーも CLAUDE.md + specs/ を読めば即戦力になる
- **レビューが容易**: 「仕様通りに実装されているか」を機械的にチェックできる
- **AI への指示が明確になる**: 曖昧な口頭指示より、構造化されたスペックの方が精度が上がる

---

## 2. CLAUDE.md の役割と育て方

### CLAUDE.md の階層構造

チームリポジトリでは CLAUDE.md を **3階層** で管理します。

```
プロジェクトルート/
├── CLAUDE.md                    ← Tier 1: プロジェクト全体（最重要）
├── src/
│   ├── CLAUDE.md                ← Tier 2: モジュール固有の規約
│   └── api/
│       └── CLAUDE.md            ← Tier 3: サブシステム固有（必要時のみ）
└── specs/
    └── CLAUDE.md                ← specs/ ディレクトリの読み方
```

Claude Code はファイルを開くと、そのパスから上位の CLAUDE.md を自動的に参照します。

### Tier 1: ルート CLAUDE.md に書くべきこと

```markdown
# プロジェクト名

## このプロジェクトについて
- 何を作っているか（1〜3行）
- 技術スタック（言語・FW・主要ライブラリ）
- 現在のフェーズ（MVP / 成長期 / 安定期）

## 開発規約
- コーディングスタイル（Linterの設定場所を示す）
- 命名規則（例: コンポーネントは PascalCase）
- ファイル構成ルール

## AI への作業指示
- やってほしいこと / やってほしくないこと
- 実装前に必ず読むファイル（例: specs/architecture.md）
- 使ってよいライブラリ / 禁止ライブラリ

## チーム固有の知識
- ドメイン用語集
- 過去の重要な決定事項（ADR: Architecture Decision Records）
- 既知のトリッキーな箇所と注意点
```

### CLAUDE.md に書いてはいけないこと

| 書くべきでないもの | 理由 | 代替 |
|---|---|---|
| 実装の詳細なコード | すぐ陳腐化する | コード自体にコメント |
| TODO リスト | GitHub Issues で管理 | Issues へのリンクだけ書く |
| 個人の設定 | リポジトリ共通ではない | `~/.claude/CLAUDE.md` に書く |
| 秘密情報 | Git に残る | .env や Secrets Manager |

---

## 3. リポジトリ構成のベストプラクティス

### 推奨ディレクトリ構成

```
プロジェクトルート/
├── CLAUDE.md                    ← AI への全体指示書
├── specs/                       ← SDD の核心
│   ├── CLAUDE.md                ← specs ディレクトリの読み方
│   ├── architecture.md          ← システム全体設計
│   ├── features/
│   │   ├── auth.md              ← 認証機能の仕様
│   │   ├── dashboard.md         ← ダッシュボードの仕様
│   │   └── notifications.md
│   └── api/
│       └── openapi.yaml         ← API 仕様（自動生成の元）
├── docs/
│   ├── adr/                     ← Architecture Decision Records
│   │   ├── 001-use-postgres.md
│   │   └── 002-adopt-graphql.md
│   └── onboarding.md
├── src/
│   └── ...
└── tests/
    └── ...
```

### specs/ の書き方

各機能スペックは以下のテンプレートで書きます:

```markdown
# 機能名: ○○○

## ステータス
- **状態**: draft | review | approved | implemented
- **作成日**: YYYY-MM-DD
- **最終更新**: YYYY-MM-DD
- **担当**: @username

## 概要
この機能が解決する問題と、何を実現するかを2〜3行で。

## ユーザーストーリー
- ○○なユーザーが、△△したいとき、□□できる

## 受け入れ条件（Acceptance Criteria）
- [ ] ○○ができる
- [ ] △△のとき □□ というエラーが出る
- [ ] パフォーマンス: ○○ms 以内にレスポンスする

## データモデル
```typescript
interface Example {
  id: string;
  // ...
}
```

## API エンドポイント
- `POST /api/v1/examples` - 作成
- `GET /api/v1/examples/:id` - 取得

## 除外スコープ（やらないこと）
- ○○は対象外（理由: ）

## 関連
- 依存する機能: [auth.md](./auth.md)
- GitHub Issue: #123
```

---

## 4. GitHub フローとの統合

### SDD を取り入れた GitHub フロー

```
Issue 作成
  ↓
specs/ に仕様書を書く（PR: spec/feature-name）
  ↓
仕様レビュー（PRレビュー）← ここでチームが合意
  ↓
実装ブランチ作成（feature/feature-name）
  ↓
Claude Code で実装（スペックを読ませながら）
  ↓
実装 PR + テスト
  ↓
マージ後 CLAUDE.md を更新
```

### ブランチ命名規則

```
spec/feature-name        ← 仕様書の追加・更新
feature/feature-name     ← 機能実装
fix/bug-description      ← バグ修正
docs/update-claude-md    ← CLAUDE.md の更新
refactor/module-name     ← リファクタリング
claude/task-description  ← Claude Code が自律的に作業するブランチ
```

### PR テンプレート（`.github/pull_request_template.md`）

```markdown
## 概要
<!-- 何をしたか、なぜしたか -->

## スペック
<!-- 対応する specs/ のファイルへのリンク -->
- [ ] specs/features/○○.md に記載の受け入れ条件をすべて満たしている

## CLAUDE.md への影響
<!-- この変更でチームの知識として記録すべきことがあるか -->
- [ ] CLAUDE.md の更新が必要 → `docs/update-claude-md` ブランチで対応済み
- [ ] CLAUDE.md の更新は不要

## テスト
- [ ] 単体テスト追加
- [ ] 統合テスト追加
- [ ] 手動確認済み

## スクリーンショット / 動作確認
```

### Issue テンプレート（`.github/ISSUE_TEMPLATE/feature.md`）

```markdown
---
name: 新機能
about: 新機能の提案・開発
---

## 概要
何を実現したいか

## 背景・目的
なぜ必要か

## スペック作成
- [ ] `specs/features/○○.md` を作成してこの Issue にリンクする

## 受け入れ条件
- [ ] ○○ができる

## 関連 Issue / PR
```

---

## 5. CLAUDE.md をチームで育てるプロセス

### 「CLAUDE.md 更新のトリガー」を決める

CLAUDE.md は「都度更新」では陳腐化します。以下のタイミングを**ルール化**します:

| タイミング | 更新内容 |
|---|---|
| 新しいライブラリを追加したとき | 使い方・使うべき場所を記載 |
| バグが再発したとき | 注意点・禁止パターンを追記 |
| 設計判断をしたとき | ADR を作成し、CLAUDE.md からリンク |
| スプリント終了時 | チームで「CLAUDE.md に追加すべき知見はないか」をレトロで確認 |
| 新メンバー参加時 | 「CLAUDE.md を読んで迷った点」を本人に聞いて改善 |

### CLAUDE.md レビューの仕組み

```
スプリント終了
  ↓
チーム全員が「今週 Claude に何度も説明したこと」を持ち寄る
  ↓
CLAUDE.md に追記 → PR → 全員レビュー
  ↓
マージ → 次スプリントから効果が出る
```

**具体例**: スプリントレトロでの質問リスト

```
1. Claude に同じことを2回以上説明しましたか？ → CLAUDE.md に書く
2. Claude が誤った方向に進んだ場面は？ → 禁止事項に追記
3. 「これはうちの会社独自のルール」と説明したことは？ → ドメイン用語集に追記
4. 新メンバーが理解に時間がかかった設計は？ → 解説を追記
```

### CLAUDE.md の品質指標

良い CLAUDE.md:
- **具体的**: 「きれいに書く」ではなく「ESLint の `airbnb` ルールに従う」
- **理由付き**: 「○○禁止（理由: △△の問題が起きたため。ADR-003参照）」
- **メンテナブル**: 実装詳細ではなく、変わりにくい原則と判断基準を書く
- **簡潔**: 長すぎると Claude が読まなくなる（目安: 200行以内）

---

## 6. 具体例：新機能開発の一連の流れ

### シナリオ: 「通知機能を追加する」

#### Step 1: Issue 作成

```
タイトル: feat: ユーザーへのメール通知機能
本文:
  背景: ユーザーがタスク期限を忘れることが多い
  要件: タスク期限の1日前にメール通知する
  スペック: specs/features/notifications.md を作成する
```

#### Step 2: スペック PR を出す

`spec/notifications` ブランチを作り、`specs/features/notifications.md` を作成:

```markdown
# 機能名: メール通知

## ステータス
- 状態: review
- 担当: @tanaka

## 概要
タスクの期限1日前に、担当ユーザーへメール通知を送る。

## 受け入れ条件
- [ ] 期限24時間前に通知メールが届く
- [ ] 通知設定をOFF にできる
- [ ] メールの件名・本文のフォーマットは designs/email-templates.md に従う

## 技術仕様
- 送信ライブラリ: nodemailer（既存）
- スケジューラ: node-cron（新規追加）
- テンプレートエンジン: handlebars

## 除外スコープ
- Slack通知は対象外（別 Issue #456 で対応）
```

チームがこの PR をレビューし、「受け入れ条件」に合意したらマージ。

#### Step 3: 実装ブランチで Claude Code を使う

`feature/notifications` ブランチで作業開始。Claude への指示:

```
specs/features/notifications.md を読んで、通知機能を実装してください。

実装前に CLAUDE.md と specs/architecture.md も確認してください。
受け入れ条件をすべて満たす実装と、対応するテストを書いてください。
```

Claude Code はスペックを読んで:
1. `src/services/NotificationService.ts` を実装
2. `src/jobs/notificationCron.ts` を実装
3. `tests/services/NotificationService.test.ts` を生成

#### Step 4: 実装 PR のレビュー

PR の説明に以下を記載:

```markdown
## スペック
- [x] specs/features/notifications.md の受け入れ条件をすべて満たしている

## CLAUDE.md への影響
- [x] node-cron の使い方を CLAUDE.md に追記する必要がある
  → docs/update-claude-md ブランチで対応済み (#PR番号)
```

#### Step 5: マージ後 CLAUDE.md を更新

```markdown
# CLAUDE.md への追記例

## スケジューラ
- バックグラウンドジョブには `node-cron` を使う（nodemailer と組み合わせ）
- cron 設定ファイルは `src/jobs/` に置く
- 開発環境では cron を無効化する: `DISABLE_CRON=true` を .env に設定
- 注意: cron は複数インスタンスで重複実行されないよう、
  将来的には Redis Lock が必要（ADR-004 参照）
```

---

## 7. よくある失敗パターンと対策

### 失敗1: CLAUDE.md が「誰も読まないドキュメント」になる

**症状**: CLAUDE.md が長文化し、Claude が指示を無視し始める

**対策**:
- 200行を超えたら分割を検討（Tier 2 の CLAUDE.md に移す）
- 「今 Claude に何が必要か」だけを書く。網羅的にしようとしない
- 定期的に「不要な行」を削除するリファクタリングPRを出す

### 失敗2: スペックを書かずにいきなり実装する

**症状**: Claude が「良さそうな実装」をするが、チームの意図と違う

**対策**:
- 「スペック PR が承認されるまで実装 PR は出せない」をルール化
- GitHub の Branch Protection Rules で強制する

### 失敗3: CLAUDE.md の更新が個人任せになる

**症状**: 特定の人しか更新せず、他の人の知見が溜まらない

**対策**:
- スプリントレトロに「CLAUDE.md 更新」を固定議題に入れる
- CLAUDE.md の更新 PR には全員のレビュー承認を必須にする
- 「毎週金曜の CLAUDE.md 更新当番」をローテーションで決める

### 失敗4: specs/ と実装が乖離する

**症状**: スペックを書いたが実装時に変更され、スペックが古くなる

**対策**:
- 実装 PR では必ず「スペックを最新化したか」をチェックリストに入れる
- スペックと実装の乖離を発見したら、スペックを先に直してから実装を修正

### 失敗5: チームメンバーそれぞれが違う CLAUDE.md を使う

**症状**: 個人の `~/.claude/CLAUDE.md` にチーム共通設定を書いてしまう

**対策**:
- チーム共通 → リポジトリの `CLAUDE.md`（Git 管理）
- 個人設定 → `~/.claude/CLAUDE.md`（Git 管理外）
- 何をどこに書くかをオンボーディングドキュメントに明記する

---

## 8. チェックリスト

### 新プロジェクト開始時

- [ ] `CLAUDE.md` をルートに作成（技術スタック・規約・AI指示を記載）
- [ ] `specs/` ディレクトリと `specs/architecture.md` を作成
- [ ] `specs/CLAUDE.md` に specs の読み方を記載
- [ ] `.github/pull_request_template.md` に CLAUDE.md 更新チェックを追加
- [ ] `.github/ISSUE_TEMPLATE/` にスペック作成チェックを追加
- [ ] ブランチ命名規則をチームで合意する

### スプリント中（毎 PR）

- [ ] 機能追加の場合、対応する `specs/features/*.md` が存在するか
- [ ] 受け入れ条件をすべて満たしているか
- [ ] CLAUDE.md への追記が必要な知見はないか

### スプリント終了時（レトロ）

- [ ] 「Claude に2回以上説明したこと」はないか → CLAUDE.md に追記
- [ ] 「禁止すべき実装パターン」は発見されなかったか → CLAUDE.md に追記
- [ ] 新しい ADR が必要な設計判断はなかったか
- [ ] CLAUDE.md が 200行を超えていないか → 分割を検討

### 新メンバー参加時

- [ ] `CLAUDE.md` + `specs/architecture.md` を最初に読んでもらう
- [ ] 「理解しにくかった点」のフィードバックを集める
- [ ] フィードバックを CLAUDE.md に反映する

---

## 参考: CLAUDE.md の育ちかたのイメージ

```
Week 1: 技術スタック・規約のみ（50行）
  ↓
Week 4: バグ対策の注意事項・ドメイン用語を追記（80行）
  ↓
Month 2: 設計判断のADRリンクが増える（100行）
  ↓
Month 3: 分割が必要になり Tier 2 CLAUDE.md が生まれる
  ↓
Month 6: specs/ が充実し、新メンバーが即日から戦力になる
```

---

*最終更新: 2026-03-29 | このドキュメントは `/company` スキルと連携して使用してください*
