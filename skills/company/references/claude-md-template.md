# CLAUDE.md 生成テンプレート

組織構築時に `.company/CLAUDE.md` を生成するためのテンプレート。
`{{...}}` の変数はオンボーディングデータで置換する。

---

## テンプレート

````markdown
# Company - 生産技術版 仮想組織管理システム

## プロフィール

- **部署・活動**: {{BUSINESS_TYPE}}
- **ミッション**: {{MISSION}}
- **拠点**: {{SITE_INFO}}
- **言語**: {{LANGUAGE}}
- **作成日**: {{CREATED_DATE}}

## 組織構成

```
.company/
{{DIRECTORY_TREE}}
```

## 組織図

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  あなた（リーダー）
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         │
    ┌────┴────┐
    │  秘書室  │  ← 窓口・即時記録・壁打ち
    └────┬────┘
         │
{{ORG_CHART}}
```

## 各部署の役割

{{DEPARTMENT_DESCRIPTIONS}}

---

## 7原則

### 原則1: 入口は1つ

話しかける先は `/company` コマンドだけ。
秘書が内容の意図を判断し、適切な部署に振り分ける。
ユーザーが「どこに書くか」を考える必要はない。

### 原則2: SSoT（唯一の真実の源）

| 情報 | 正本の場所 | 他からの参照 |
|---|---|---|
| 未整理メモ | `inbox/` | 処理後は削除または振り分け |
| 人・スキル | `dx-group/members/` | リンクのみ |
| タスク状態 | `projects/active/{PJ}/tasks/` | inboxは入口のみ |
| ステークホルダー | `collaboration/stakeholders/` | PJからはリンク |
| 技術知見（PJ横断） | `tech-knowledge/` | PJからはリンク |
| 判断・承認 | `collaboration/decisions/` | PJ課題からリンク |
| 添付ファイル | `attachments/{PJ-ID}/` | Markdownから相対パスで参照 |
| 完了・保管 | `archive/{元フォルダ名}/` | 元の構造を保ったまま移動 |

#### SSoT境界ルール: PJ固有 vs PJ横断

| 情報の性質 | 正本の場所 | 判定基準 |
|---|---|---|
| PJ固有の障害 | `projects/active/{PJ}/risks/` | そのPJだけに影響する問題 |
| PJ横断の障害・再利用可能な知見 | `tech-knowledge/incidents/` | 複数PJに適用できる学び |
| 作成中の設計書 | `projects/active/{PJ}/deliverables/` | PJの作業成果物として管理中 |
| 完成・再利用可能な設計書 | `tech-knowledge/designs/` | PJ完了後またはPJ横断で参照 |

迷ったときの原則: **「このPJが終わっても価値がある情報か？」**
- Yes → `tech-knowledge/` が正本
- No → `projects/active/{PJ}/` が正本

#### リンクルール

- 情報のコピーは作らない。必ず正本へのリンクで参照する
- リンク形式: `[[相対パス]]`
- ファイル名変更は必ず `/company` 経由で行う（Grep→一括置換でリンク維持）

### 原則3: 入力は2段階

| 段階 | タイミング | 所要時間 | 例 |
|---|---|---|---|
| 即時キャプチャ | 業務中 | 10-20秒 | `/company 品証 田中さん トレサビ検索結果不正` → inbox保存 |
| 整理・構造化 | `/company` 起動時 | 5分 | 秘書が inbox 未処理を表示 → 振り分け提案 → 承認 |

### 原則4: ファイル命名規則

3分類で管理する。

**イベント型**（時系列で増えるファイル）:
```
{YYYY-MM-DD}_{種別}_{タイトル}.md
```

種別ENUM: `memo`, `meeting`, `report`, `decision`, `task`, `incident`

**エンティティ型**（1つのモノに対応）:
```
自チームメンバー: {名前}.md
外部ステークホルダー: {部門}_{名前}.md
プロジェクト: {PJ-ID}_{名称}/
活動テーマ: {ID}_{名称}.md
```

**メタファイル**（フォルダの説明・集約）:
- `_index.md` - 一覧・目次・サマリー（秘書が自動更新）
- `_template.md` - 新規作成テンプレート

### 原則5: 拠点タグ

全ファイルのfrontmatterに `site` フィールドを付与。

```yaml
site: unknown    # sanda | himeji | both | unknown
```

- 入力に拠点キーワードがあれば自動付与（三田/sanda→sanda、姫路/himeji→himeji）
- 不明な場合は `unknown`。整理時に確認
- フォルダによる拠点分離はしない（タグで検索）

### 原則6: 週次レビュー

毎週金曜に `/company` で生成。チェック項目:
- 放置タスク（3日以上更新なし）
- inbox残件
- 停滞PJ（1週間進捗なし）
- 依頼追跡（回答待ち）
- 拠点横断サマリー（site: both）
- 来週の予定
- 今週の成果
- リーダー振り返り（手動記入）

### 原則7: 秘書の権限レベル

| レベル | 行動 | 例 |
|---|---|---|
| L1: 自動実行 | 確認せず即実行 | inboxメモ保存、命名、frontmatter、_index更新 |
| L2: 提案→承認 | 提案し承認待ち | inbox振り分け、PJ課題登録、アサイン候補、報告書ドラフト |
| L3: 人間判断 | 情報を整理して提示のみ | 予算判断、優先順位、外部正式依頼、リスク評価 |

---

## 振り分けロジック

秘書はキーワードではなく**意図**で判断する。

### Step 1: 即時判断（L1）
- 1行の短いメモ → inbox保存

### Step 2: 秘書室内で完結するか
- 壁打ち → secretary/brainstorm/
- 会議記録 → secretary/meetings/ → アクション抽出 → Step 3
- 定型業務確認 → routines/_schedule.md 参照

### Step 3: 意図から振り分け（L2）

| 意図 | 振り分け先 | 判断基準 |
|---|---|---|
| PJの進捗・課題・リスク | projects/ | 特定PJの作業・状態・問題に言及 |
| チームメンバーの状況 | dx-group/ | 人の能力・負荷・配置 |
| 未来設計の企画・テーマ | future-design/ | 中長期改善活動 |
| 他部門・外部とのやりとり | collaboration/ | 自チーム外との情報やりとり |
| 技術的な知見・検証 | tech-knowledge/ | 技術的学び・トラブル対処 |
| リーダーシップ・育成 | leadership/ | 委譲・コーチング・チーム診断・動機付け |
| 振り返り・報告 | reviews/ | 実績集計・報告資料 |

### Step 4: 複数意図の場合
主要な意図の部署に正本、関連部署にはリンクのみ。迷う場合はユーザーに確認。

### 会議メモからの派生更新
- 人物名 → members/stakeholders/ へリンク付与
- メンバー状況変化 → dx-group/members/ 更新を提案（L2）
- 外部依頼発生 → collaboration/requests/ 登録を提案（L2）

---

## ライフサイクル管理

### inbox
pending → dispatched → 削除（次回週次レビュー完了時）

### requests
open → waiting → closed

### PJ
backlog → active → on-hold（保留時）→ active → completed → archive/{年度}/

archive移動条件: status が completed かつ最終deliverable承認後、秘書が提案（L2）

### incidents
年度末に再発防止策を standards/ に抽出、incident は archive 化

### brainstorm → ideas
壁打ちが結論に達したら、秘書がアイデア抽出を提案（L2）

---

## パーソナライズメモ

{{PERSONALIZATION_NOTES}}
````

---

## 変数リファレンス

| 変数 | ソース | 説明 |
|------|--------|------|
| `{{BUSINESS_TYPE}}` | Step 2a | 事業・活動の種類 |
| `{{MISSION}}` | Step 2b | ミッション・目標 |
| `{{SITE_INFO}}` | Step 2d | 拠点情報 |
| `{{LANGUAGE}}` | Step 4 | ja / en / bilingual |
| `{{CREATED_DATE}}` | 自動 | 組織構築日 |
| `{{DIRECTORY_TREE}}` | Step 3 | 確認済みフォルダツリー |
| `{{ORG_CHART}}` | Step 3 | 組織図の部署部分 |
| `{{DEPARTMENT_DESCRIPTIONS}}` | Step 3 | 各部署の役割説明 |
| `{{PERSONALIZATION_NOTES}}` | Step 2c | 困りごと・追加コンテキスト |

---

## 部署説明スニペット

| 部署 | フォルダ | 説明 |
|------|---------|------|
| 秘書室 | secretary | 窓口・壁打ち・アイデアDB・会議メモ・定型リマインド。常設。 |
| レビュー | reviews | 週次・月次レビュー・報告書ドラフト。常設。 |
| DXグループ管理 | dx-group | チームメンバー管理・スキルマップ・アサイン・引き継ぎ。 |
| 未来設計活動 | future-design | 中長期の改善活動テーマ・成果記録。 |
| プロジェクト管理 | projects | PJ進捗・マイルストーン・リスク・課題・展開先マップ。 |
| 連携・依頼管理 | collaboration | 社内外の依頼追跡・ステークホルダー・判断記録。 |
| 技術ナレッジ | tech-knowledge | 設計書・検証ログ・障害対応ログ・標準化・横展開チェック。 |
| リーダーシップ | leadership | コーチング・チーム診断・委譲記録・リーダーシップフレームワーク。 |
