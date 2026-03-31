---
name: kriteia-design-review
description: 'Review Kriteia design and specification files for consistency, terminology, inheritance boundaries, and specification gaps. Use for レビューして, 設計レビュー, 仕様レビュー, 矛盾確認, consistency review.'
argument-hint: 'レビュー対象ファイルやレビュー観点があれば入力してください'
---

# Kriteia Design Review

## When to Use
- `Documents`、`Plans`、`Specifications` の設計内容をレビューするとき
- Noesis、Anchor、Kriteia の継承境界を確認するとき
- 仕様の抜けや矛盾、用語の揺れを見つけたいとき

## Review Priorities
1. 継承関係の整合性
2. 用語の一貫性
3. 章責務からの逸脱
4. 未決論点の隠蔽
5. 実装可能性に関わる仕様欠落

## Procedure
1. 対象ファイルと、それを支える一次資料を確認する。
2. Kriteia の核概念と章責務に照らして矛盾や不足を探す。
3. findings は重要度順に並べ、ファイルと具体箇所を示す。
4. 勝手に修正案へ飛ばず、まず問題点を明確にする。
5. 修正提案を出す場合は、何を守るための提案かを短く添える。

## Output Expectations
- findings 優先
- 要約は短く
- 根拠のない断定はしない

## Avoid
- 賞賛中心のレビュー
- 一般論だけの指摘
- Noesis と Anchor の違いを無視した評価
