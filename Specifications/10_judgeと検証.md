# Kriteia 仕様書 第10章 judge と検証

## この章の目的

`judge` の構文、複数検証器の最小合成、`policy` と `metric` の評価、judge 結果の最小構造を、第1版に必要な粒度で定義する。Judge Pipeline のような内部構成は概念的役割に留め、Kriteia における検証を単なる真偽値判定ではない受理前の構造化判断として固定する。

## この章で確定する事項

- `judge` の役割
- judge 構文の最小骨子
- 複数 `using` の最小合成方針
- JudgeResult の最小構造
- `policy` と `metric` の最小接続
- Judge Pipeline の概念的役割
- 第1版で扱わない検証詳細

## 一次資料

- [Kriteia言語設計詳細](../Documents/Kriteia言語設計詳細.md)
- [追加のAIネイティブ言語アイディア](../Documents/追加のAIネイティブ言語アイディア.md)
- [仕様書策定計画](../Plans/kriteia-specification-authoring-plan-2026-03-31.md)
- [用語集](./glossary.md)
- [open issues log](./open-issues-log.md)
- [decision log](./decision-log.md)

## 本文草案

### 10.1 judge の位置づけ

`judge` は候補成果物に対して検証器、整合性検査、policy 判定を適用し、構造化結果を返す段階である。Kriteia では、探索が良好でも、grounding が一定程度進んでいても、それだけでは候補は採用されない。judge は、その候補が accept に進む条件を満たしているかを判断する公式の検証境界であり、第1版でも外せない中核である。

### 10.2 judge 構文

既存資料では、`judge candidate using CitationVerifier using ConsistencyCheck using PolicyChecker(ReleaseGate)` のような構文例が示されている。ここから読み取れるのは、judge が単一検査だけでなく、複数の検証器を並べて適用できる構文だという点である。第1版では、`using` 列が左から右へ評価される最小読解規則だけを仮置きし、短絡評価や高度な合成最適化は後ろへ送る方が簡素である。`PolicyChecker` が受け取る `ReleaseGate` のような `policy` は、名前付き `policy` ブロックに `require` 条件を並べたものとして解釈する。

### 10.3 JudgeResult

judge の結果は単なる bool ではなく、失敗理由、根拠不足箇所、改善提案、評価値を含む構造化値として扱う必要がある。これにより、judge は accept 前の関門であるだけでなく、再探索、fallback、人手エスカレーションの入力にもなる。第1版では JudgeResult を「pass/fail と詳細情報を持つ最小構造」として扱い、完全な集約理論や豊富な付加情報までは正文で固定しない。

### 10.4 policy と metric

`policy` は「通してよいか」を決める受理条件に近く、`metric` は候補や検証結果の良し悪しを評価する補助指標に近い。judge はこの両者をまとめて扱うが、同一視はしない。第1版では `policy` を名前付き `policy` ブロックと `require` 条件列として扱えれば十分であり、`metric` はその結果に補助情報を与える値として扱う。`policy` の不成立は runtime exception ではなく、JudgeResult の失敗詳細として表現する。

### 10.5 Judge Pipeline

Judge Pipeline は、どの検証器をどの順で適用し、どの結果を最終判断へ渡すかを支える概念的なランタイム要素である。Kriteia の強みは、検証が外部フレームワークの後付けではなく、言語ライフサイクルの正式要素として組み込まれている点にある。第1版では、Judge Pipeline を内部モジュールとして細かく凍結せず、複数検証器を順に適用できるという概念的役割だけを残す。

## 検証の最小分類案

- 引用整合性検証
- 構造整合性検証
- policy 適合性検証
- metric 評価
- 根拠不足検出

## 第1版で扱わないもの

- JudgeResult の完全な集約理論
- 複数 judge の高度な最適化戦略
- 検証器カタログの完全列挙
- Judge Pipeline の内部モジュール固定

## 未決論点

- `using` の適用順序が結果に影響するか。
- JudgeResult に総合スコアを必須にするか。
- 複数 judge の失敗を短絡評価するか集約するか。

## 決定メモ

- 第1版では judge を構造化検証境界として維持する。
- JudgeResult は pass/fail と詳細情報を持つ最小構造として扱う。
- `policy` は `require` 条件列を持つ名前付きブロックとして judge から参照する。

## TODO

- JudgeResult の最小フィールド案を作る。
- `policy` 違反と grounding 不足の報告形式を切り分ける。
- judge 失敗時の fallback 接続を第12章と整合させる。

## 完了条件

- `judge` の構文と役割が説明されている。
- JudgeResult が bool ではなく構造化結果であることが明確になっている。
- `policy`、`metric`、Judge Pipeline の関係が最小粒度で整理されている。
