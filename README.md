# seigi-company - 生産技術版 Company プラグイン

製造業の生産技術部門リーダー向けに最適化された、Claude Code 仮想会社組織プラグイン。

[cc-company](https://github.com/Shin-sibainu/cc-company) をベースに、製造業の生産技術業務に特化した部署構成・テンプレート・振り分けロジックを実装。

## 特徴

- **入口は1つ**: `/company` で秘書に話しかけるだけ。部署を意識する必要なし
- **意図ベースの振り分け**: キーワードではなく、内容の意図をAIが判断して適切な部署に振り分け
- **SSoT（唯一の真実の源）**: 情報の重複を防ぎ、リンクで参照する原則
- **週次/月次レビュー自動生成**: 全部署を横断して進捗を集約、報告書ドラフトまで自動生成
- **拠点タグ**: 複数拠点の業務を横串で管理

## 部署構成（7部署）

| 部署 | フォルダ | 役割 |
|------|---------|------|
| 秘書室 (常設) | `secretary/` | 窓口・壁打ち・アイデアDB・会議メモ |
| レビュー (常設) | `reviews/` | 週次/月次レビュー・報告書ドラフト |
| DXグループ管理 | `dx-group/` | チーム管理・スキルマップ・アサイン・引き継ぎ |
| 未来設計活動 | `future-design/` | 中長期改善テーマ・成果記録 |
| プロジェクト管理 | `projects/` | PJ進捗・リスク・課題・展開先マップ |
| 連携・依頼管理 | `collaboration/` | 依頼追跡・ステークホルダー・判断記録 |
| 技術ナレッジ | `tech-knowledge/` | 設計書・検証ログ・障害対応・標準化 |

## インストール

```bash
# マーケットプレースを追加
claude plugins marketplace add MasatoshiSano/seigi-company

# プラグインをインストール
claude plugins install seigi-company@seigi-company
```

## 使い方

```bash
# 初回: オンボーディング（ヒアリング → 部署選択 → フォルダ自動生成）
/company

# 2回目以降: 秘書が運営モードで起動
/company

# 使用例
/company 品証の田中さんからトレサビの件で連絡があった
/company 壁打ちしたい。稼働率の可視化について
/company 週次レビューして
/company 報告書のドラフト作って
/company ダッシュボード
```

## 7原則

1. **入口は1つ** - `/company` だけ
2. **SSoT** - 情報の正本は1箇所、リンクで参照
3. **2段階入力** - 即時キャプチャ(10-20秒) + 整理(5分)
4. **ファイル命名規則** - イベント型/エンティティ型/メタファイルの3分類
5. **拠点タグ** - `site: sanda | himeji | both | unknown`
6. **週次レビュー** - 放置タスク・停滞PJ・依頼追跡を自動検出
7. **秘書の権限レベル** - L1自動実行 / L2提案→承認 / L3人間判断

## カスタマイズ

このプラグインは製造業の生産技術部門を想定していますが、部署名やテンプレートは自由にカスタマイズできます。

- `skills/company/references/departments.md` - 部署テンプレート集
- `skills/company/references/claude-md-template.md` - CLAUDE.md生成テンプレート
- `skills/company/SKILL.md` - スキル定義・振り分けロジック

## ライセンス

MIT

## クレジット

- オリジナル: [cc-company](https://github.com/Shin-sibainu/cc-company) by [Shin-sibainu](https://github.com/Shin-sibainu)
