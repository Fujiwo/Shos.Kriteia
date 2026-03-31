# Kriteia 仕様書 第11章 accept と受理ゲート

## この章の目的

`accept` の意味、`accepted<T>` への昇格条件、Acceptance Gate の最小責務、外部公開可能性の意味を、第1版に必要な粒度で定義する。Kriteia の中心命題である「何を通してよいか」を、型と実行モデルの両面から最小契約として固定する章とする。

## この章で確定する事項

- `accept` の役割
- Acceptance Gate の最小責務
- `draft<T>` から `accepted<T>` への昇格の骨子
- `accepted<T>` の第1版での形式
- `policy` と judge 結果の accept への接続
- 外部公開可能性の意味
- accept 不成立時の扱い
- 受理状態と業務境界の関係
- 第1版で扱わない受理詳細

## 一次資料

- [Kriteia言語設計詳細](../Documents/Kriteia言語設計詳細.md)
- [追加のAIネイティブ言語アイディア](../Documents/追加のAIネイティブ言語アイディア.md)
- [Kriteia比較総覧](../Documents/Kriteia比較総覧.md)
- [仕様書策定計画](../Plans/kriteia-specification-authoring-plan-2026-03-31.md)
- [用語集](./glossary.md)
- [open issues log](./open-issues-log.md)

## 本文草案

### 11.1 accept の位置づけ

`accept` は、候補成果物を受理済み成果物へ昇格させる唯一の操作である。Kriteia の他の構文が探索、根拠収集、検証を担うのに対し、`accept` は「業務境界を越えてよいか」を決める最後の操作である。したがって、`accept` は単なる return の別名ではなく、`accepted<T>` を生成できる唯一の正規経路として扱われる。

### 11.2 受理ゲート

Acceptance Gate は、`grounded draft candidate`、JudgeResult、named `policy`、`metric` を統合判定し、候補を `accepted artifact` へ昇格させてよいかを決める概念的境界である。第1版では、この 4 つを最小入力集合として固定し、内部モジュールとしての詳細は固定しない。JudgeResult については、少なくとも `status`、`violation_summary`、`grounding_gaps`、`metric_snapshot` の 4 フィールドを参照し、named `policy` ブロックの `require` 条件が満たされているかと合わせて判定する。ここで重要なのは、どれか一つが良好でも、他の条件が不足すれば受理不可になりうる点である。

### 11.3 accepted 型への昇格

型システム上は、`accept` は `draft<T>` を `accepted<T>` へ変換する唯一の正規経路として扱う。第1版では `accepted<T>` を opaque wrapper とみなし、通常の構築子やキャストからは生成できない型とする。これにより、どの値が受理済みであるかを型で区別でき、未受理の候補がそのまま外部 API 応答や自動アクションへ使われることを防げる。typestate、線形性、部分受理のような重い設計は将来版へ送る。

### 11.4 外部公開可能性

Kriteia における外部公開可能性とは、単にシリアライズ可能であることではなく、定義済みの `policy` 条件と judge 条件を満たし、正式に `accept` されたことを意味する。これは API 境界、外部システム呼び出し、自動修復適用、承認レポート出力などを含む広い概念であり、Kriteia はその境界を `accepted<T>` と Acceptance Gate で守ろうとする。

### 11.5 accept 不成立時の扱い

`accept` に至らない候補は、再探索、fallback、人手エスカレーションの対象になる。ここでの不成立は単なる例外ではなく、Kriteia の通常経路の一部である。judge が構造化失敗を返した場合も、Acceptance Gate が最終的に受理不可と判断した場合も、候補は `accepted<T>` にならない。この受理不可を明示的に扱えることが、Kriteia を軽量生成 DSL と分ける大きな違いである。

## 受理の原則

- `accept` なしに候補は正式成果物にならない。
- `accepted<T>` は `accept` のみが生成できる opaque wrapper として扱う。
- `accepted artifact` は通常候補と同一視しない。
- 外部境界を越えられるのは受理済み成果物に限る。

## Acceptance Gate が参照する最小情報

- `grounded draft candidate`: 主張と根拠が接続された受理直前候補
- `JudgeResult.status`: 受理可能かどうかの最終状態
- `JudgeResult.violation_summary`: 不受理理由の要約
- `JudgeResult.grounding_gaps`: 未接続根拠や不足箇所
- `JudgeResult.metric_snapshot`: 補助的な metric 値の要約
- named `policy`: `require` 条件列で記述された受理条件

## 第1版で扱わないもの

- partial accept の形式化
- 人手承認を型へ埋め込む詳細モデル
- Acceptance Gate の内部モジュール固定
- `accepted<T>` の typestate 化や線形性

## 未決論点

- 部分受理を認めるか。
- 人手承認を accept の一部として表現するか。

## 決定メモ

- 第1版の `accepted<T>` は `accept` のみが生成できる opaque wrapper とする。
- 第1版の受理判定は `grounded draft candidate`、JudgeResult、named `policy`、`metric` を最小入力集合として扱う。
- JudgeResult の最小フィールドは `status`、`violation_summary`、`grounding_gaps`、`metric_snapshot` とする。

## TODO

- 外部公開可能性の具体例を第14章と往復参照できる形で整理する。
- Acceptance Gate の最小入力集合と各入力の責務を一覧化する。
- partial accept を第16章の非スコープ記述へ寄せるか判断する。

## 完了条件

- `accept` と Acceptance Gate の役割が第1版の最小粒度で説明されている。
- `draft<T>` と `accepted<T>` の違いが受理条件と接続されている。
- `accepted<T>` の opaque wrapper としての意味が明確になっている。
- Acceptance Gate が参照する JudgeResult の最小情報が明示されている。
- Kriteia が「何を通してよいか」を中心に置く言語であることが、この章で再確認できる。
