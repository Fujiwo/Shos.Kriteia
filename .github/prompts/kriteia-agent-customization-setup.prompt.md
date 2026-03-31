---
name: "Kriteia Agent Customization Setup"
description: "Kriteia リポジトリ向けに .github/skills と .github/copilot-instructions.md を計画し、必要なファイル作成まで実行する。Agent Skills、copilot-instructions、workspace customization、repo-specific Copilot setup に使う。"
argument-hint: "追加したい Skill 候補、除外したい対象、優先したい作業領域があれば入力してください"
agent: "agent"
model: "GPT-5.4 (copilot)"
---

あなたのタスクは、このリポジトリ向けの Copilot カスタマイズを計画し、必要なファイル作成まで完了することです。計画だけで終わらせず、実際に workspace 内のファイルを作成または更新してください。

今回の対象は次の 2 系統です。
- `.github/copilot-instructions.md`
- `.github/skills/<skill-name>/SKILL.md` を中心とした Agent Skills 群

このタスクでは、AGENTS.md は作成しないこと。workspace-wide instructions は `.github/copilot-instructions.md` に統一すること。

最初に確認すること:
- `.github` 配下の既存ファイルと既存構成
- `Documents` 配下の主要資料
- `Plans` 配下の既存計画
- `Specifications` 配下の既存骨子

最低限参照する資料:
- [Kriteia 仕様策定計画](../../Plans/kriteia-specification-authoring-plan-2026-03-31.md)
- [第1章 導入と位置づけ](../../Specifications/01_導入と位置づけ.md)
- [第2章 設計目標と問題設定](../../Specifications/02_設計目標と問題設定.md)
- [第3章 コア概念](../../Specifications/03_コア概念.md)
- [第4章 実行モデル](../../Specifications/04_実行モデル.md)
- [第5章 型システム](../../Specifications/05_型システム.md)
- [Kriteia言語設計詳細](../../Documents/Kriteia言語設計詳細.md)
- [Kriteia比較総覧](../../Documents/Kriteia比較総覧.md)

必要に応じて参照してよい資料:
- [Noesis](../../Documents/Noesis.md)
- [追加のAIネイティブ言語アイディア](../../Documents/追加のAIネイティブ言語アイディア.md)
- [新プログラミング言語設計アイディア集](../../Documents/新プログラミング言語設計アイディア集.md)

実行順序:
1. 現状を調査し、常時適用ルールとオンデマンド Skill の境界を決める。
2. `Plans` フォルダーに、今回のカスタマイズ作業用の計画書を新規 Markdown で作成する。
3. `.github/copilot-instructions.md` を新規作成または更新する。
4. `.github/skills` 配下に必要最小限の Skill フォルダー群を作成し、それぞれに `SKILL.md` を作成する。
5. 必要なら `references`、`assets`、`scripts` サブディレクトリを作る。ただし本当に必要な場合だけに限定する。
6. 作成後、ファイル配置と frontmatter の整合性を確認し、何を作成したか要約する。

実行要件:
1. 計画書は `Plans` フォルダー内に新規 Markdown ファイルとして保存すること。
2. 既定の計画書名は `agent-customization-setup-plan.md` とすること。
3. 同名ファイルが存在する場合は `agent-customization-setup-plan-YYYY-MM-DD.md` の形式にし、それも存在する場合は末尾に `-NN` を付けて連番化すること。
4. 出力言語は日本語とすること。
5. `copilot-instructions.md` には全作業に共通して効く最小限のルールだけを書くこと。
6. Skill に書くべき内容と `copilot-instructions.md` に書くべき内容を混同しないこと。
7. 汎用的すぎる Skill は避け、このリポジトリ固有の作業に効く Skill を優先すること。
8. 既存資料に根拠がない内容は断定せず、`要確認` として明示すること。
9. Skill の `name` はフォルダー名と一致させること。
10. Skill の `description` には、用途とトリガー語を明示すること。
11. `copilot-instructions.md` と `AGENTS.md` を併用しないこと。

計画書に必ず含める見出し:
- 概要
- 現状認識
- copilot-instructions.md に入れるべき内容
- Skill 化すべきテーマ一覧
- Skill 候補詳細
- 推奨ディレクトリ構成
- 実装フェーズ
- フェーズごとの成果物
- 依存関係
- レビュー方針
- 完了条件
- リスクと対策
- 要確認事項

Skill 候補詳細では、各 Skill ごとに次の項目を必ず同じ順序で書くこと:
- 名前
- 目的
- 対象タスク
- 想定トリガー
- 含めるべき手順
- 必要な補助ファイル
- `copilot-instructions.md` に書かず Skill 側へ分離する理由

`copilot-instructions.md` の制約:
- 全作業に共通して効くルールだけを書くこと。
- 長文化しすぎないこと。
- 既存文書の要約ではなく、エージェントの行動指針に限定すること。
- 参照すべき資料、仕様策定時の優先順位、レビュー観点、ファイル配置方針などの横断ルールを優先すること。

Skill 設計の制約:
- 1 Skill = 1 つの明確な反復可能ワークフローに寄せること。
- `description` には具体的なキーワードを入れ、曖昧な説明を避けること。
- `SKILL.md` は肥大化させず、必要な場合だけ `./references/...` を参照すること。
- `user-invocable` や `disable-model-invocation` は必要な場合だけ設定すること。
- `scripts` や `assets` は空のまま作らないこと。本当に必要なときだけ作成すること。

実装方針:
- まず最小構成で始めること。
- `.github/copilot-instructions.md` は薄く保つこと。
- 初回導入では Skill 数を絞ること。
- Kriteia の仕様策定、言語設計レビュー、概念整理、資料横断参照など、このリポジトリに繰り返し現れる作業を優先すること。
- `Documents`、`Plans`、`Specifications` の役割の違いを反映すること。

出力テンプレート:
最初に `Plans` に保存する計画書を作成し、その後で実装を行うこと。計画書本文は次の構造を保つこと。

```md
# Agent Customization Setup Plan

## 概要
- 

## 現状認識
- 

## copilot-instructions.md に入れるべき内容
- 

## Skill 化すべきテーマ一覧
- 

## Skill 候補詳細
### Skill 1
- 名前:
- 目的:
- 対象タスク:
- 想定トリガー:
- 含めるべき手順:
- 必要な補助ファイル:
- `copilot-instructions.md` に書かず Skill 側へ分離する理由:

## 推奨ディレクトリ構成
```text
.github/
```

## 実装フェーズ
### フェーズ1
- 

## フェーズごとの成果物
- 

## 依存関係
- 

## レビュー方針
- 

## 完了条件
- 

## リスクと対策
| リスク | 影響 | 対策 |
|---|---|---|
|  |  |  |

## 要確認事項
- 
```

実装時に作成または更新するファイルについて:
- `.github/copilot-instructions.md`
- `.github/skills/<skill-name>/SKILL.md`
- 必要な場合のみ `.github/skills/<skill-name>/references/*`
- 必要な場合のみ `.github/skills/<skill-name>/assets/*`
- 必要な場合のみ `.github/skills/<skill-name>/scripts/*`

最後に行うこと:
- 作成または更新したファイル一覧を示す。
- 計画書ファイル名を示す。
- 作成した Skill の数と名前を示す。
- `copilot-instructions.md` に入れたルールの要点を 5 項目以内で要約する。
- 追加で確認すべき重大論点があれば、最後に `要確認` として短く添える。