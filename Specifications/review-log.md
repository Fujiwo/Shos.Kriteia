# Kriteia Review Log

## このファイルの目的

レビュー実施日、対象、主な指摘、反映状況を記録する。設計レビューと仕様レビューの履歴を追跡可能にし、同じ指摘が繰り返されることを防ぐ。

## 運用ルール

- 各レビューごとに 1 エントリ追加する。
- findings の件数と、反映済みか未反映かを残す。
- 未反映の場合は理由または保留先を記載する。

## 記録テンプレート

### エントリテンプレート

- ID:
- 日付:
- 対象:
- レビュー種別:
- 主な指摘:
- 件数:
- 状態:
- 次アクション:

### RV-001

- 日付: 2026-03-31
- 対象: Plans/kriteia-specification-authoring-plan-2026-03-31.md
- レビュー種別: 計画レビュー
- 主な指摘: フェーズゲート、想定読者、decision log / review log の常設管理が不足
- 件数: 3
- 状態: reflected
- 次アクション: 仕様章ドラフトとログ雛形へ反映する

### RV-002

- 日付: 2026-03-31
- 対象: Plans/kriteia-specification-simplification-plan.md, Specifications/04_実行モデル.md, Specifications/05_型システム.md, Specifications/06_文法と構文.md, Specifications/07_promptと生成.md, Specifications/08_探索と分岐.md, Specifications/10_judgeと検証.md, Specifications/13_標準ライブラリ.md, Specifications/16_将来拡張と非スコープ.md
- レビュー種別: 簡素化進捗レビュー
- 主な指摘: フェーズ1の簡素化対象棚卸し、フェーズ2の最小コア凍結、フェーズ3の章別削減方針、フェーズ5の decision log / open issues 更新、フェーズ6の本文反映は進行済み。残る重点は `branch` の正文粒度、`policy` 形式、accepted 型の形式、将来版送り項目の最終整理
- 件数: 4
- 状態: in-progress
- 次アクション: 未解決論点を潰しつつ、残章と比較して簡素化粒度をそろえる
