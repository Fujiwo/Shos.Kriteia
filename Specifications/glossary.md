# Kriteia 用語集

## このファイルの目的

本ファイルは、Kriteia の仕様策定で用いる主要用語の唯一の参照元として使う。`Documents`、`Plans`、`Specifications` の間で用語が揺れた場合は、このファイルを基準に調整する。

## 運用ルール

- 新しい核概念を追加するときは、本文へ先に散らさず、このファイルへ定義を追加する。
- 既存用語の意味を変更するときは、decision log へ理由を残す。
- 訳語や英語表記が未確定な場合は `要確認` と明記する。

## 用語一覧

### prompt

- 英語表記: prompt
- 定義: 入力型、期待出力、制約候補を伴う生成契約
- 主な根拠資料: Documents/Kriteia言語設計詳細.md
- 関連用語: branch, semantic type
- 要確認: なし

### claimset

- 英語表記: claimset
- 定義: 候補成果物と、その中に含まれる主張集合を表す構造
- 主な根拠資料: Documents/Kriteia言語設計詳細.md
- 関連用語: evidence, judge
- 要確認: 値と型付き構造体のどちらを主と見るか

### evidence

- 英語表記: evidence
- 定義: 主張を支える根拠。文書断片、コード行、監査記録などを含む
- 主な根拠資料: Documents/Kriteia言語設計詳細.md, Documents/追加のAIネイティブ言語アイディア.md
- 関連用語: claimset, evidence graph
- 要確認: 標準バリアントの固定範囲

### branch

- 英語表記: branch
- 定義: 候補を複数展開し、評価と枝刈りを行う探索構文
- 主な根拠資料: Documents/Kriteia言語設計詳細.md, Documents/Noesis.md
- 関連用語: checkpoint, self-eval
- 要確認: 探索戦略を仕様で固定するか

### checkpoint

- 英語表記: checkpoint
- 定義: 実行状態を保存する地点
- 主な根拠資料: Documents/Kriteia言語設計詳細.md, Documents/Noesis.md
- 関連用語: rollback, fallback
- 要確認: 副作用の巻き戻し範囲

### judge

- 英語表記: judge
- 定義: 候補成果物に検証器や policy を適用し、構造化結果を返す段階
- 主な根拠資料: Documents/Kriteia言語設計詳細.md, Documents/追加のAIネイティブ言語アイディア.md
- 関連用語: policy, accept
- 要確認: 複数 judge の合成規則

### JudgeResult

- 英語表記: JudgeResult
- 定義: `judge` が返す構造化結果。第1版では `status`、`violation_summary`、`grounding_gaps`、`metric_snapshot` を持つ最小構造として扱う
- 主な根拠資料: Specifications/10_judgeと検証.md, Specifications/04_実行モデル.md
- 関連用語: judge, grounded draft candidate, accept
- 要確認: verifier ごとの詳細ログや改善提案を正文必須に含めるか

### grounded draft candidate

- 英語表記: grounded draft candidate
- 定義: Ground 段階を経て主張と `evidence` の対応付けを受けた draft 候補。Acceptance Gate に進める受理前候補状態を指す
- 主な根拠資料: Specifications/04_実行モデル.md, Specifications/08_探索と分岐.md, Specifications/11_acceptと受理ゲート.md
- 関連用語: draft 型, JudgeResult, accept
- 要確認: なし

### accept

- 英語表記: accept
- 定義: 候補成果物を受理済み成果物へ昇格させる唯一の操作
- 主な根拠資料: Documents/Kriteia言語設計詳細.md, Documents/追加のAIネイティブ言語アイディア.md
- 関連用語: accepted artifact, judge
- 要確認: fallback と人手承認をどこまで accept 周辺構文へ含めるか

### accepted artifact

- 英語表記: accepted artifact
- 定義: `accept` を通過した `accepted<T>` の値を、外部公開や後続処理の側から見た概念名
- 主な根拠資料: Documents/Kriteia言語設計詳細.md
- 関連用語: accept, accepted 型
- 要確認: なし

### semantic type

- 英語表記: semantic type
- 定義: 構造だけでなく意味情報や制約を保持する型
- 主な根拠資料: Documents/Kriteia言語設計詳細.md, Documents/Noesis.md
- 関連用語: prompt, policy
- 要確認: 訳語と形成規則

### draft 型

- 英語表記: draft type
- 定義: 生成済みだが未受理の候補を表す型
- 主な根拠資料: Documents/Kriteia言語設計詳細.md, Documents/追加のAIネイティブ言語アイディア.md
- 関連用語: accepted 型, judge
- 要確認: 線形性が必要か

### accepted 型

- 英語表記: accepted type
- 定義: `accept` のみが生成できる opaque wrapper としての受理済み型
- 主な根拠資料: Documents/Kriteia言語設計詳細.md
- 関連用語: draft 型, accept
- 要確認: 将来版で typestate や線形性を導入するか

### policy

- 英語表記: policy
- 定義: 名前付き `policy` ブロックと `require` 条件列で表される受理条件の束
- 主な根拠資料: Documents/Kriteia言語設計詳細.md, Documents/追加のAIネイティブ言語アイディア.md
- 関連用語: judge, metric
- 要確認: `require` 以外の論理合成や外部 evaluator 連携をどこまで標準化するか

### metric

- 英語表記: metric
- 定義: 候補や検証結果を評価する値または評価規則
- 主な根拠資料: Documents/Kriteia言語設計詳細.md
- 関連用語: policy, judge
- 要確認: 型制約へ統合するか
