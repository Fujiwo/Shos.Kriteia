# 追加のAIネイティブ言語アイディア

Documents.Old の [Noesis.md](Documents.Old/Noesis.md) では、AIネイティブ言語の中核が「意味的型システム」「バックトラッキング」「Tree-of-Thought」「自動制約生成」に置かれていた。これは推論の探索と制御に強い方向性である。一方で、実務の AI アプリケーションでは、推論そのものと同じくらい「どの根拠に基づくか」「どの条件を満たしたら採用してよいか」「評価をどこで通すか」が重要になる。そこで本書では、Noesis とは別軸の AI ネイティブ言語として、根拠・検証・受理条件を第一級にする新案を提示する。

## アイディア 1: Anchor

* **キャッチコピー**: 生成ではなく受理を中心に据える、根拠指向AIネイティブ言語
* **ターゲット層と主なユースケース**: RAG システム、社内ナレッジ検索、法務・医療・金融のような高説明責任領域、AI コードレビュー、規程準拠チェック、自動レポート生成基盤の開発者を主対象とする。特に、「LLM は便利だが、そのまま本番へ流せない」という現場に向く。
* **解決する既存言語の課題**: Python や TypeScript で LLM アプリケーションを書く場合、モデル呼び出し、検索、引用付け、検証、スコアリング、リトライ、監査ログが別々のライブラリや外部設定に分散しやすい。その結果、最終出力に対して「なぜそれを採用したか」がコード上から読み取りにくい。Noesis が推論探索の構文化に強いのに対し、Anchor は「生成物を受理してよいか」を言語レベルで判定する点に主眼を置く。つまり、AI の中心問題を思考探索ではなく「根拠付き受理」に置き換える。
* **コアとなる言語仕様とパラダイム**: パラダイムは「証拠指向 + 制約付き生成 + 評価駆動データフロー」である。実行モデルは、AOT コンパイルされた制御コードと、モデル・検索・検証器を束ねる AI ランタイムの二層構造を採る。言語の第一級概念は `claim`、`evidence`、`draft`、`judge`、`accept` である。モデルは単にテキストを返すのではなく、`claim set` とその根拠候補を返す。ランタイムは retrieval、structured validation、policy check、evaluation metric を順に適用し、所定の基準を満たしたときのみ出力を `accepted artifact` として外部へ公開できる。これにより、AI 出力のライフサイクル全体を、生成から受理まで一つの言語で表現できる。
* **特徴的なシンタックスの例**:

```anchor
claimset SecurityAdvice {
    summary: Text
    findings: List<Finding>
    confidence: Score<0.0..1.0>
}

policy ReviewGate {
    require grounded(findings)
    require confidence >= 0.85
    require no_severity_below("high") for production_fix
}

flow analyze(repo: RepoSnapshot) -> accepted SecurityAdvice {
    let draft = model "gpt-5.4" generate SecurityAdvice
        from repo
        with retriever CodeSearch(top: 8)

    let checked = judge draft
        using CitationVerifier
        using PolicyChecker(ReviewGate)
        using Eval.Relevance(min: 0.80)

    accept checked
}
```

この構文では、`draft` はまだ採用前の生成物であり、そのまま外部へ返せない。`judge` は複数の検証器を流し、最後に `accept` が成功したものだけが `accepted SecurityAdvice` になる。重要なのは、AI アプリケーションの本質が「よいテキストを出すこと」ではなく「受理基準を満たした成果物だけを通すこと」として再定義されている点である。Anchor は、プロンプトや推論手順よりも、根拠、評価、採用条件、監査可能性を中心概念に据えることで、Noesis とは異なる AI ネイティブ言語像を提示する。

---

Anchor の新規性は、AI を「推論エンジン」としてだけでなく、「採用前提の不確実な下書き生成器」と見なし、その受理手続きを言語仕様に昇格させる点にある。Noesis が探索のための AI ネイティブ言語だとすれば、Anchor は根拠付き受理のための AI ネイティブ言語である。両者は補完関係にあり、AIネイティブ言語の設計空間を広げる追加案として成立する。
