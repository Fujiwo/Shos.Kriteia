# Kriteia比較総覧

本書は、[Kriteia言語設計詳細.md](Kriteia言語設計詳細.md) を中心に、既存の言語案である [Lattice言語設計詳細.md](Lattice言語設計詳細.md) 、 [Chord言語設計詳細.md](Chord言語設計詳細.md) 、 [Mosaic言語設計詳細.md](Mosaic言語設計詳細.md) 、 [Noesis.md](Noesis.md) 、 [追加のAIネイティブ言語アイディア.md](追加のAIネイティブ言語アイディア.md) 、 [追加の新プログラミング言語アイディア集.md](追加の新プログラミング言語アイディア集.md) に含まれる Chronicle、Helix、Strata と比較し、設計上の位置づけを整理する。

## 1. Kriteiaの立ち位置

Kriteia は、Noesis と Anchor を統合した AI ネイティブ言語案であり、単に「AI を呼び出しやすくする言語」ではない。設計の中心にあるのは次の一点である。

* 候補生成から探索、根拠収集、検証、採否判定までを一つの言語モデルへ統合すること

この性質により、Kriteia は AI 領域の中でも特に「高説明責任」「高誤りコスト」「内部統制付き運用」に強い。

## 2. 全体比較表

| 言語案 | 中核テーマ | 主な対象領域 | 実行モデル | 既存言語の主な痛点 | Kriteiaとの違い |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Kriteia** | 推論と根拠受理の統合 | AI エージェント、RAG、コード生成、レビュー | 意味型付き IR + evidence graph runtime | 推論、検証、採否の分散 | AI 成果物の受理までを一体化 |
| **Noesis** | 探索・分岐・制約付き推論 | AI エージェント、計画立案、確率的探索 | 非同期 JIT + choice point runtime | 探索がライブラリ依存 | 探索は強いが accept gate は弱い |
| **Anchor** | 根拠・評価・受理 | 高説明責任 RAG、レビュー基盤 | AI runtime + judge pipeline | 出力採否が後付け | 受理は強いが探索表現は弱い |
| **Lattice** | 安全性と性能の両立 | システム、ゲーム、低レイテンシ基盤 | AOT ネイティブ | 借用の学習コスト、GC 揺らぎ | AI よりメモリ/レイアウト問題が中心 |
| **Chord** | 会話順序の型付け | 分散サービス、ワークフロー | 軽量 VM + session runtime | 並行フローの破綻 | 会話安全性が中心で AI 特化ではない |
| **Mosaic** | 業務ルールの言語内蔵 | 業務システム、審査、料金計算 | AOT + incremental rule engine | if 文とルール散逸 | 推論は deterministic で AI 非依存 |
| **Chronicle** | 時間・因果・再実行 | イベント駆動、監査、ストリーム処理 | Event VM + causal replay | 非同期障害の再現困難 | AI ではなく因果追跡が中心 |
| **Helix** | 異種計算資源の統合 | HPC、ML 推論、画像処理 | Staged JIT + heterogeneous backend | CPU/GPU/NPU の分断 | AI モデル利用より計算配置が中心 |
| **Strata** | 型進化と永続互換性 | 長寿命 SaaS、業務基幹、データ基盤 | AOT + version-aware runtime | migration と互換性の分断 | AI ではなくスキーマ進化が中心 |

## 3. KriteiaとNoesisの比較

Noesis と Kriteia は最も近いが、設計焦点は明確に異なる。

| 観点 | Noesis | Kriteia |
| :---- | :---- | :---- |
| 中核問題 | どう探索するか | 何を通してよいか |
| 第一級概念 | prompt, branch, rollback | prompt, branch, evidence, judge, accept |
| 成果物の扱い | 制約付き候補 | 受理前候補と受理済み成果物を分離 |
| 強い場面 | 計画生成、試行錯誤、自己評価探索 | 高説明責任、本番運用、監査可能生成 |
| 弱い場面 | 採否基準の制度化 | 軽量な生成用途 |

一言で言えば、Noesis は「探索する AI 言語」であり、Kriteia は「探索し、かつ採用を統治する AI 言語」である。

## 4. KriteiaとAnchorの比較

Anchor は Kriteia に吸収されたが、思想上の違いは残る。

| 観点 | Anchor | Kriteia |
| :---- | :---- | :---- |
| 中核問題 | 根拠付き受理 | 探索から受理までの統合 |
| 候補生成 | 比較的単線的 | branch による多分岐探索 |
| ランタイム重点 | judge pipeline | choice point engine + judge pipeline |
| 実務適性 | 高い | 高い |
| 探索表現力 | 中程度 | 高い |

Anchor は実務向けガードレールとして優秀だが、Kriteia はそれに探索系の能力を統合した上位案である。

## 5. Kriteiaと他の非AI言語案の比較

## 5.1 Latticeとの比較

Lattice は「安全だが難しい Rust」と「簡単だが予測不能な GC 言語」の間を埋める案であり、対象はメモリ安全性と性能である。Kriteia は実行性能や所有権ではなく、AI 成果物の統治が中心である。両者は補完関係にあり、将来的には Kriteia のランタイムを Lattice で実装する、という組み合わせすら成立する。

## 5.2 Chordとの比較

Chord は「会話順序」を型にする。Kriteia は「受理されていない成果物を境界の外へ出さない」ことを型にする。前者は分散プロトコルの正しさ、後者は AI 出力の統治理性に焦点がある。

## 5.3 Mosaicとの比較

Mosaic は業務ルールの散逸を防ぐために deterministic なルールエンジンを内蔵する。Kriteia は不確実な生成を扱うため、候補と根拠と accept gate を持つ。もし両者を組み合わせるなら、Kriteia の policy evaluator の内部に Mosaic 的ルールブロックを使う構成が自然である。

## 5.4 Chronicleとの比較

Chronicle はイベント因果と replay に強い。Kriteia も checkpoint と trace を持つが、目的は AI 候補生成の統制であり、因果追跡そのものではない。Chronicle は「何が起きたか」を扱い、Kriteia は「何を採用するか」を扱う。

## 5.5 Helixとの比較

Helix は CPU、GPU、NPU を跨ぐ計算資源配置の言語であり、AI を含む高性能計算全般が対象である。Kriteia は AI を使うアプリケーションレベルの統制言語であり、モデル推論をどこで動かすかより、結果を通してよいかを重視する。

## 5.6 Strataとの比較

Strata は型進化と永続互換性を中心に据える。Kriteia も claimset や evidence schema の進化問題を将来的に抱えるため、Strata 的な型進化モデルと相性がよい。だが、設計の主眼は異なる。

## 6. Kriteiaが特に有望な理由

Kriteia を他案と比較したときの強みは、単に AI を扱うだけではなく、AI の不確実性を言語仕様として受け止め、その出口を制御している点にある。既存の AI フレームワークの多くは「どう呼ぶか」には強いが、「何を通してよいか」には弱い。Kriteia はこの空白に対して、型、ランタイム、ポリシー、評価を使って正面から答えている。

## 7. どの案をどこで選ぶべきか

* AI の探索と本番受理を一体化したいなら Kriteia。
* AI の探索制御だけを極めたいなら Noesis。
* 高説明責任の採否判定だけに集中したいなら Anchor。
* システム性能と安全性が主題なら Lattice。
* 分散プロトコルの正しさが主題なら Chord。
* 複雑な業務ルールの保守性が主題なら Mosaic。
* イベント因果と replay が主題なら Chronicle。
* 異種デバイス計算が主題なら Helix。
* 型進化と長寿命互換性が主題なら Strata。

## 8. まとめ

Kriteia は、このリポジトリ内の言語案群の中で、もっとも強く「AI を本番で安全に使う」ことへ焦点を当てた案である。Noesis の探索力と Anchor の根拠付き受理を統合した結果、Kriteia は単なる AI 向け DSL ではなく、候補生成から受理済み成果物までを型とランタイムで管理する統合言語になっている。

したがって Kriteia の独自価値は、「AI が何を言うか」ではなく、「AI の何を採用してよいか」を中心概念に据えた点にある。これは他の言語案にはない、AIネイティブ言語としての最も鮮明な識別点である。
