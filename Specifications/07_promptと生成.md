# Kriteia 仕様書 第7章 prompt と生成

## この章の目的

`prompt` を意味型付き生成契約として定義し、入力型、出力型、制約候補、materialization の扱いを明確にする。第1版では `semantic type` を軽量な意味注釈層として扱い、`think` は言語組み込みではなく候補生成の標準入口となる最小 stdlib 関数として位置づける。

## この章で確定する事項

- `prompt` の正式な役割
- 入力型と出力型の関係
- Meaning-Typed IR との概念的接続
- 制約候補の位置づけ
- `think<T>` と `prompt` の関係
- materialization の扱い

## 一次資料

- [Kriteia言語設計詳細](../Documents/Kriteia言語設計詳細.md)
- [Noesis](../Documents/Noesis.md)
- [Kriteia比較総覧](../Documents/Kriteia比較総覧.md)
- [仕様書策定計画](../Plans/kriteia-specification-authoring-plan-2026-03-31.md)
- [用語集](./glossary.md)
- [open issues log](./open-issues-log.md)

## 本文草案

### 7.1 prompt の役割

Kriteia における `prompt` は、単なる文字列テンプレートではなく、入力型、期待出力、制約候補を伴う意味型付き生成契約である。これにより、開発者は「何をモデルへ送るか」だけでなく、「どの型の候補を得たいか」「後続の検証や受理とどう接続するか」を同じ構文上で扱える。

### 7.2 入力型と出力型

`prompt` は入力型と出力型を持つ。入力型は生成コンテキストを、出力型は候補成果物の形を表す。ここでの出力型は最終受理済み成果物を直接意味するのではなく、少なくとも初期状態では draft 候補の期待形を表す。したがって `prompt` の戻り型と `accept` 後の公開可能型は区別して考える必要がある。

### 7.3 Meaning-Typed IR への変換

Kriteia のコンパイラは `prompt` 宣言を Meaning-Typed IR へ落とし込むと考えられるが、第1版ではこれを内部実装の完全仕様として固定しない。ここで重要なのは、`prompt` が単なる文字列ではなく、入力型、期待出力、制約候補、関連 `policy` といった意味情報を共有する入口であることである。第1版の `semantic type` も、この意味契約を支える軽量注釈層として理解する。

### 7.4 制約候補と materialization

生成時の制約候補は、Kriteia では `prompt` に隣接する概念として扱う。これは、Noesis 由来の制約生成とも接続し、Explore 段階で候補空間を狭める役割を持つ。また、生成された候補をどの時点で具体的な `claimset` 値として materialize するかは重要であり、第1版では最小 stdlib 関数である `think<T>` の結果を最初の materialized candidate とみなす。

### 7.5 文字列テンプレートとの差分

通常のテンプレート文字列や既存の prompt ライブラリとの最大の差は、`prompt` が単独で完結せず、`branch`、`judge`、`accept` と同じ仕様空間に属する点にある。Kriteia では、prompt は「モデルへ投げる文字列」ではなく、「候補生成の意味契約」であり、後続段階の入力として型と制約を保持したまま扱われる。

## prompt に関する構成要素案

- 入力コンテキスト
- 期待出力型
- 制約候補
- 関連 policy
- materialization 時点

## 未決論点

- `prompt` が暗黙に draft 型を返すとみなすか。
- materialization を明示演算として持つか。
- prompt に policy 参照を直接埋め込めるようにするか。

## 決定メモ

- 第1版では `semantic type` を軽量な意味注釈層として扱う。
- 第1版では `think` を言語組み込みではなく最小 stdlib 関数として扱う。

## TODO

- `prompt` 宣言の最小構文を第6章と整合する形で補強する。
- Meaning-Typed IR に保持する属性を概念レベルで一覧化する。
- `think<T>`、`prompt`、draft 型の関係を図示する。

## 完了条件

- `prompt` が文字列テンプレートではなく意味型付き契約であることが説明されている。
- 入力型、出力型、制約候補、materialization の関係が整理されている。
- 第8章以降で `prompt` を前提に探索と検証の説明へ進める状態になっている。
