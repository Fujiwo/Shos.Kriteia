# Kriteia Decision Log

## このファイルの目的

仕様策定中に行った判断、その採用理由、棄却理由、影響範囲を記録する。本文から判断経緯が消えることを避けるため、章本文に埋め込まずここへ集約する。

## 運用ルール

- 仕様上の判断を新たに行ったら、同日に 1 行追加する。
- 保留は `deferred`、採用は `accepted`、棄却は `rejected` とする。
- 影響する章またはファイルを必ず書く。

## 記録テンプレート

### エントリテンプレート

- ID:
- 日付:
- 論点:
- 判断:
- 状態:
- 理由:
- 影響ファイル:
- 次アクション:

### DL-001

- 日付: 2026-03-31
- 論点: `Documents` / `Plans` / `Specifications` の役割分離
- 判断: `Documents` は一次資料、`Plans` は計画、`Specifications` は仕様ドラフトとして扱う
- 状態: accepted
- 理由: repo 全体で文書の役割を固定し、執筆対象と根拠資料を分離するため
- 影響ファイル: Plans/kriteia-specification-authoring-plan-2026-03-31.md, Specifications/*
- 次アクション: instructions と各章へ反映済み

### DL-002

- 日付: 2026-03-31
- 論点: 用語の言い換え方針
- 判断: `prompt`、`claimset`、`evidence`、`branch`、`judge`、`accept`、`accepted artifact` は勝手に別語へ置換しない
- 状態: accepted
- 理由: Kriteia の核概念を設計横断で安定させるため
- 影響ファイル: .github/copilot-instructions.md, Specifications/glossary.md
- 次アクション: glossary へ継続反映

### DL-003

- 日付: 2026-03-31
- 論点: 第1版仕様簡素化の中心線
- 判断: 第1版の Kriteia は「高説明責任 AI 出力のための最小統治言語」として整理し、`prompt`、`claimset`、`evidence`、`judge`、`accept`、draft / accepted の区別を中核に残す
- 状態: accepted
- 理由: Kriteia の独自性を保ちつつ、仕様の同時確定対象を減らすため
- 影響ファイル: Plans/kriteia-specification-simplification-plan.md, Specifications/05_型システム.md, Specifications/08_探索と分岐.md, Specifications/13_標準ライブラリ.md
- 次アクション: 対象章の本文と未決論点へ反映する

### DL-004

- 日付: 2026-03-31
- 論点: ランタイム内部要素の正文での扱い
- 判断: Meaning-Typed IR、Evidence Graph IR、Choice Point Engine、Judge Pipeline、Acceptance Gate は第1版正文では概念的役割に留め、内部構成や最適化は参考記述または将来拡張へ回す
- 状態: accepted
- 理由: 中心契約と実装自由度を分離し、仕様の肥大化を抑えるため
- 影響ファイル: Plans/kriteia-specification-simplification-plan.md, Specifications/04_実行モデル.md, Specifications/08_探索と分岐.md, Specifications/13_標準ライブラリ.md, Specifications/16_将来拡張と非スコープ.md
- 次アクション: 実装寄りの詳細は本文から薄め、追跡が必要なものは open issues へ残す

### DL-005

- 日付: 2026-03-31
- 論点: 第1版標準ライブラリのスコープ
- 判断: 第1版の stdlib は最小 API 境界のみを正文に残し、vendor 固有設定、完全な Retriever / Verifier / Evaluator カタログ、監査ストレージ標準化は非スコープとする
- 状態: accepted
- 理由: 第1版で必要な言語境界を示しつつ、ドメイン依存の詳細で仕様を肥大化させないため
- 影響ファイル: Plans/kriteia-specification-simplification-plan.md, Specifications/13_標準ライブラリ.md, Specifications/16_将来拡張と非スコープ.md
- 次アクション: stdlib 章は境界定義中心へ簡素化する

### DL-006

- 日付: 2026-03-31
- 論点: `semantic type` の第1版での定義
- 判断: 第1版の `semantic type` は独立した重い型カテゴリとしては扱わず、`prompt`、期待出力、受理条件に結び付く軽量な意味注釈層として扱う
- 状態: accepted
- 理由: Kriteia の意味契約を残しつつ、型理論の同時確定対象を減らすため
- 影響ファイル: Specifications/05_型システム.md, Specifications/07_promptと生成.md, Specifications/16_将来拡張と非スコープ.md
- 次アクション: effect type や高度な型形成規則は将来版候補として扱う

### DL-007

- 日付: 2026-03-31
- 論点: `think` の位置づけ
- 判断: 第1版では `think` を言語組み込みではなく、候補生成の標準入口となる最小 stdlib 関数として扱う
- 状態: accepted
- 理由: 生成入口を固定しつつ、言語構文の増加を避けて本体仕様を簡素に保つため
- 影響ファイル: Specifications/07_promptと生成.md, Specifications/13_標準ライブラリ.md, Specifications/16_将来拡張と非スコープ.md
- 次アクション: 将来版では生成 API の拡張余地のみを残す

### DL-008

- 日付: 2026-03-31
- 論点: `policy` の第1版での形式
- 判断: 第1版の `policy` は、名前付き `policy` ブロックの中に `require` 条件を列挙する最小 DSL として扱い、既存の一般式へ埋め込む形や高度な制約言語化は採らない
- 状態: accepted
- 理由: 受理条件を業務ロジックから分離して明示でき、judge / accept 境界の説明責任を最小仕様のまま保てるため
- 影響ファイル: Specifications/05_型システム.md, Specifications/10_judgeと検証.md, Specifications/11_acceptと受理ゲート.md, Specifications/12_エラーモデル.md
- 次アクション: `policy` 構文の完全形式化は将来版へ送り、第1版では `require` 列の最小読解規則だけを維持する

### DL-009

- 日付: 2026-03-31
- 論点: `accepted` 型の第1版での形式
- 判断: 第1版の `accepted<T>` は `accept` のみが生成できる opaque wrapper として扱い、通常の構築子やキャストでは生成できない受理済み型とする
- 状態: accepted
- 理由: 受理境界を最小コストで強制でき、typestate や線形型を導入せずに「何を通してよいか」を型で表現できるため
- 影響ファイル: Specifications/05_型システム.md, Specifications/10_judgeと検証.md, Specifications/11_acceptと受理ゲート.md, Specifications/12_エラーモデル.md
- 次アクション: typestate、線形性、部分受理は将来版候補として非スコープ側で管理する

### DL-010

- 日付: 2026-03-31
- 論点: Acceptance Gate の第1版入力集合
- 判断: 第1版の Acceptance Gate は `grounded draft candidate`、JudgeResult、named `policy`、`metric` を最小入力集合として扱う
- 状態: accepted
- 理由: 04章の実行モデルと 11章の受理モデルで同じ境界を共有し、accept 判定の責務を過不足なく固定するため
- 影響ファイル: Specifications/04_実行モデル.md, Specifications/11_acceptと受理ゲート.md
- 次アクション: judge 実行と Acceptance Gate 判定の順序はこの入力集合を前提に記述を揃える

### DL-011

- 日付: 2026-03-31
- 論点: `accepted artifact` と `accepted<T>` の関係
- 判断: 第1版では `accepted artifact` を独立した第三の型とはせず、`accept` を通過した `accepted<T>` の値を業務境界側から見た概念名として扱う
- 状態: accepted
- 理由: 概念章と型章の用語を分離しすぎず、受理済み値の意味論と外部公開可能性を同じ境界で説明できるため
- 影響ファイル: Specifications/03_コア概念.md, Specifications/glossary.md, Specifications/05_型システム.md, Specifications/11_acceptと受理ゲート.md
- 次アクション: 概念説明では `accepted artifact`、型説明では `accepted<T>` を使い分ける

### DL-012

- 日付: 2026-03-31
- 論点: `accept` の第1版文法上の位置づけ
- 判断: 第1版の `accept` は `accept <judge-result-or-candidate>` の形を取る文として扱い、値を返す式としては許可しない
- 状態: accepted
- 理由: 一次資料の `accept judged` 例と実行モデル上の最終段階という役割に最も整合し、`accepted<T>` の opaque wrapper 方針とも衝突しないため
- 影響ファイル: Specifications/03_コア概念.md, Specifications/glossary.md, Specifications/06_文法と構文.md, Specifications/11_acceptと受理ゲート.md
- 次アクション: fallback 側の値返却や人手エスカレーションは `accept` とは別経路として記述を整理する

### DL-013

- 日付: 2026-03-31
- 論点: JudgeResult の第1版最小フィールド
- 判断: 第1版の JudgeResult は `status`、`violation_summary`、`grounding_gaps`、`metric_snapshot` の 4 フィールドを最小集合として扱う
- 状態: accepted
- 理由: Acceptance Gate が pass / fail 判定、主要な不受理理由、根拠不足、metric の補助値を過不足なく受け取れる最小構造になるため
- 影響ファイル: Specifications/10_judgeと検証.md, Specifications/11_acceptと受理ゲート.md, Specifications/glossary.md, Specifications/open-issues-log.md
- 次アクション: 改善提案や verifier ごとの詳細ログは第1版では付随詳細として扱い、完全な集約理論は将来版へ送る
