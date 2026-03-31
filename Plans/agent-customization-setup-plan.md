# Agent Customization Setup Plan

## 概要
- Kriteia リポジトリ向けに、常時適用ルールとオンデマンド Skill の境界を明確化する。
- `.github/copilot-instructions.md` は薄い横断ルールに限定し、反復的な作業は `.github/skills` へ分離する。
- 初回導入では Skill 数を 3 個に絞り、仕様策定、設計レビュー、未決論点管理の反復作業を対象にする。
- `Documents`、`Plans`、`Specifications` の役割差を instructions と Skill の両方へ反映する。

## 現状認識
- `.github` 配下には prompt 群のみが存在し、workspace-wide instructions と Agent Skills は未整備である。
- リポジトリの中心作業は、Kriteia の仕様策定、概念整理、継承関係の比較、未決論点の管理である。
- `Documents` は一次資料、`Plans` は実行計画、`Specifications` は仕様ドラフトの役割を持つ。
- 常時ルールを厚くしすぎると context を圧迫するため、汎用ルールは最小限に抑える必要がある。

## copilot-instructions.md に入れるべき内容
- `Documents`、`Plans`、`Specifications` の役割の違い
- Kriteia の核概念 `prompt`、`claimset`、`evidence`、`branch`、`judge`、`accept` を勝手に言い換えないこと
- 資料間に矛盾があれば黙って解消せず、`要確認` または open issues として残すこと
- 仕様策定では、先に計画書と骨子を見てから本文を書くこと
- repo 固有の反復作業は Skill を使い、instructions に手順を埋め込みすぎないこと

## Skill 化すべきテーマ一覧
- 仕様章の下書き作成
- 設計・仕様レビュー
- 未決論点、decision log、review log の整理

## Skill 候補詳細
### Skill 1
- 名前: `kriteia-spec-drafting`
- 目的: `Documents`、`Plans`、`Specifications` を横断して、Kriteia の仕様章や節を一貫した形で下書きする
- 対象タスク: 章本文の執筆、章骨子の展開、節の追記、定義文の整備
- 想定トリガー: `仕様書を書く`、`章本文を書く`、`spec drafting`、`仕様章を埋める`、`章骨子から本文化`
- 含めるべき手順: 関連資料確認、計画と既存章の確認、継承内容と未決論点の分離、本文作成、要確認事項の明示
- 必要な補助ファイル: なし
- `copilot-instructions.md` に書かず Skill 側へ分離する理由: 章執筆は全作業に共通しない反復ワークフローであり、常時読み込むには重い

### Skill 2
- 名前: `kriteia-design-review`
- 目的: Kriteia の仕様や設計文書を、継承関係、用語、設計トレードオフ、仕様整合性の観点でレビューする
- 対象タスク: 仕様レビュー、設計レビュー、比較レビュー、概念整合性確認
- 想定トリガー: `レビューして`、`設計レビュー`、`仕様レビュー`、`矛盾確認`、`consistency review`
- 含めるべき手順: 対象文書確認、一次資料への照合、用語の揺れ確認、継承境界の確認、 findings 優先順での報告
- 必要な補助ファイル: なし
- `copilot-instructions.md` に書かず Skill 側へ分離する理由: レビューは特定タスク時のみ必要で、判断手順を常時読み込むと instructions が肥大化する

### Skill 3
- 名前: `kriteia-open-issues-tracking`
- 目的: 未決論点、decision log、review log を更新し、仕様策定の保留事項を構造化して維持する
- 対象タスク: 要確認事項の集約、open issues 更新、判断理由の記録、レビュー指摘の反映管理
- 想定トリガー: `要確認を整理`、`open issues を更新`、`decision log`、`review log`、`未決論点をまとめる`
- 含めるべき手順: 対象文書から論点抽出、既存ログとの重複確認、優先度付け、記録先の更新、次アクションの明記
- 必要な補助ファイル: なし
- `copilot-instructions.md` に書かず Skill 側へ分離する理由: 論点管理は仕様策定タスクに付随する個別ワークフローであり、常時ルールよりも手順依存が強い

## 推奨ディレクトリ構成
```text
.github/
  copilot-instructions.md
  prompts/
  skills/
    kriteia-spec-drafting/
      SKILL.md
    kriteia-design-review/
      SKILL.md
    kriteia-open-issues-tracking/
      SKILL.md
```

## 実装フェーズ
### フェーズ1
- `.github` の現状確認と役割分担の決定

### フェーズ2
- `.github/copilot-instructions.md` の作成

### フェーズ3
- `.github/skills` 配下の Skill 作成

### フェーズ4
- frontmatter、命名、責務分離の確認

## フェーズごとの成果物
- フェーズ1: この計画書
- フェーズ2: `.github/copilot-instructions.md`
- フェーズ3: 各 Skill の `SKILL.md`
- フェーズ4: 作成ファイル一覧と検証結果

## 依存関係
- instructions と Skill の境界を決める前に、Skill 内容を確定しない
- repo 固有の反復作業を特定した後に Skill 名と説明を決める
- instructions を薄く保つ方針を先に固定する

## レビュー方針
- `copilot-instructions.md` が全作業共通ルールだけに留まっているか確認する
- Skill の `description` が具体的なトリガー語を持っているか確認する
- Skill の粒度が 1 ワークフロー 1 Skill になっているか確認する
- 既存資料にない内容を断定していないか確認する

## 完了条件
- `Plans` に今回の計画書が保存されている
- `.github/copilot-instructions.md` が作成されている
- `.github/skills` 配下に 3 Skill が作成されている
- `name` とフォルダー名が一致し、frontmatter に重大な不整合がない
- instructions と Skill の責務分離が説明可能である

## リスクと対策
| リスク | 影響 | 対策 |
|---|---|---|
| instructions に手順を書きすぎる | 常時読み込みが重くなる | 横断ルールだけに限定し、手順は Skill へ寄せる |
| Skill を細かく分けすぎる | 発見しづらく、保守しづらい | 初回導入は 3 Skill に限定する |
| description が曖昧 | 自動発見されにくい | 用途とトリガー語を明示する |
| 既存資料にない前提を混ぜる | repo との整合性が崩れる | `要確認` として残し、断定を避ける |

## 要確認事項
- glossary、decision log、open issues log、review log を今後どこへ置くか
- `kriteia-open-issues-tracking` を初回導入から使うか、後続で追加するか
