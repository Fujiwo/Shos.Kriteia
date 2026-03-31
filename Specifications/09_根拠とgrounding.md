# Kriteia 仕様書 第9章 根拠と grounding

## この章の目的

`claimset`、`evidence`、retrieval、grounding 条件、evidence graph の役割を定義し、候補成果物がどのように根拠付き候補へ進むかを明確にする。Anchor 由来の根拠中心設計を、Kriteia の探索モデルへ統合した章として位置づける。

## この章で確定する事項

- `claimset` と `evidence` の関係
- retrieval の役割
- grounding 条件の骨子
- evidence graph の役割
- Ground 段階の入出力
- 監査可能性との接続

## 一次資料

- [Kriteia言語設計詳細](../Documents/Kriteia言語設計詳細.md)
- [追加のAIネイティブ言語アイディア](../Documents/追加のAIネイティブ言語アイディア.md)
- [Kriteia比較総覧](../Documents/Kriteia比較総覧.md)
- [仕様書策定計画](../Plans/kriteia-specification-authoring-plan-2026-03-31.md)
- [用語集](./glossary.md)
- [open issues log](./open-issues-log.md)

## 本文草案

### 9.1 grounding の位置づけ

Kriteia における grounding は、候補に含まれる主張を外部根拠へ接続し、単なる生成物を検証可能な候補へ変える段階である。これは、検索の付加機能ではなく、受理可能性を成立させる前提条件として位置づけられる。

### 9.2 claimset と evidence

`claimset` は候補成果物と主張集合を保持し、`evidence` は各主張を支える根拠を表す。Kriteia の重要点は、成果物全体に一括で根拠を付けるのではなく、主張単位で根拠との対応を追跡するところにある。これにより、どの主張が十分に支えられ、どの主張が未接地なのかを区別できる。

### 9.3 retrieval の役割

retrieval は grounding の補助操作であり、文書断片、コード行、監査記録、検索結果などの根拠候補を収集する。Kriteia では retrieval そのものが受理を決めるわけではないが、根拠収集なしに高説明責任出力を成立させることはできない。そのため retrieval は Explore と Ground の境界にまたがる重要な機能になる。

### 9.4 evidence graph

evidence graph は、claim と evidence、引用元、検査結果を結ぶ構造である。Kriteia が監査可能性を持つのは、最終結果だけでなく、その結果がどの根拠に支えられ、どの judge を通ったかを辿れるからである。第1版では保存形式は未確定だが、少なくとも概念上はランタイムの正式成果物として扱う必要がある。

### 9.5 grounding 条件

grounding 条件は、「十分な根拠が存在する」とは何かを定める。第1版では完全な形式化より、少なくとも「重大な主張に evidence がない候補は accept に進めない」「引用整合性が崩れている候補は grounded とみなさない」という基準を明記する方向が妥当である。

## Ground 段階の最小責務

- claim と evidence の接続
- retrieval 結果の取り込み
- 引用整合性の確認
- judge 前に必要な根拠不足箇所の抽出

## 未決論点

- evidence の標準バリアントを固定するか。
- evidence graph の永続化形式を第1版へ含めるか。
- grounding を二値判定で表すか、段階的指標で表すか。

## 決定メモ

- 要確認

## TODO

- claim 単位と artifact 単位の grounding 条件を分けて整理する。
- evidence graph の概念図を追加する。
- retrieval の責務を stdlib と組み込みのどちらへ置くか整理する。

## 完了条件

- `claimset`、`evidence`、grounding、evidence graph の関係が説明されている。
- Ground 段階が単なる検索処理ではないことが明示されている。
- 第10章と第11章で judge、accept へ自然に接続できる。
