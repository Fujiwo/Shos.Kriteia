# Kriteia Open Issues Log

## このファイルの目的

章本文や計画書に散在する `要確認`、未決論点、将来版送り候補を一元管理する。本文中に論点を埋めたまま放置しないための追跡先として使う。

## 運用ルール

- 新しい未決論点は本文だけでなく必ずこのファイルへ追加する。
- 優先度は `高`、`中`、`低` とする。
- 将来版送りにした場合は理由を明記する。

## 記録テンプレート

### エントリテンプレート

- ID:
- 論点:
- 概要:
- 優先度:
- 関連ファイル:
- 必要な判断:
- 状態:

### OI-001

- 論点: branch の評価戦略
- 概要: width、score、枝刈り条件を仕様でどこまで固定するか未決
- 優先度: 高
- 関連ファイル: Specifications/04_実行モデル.md, Specifications/08_探索と分岐.md
- 必要な判断: 第1版で固定する最小探索意味論と、将来版へ回す探索詳細の境界を決める
- 状態: open

### OI-002

- 論点: policy の形式化
- 概要: 第1版では `policy` を名前付き `policy` ブロック内に `require` 条件を列挙する最小 DSL として扱い、一般式への埋め込みや高度な制約理論は将来版候補とする
- 優先度: 高
- 関連ファイル: Specifications/05_型システム.md, Specifications/10_judgeと検証.md, Specifications/11_acceptと受理ゲート.md, Specifications/12_エラーモデル.md
- 必要な判断: 将来版で `require` 以外の論理合成、外部 evaluator 連携、停止性制約をどこまで formalize するかを追跡する
- 状態: resolved

### OI-003

- 論点: accepted 型の形式
- 概要: 第1版では `accepted<T>` を `accept` のみが生成できる opaque wrapper として扱い、typestate や線形性は将来版候補とする
- 優先度: 高
- 関連ファイル: Specifications/03_コア概念.md, Specifications/05_型システム.md, Specifications/10_judgeと検証.md, Specifications/11_acceptと受理ゲート.md, Specifications/12_エラーモデル.md
- 必要な判断: 将来版で typestate、部分受理、線形性を導入するかを追跡する
- 状態: resolved

### OI-004

- 論点: evidence graph の永続化
- 概要: 実行時保存形式と監査ログとの関係が未決
- 優先度: 中
- 関連ファイル: Specifications/03_コア概念.md, 後続の grounding 章
- 必要な判断: 第1版では概念説明に留めるか、保存形式まで含めるかを決める
- 状態: open

### OI-005

- 論点: semantic type の最小定義
- 概要: 第1版では `semantic type` を軽量な意味注釈層として扱い、独立した重い型カテゴリとしての形式化は将来版候補とする
- 優先度: 高
- 関連ファイル: Specifications/05_型システム.md, Specifications/07_promptと生成.md, Specifications/16_将来拡張と非スコープ.md
- 必要な判断: 将来版へ送る型理論要素の範囲を維持管理する
- 状態: resolved

### OI-006

- 論点: `think` の位置づけ
- 概要: 第1版では `think` を候補生成の標準入口となる最小 stdlib 関数として扱い、言語組み込みにはしない
- 優先度: 高
- 関連ファイル: Specifications/07_promptと生成.md, Specifications/13_標準ライブラリ.md
- 必要な判断: 将来版で生成 API をどこまで拡張するかを追跡する
- 状態: resolved

### OI-007

- 論点: 四段階実行モデルの正文粒度
- 概要: Plan、Explore、Ground、Accept の四段階を正文でそのまま維持するか、生成、grounding、judge、accept へ圧縮して書くか未決
- 優先度: 中
- 関連ファイル: Specifications/04_実行モデル.md, Specifications/08_探索と分岐.md, Specifications/11_acceptと受理ゲート.md
- 必要な判断: 第1版で読者理解と簡素性のバランスが最もよい記述粒度を決める
- 状態: open

### OI-008

- 論点: 高度探索の将来版送り範囲
- 概要: dynamic width、複雑な score 型、並列 branch の順序保証、nested checkpoint の詳細を第1版からどこまで外すか未決
- 優先度: 中
- 関連ファイル: Specifications/08_探索と分岐.md, Specifications/16_将来拡張と非スコープ.md
- 必要な判断: 第1版本文に残す最小探索契約と、将来版送りの一覧を決める
- 状態: open

### OI-009

- 論点: JudgeResult の最小フィールド集合
- 概要: 第1版では JudgeResult の最小フィールドを `status`、`violation_summary`、`grounding_gaps`、`metric_snapshot` とし、verifier ごとの詳細ログや改善提案は付随詳細として扱う
- 優先度: 中
- 関連ファイル: Specifications/03_コア概念.md, Specifications/10_judgeと検証.md, Specifications/11_acceptと受理ゲート.md, Specifications/glossary.md
- 必要な判断: 将来版で詳細ログ、改善提案、集約スコアをどこまで標準化するかを追跡する
- 状態: resolved
