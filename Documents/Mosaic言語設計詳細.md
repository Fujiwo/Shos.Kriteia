# Mosaic言語設計詳細

## アイディア 3: Mosaic

* **キャッチコピー**: if文の山をルールへ戻す、業務ロジック内蔵言語
* **ターゲット層と主なユースケース**: 業務システム、料金計算、審査、アクセス制御、構成チェック、データ品質検証のように「条件は複雑だが処理自体は単純」な領域に向く。特に、アプリケーションコードとSQL、設定DSL、ルールエンジンが分断されている現場で威力を発揮する。
* **解決する既存言語の課題**: PythonやTypeScriptはアプリケーション本体の記述は速いが、複雑な業務ルールは最終的に巨大な if 文、フラグ、設定ファイルの組み合わせに崩れやすい。PrologやDatalogはルール記述に強いが、汎用アプリケーション全体を書くには癖が強く、停止性や副作用の扱いも学習コストになる。Mosaicは汎用言語の中に停止性が高いルールブロックを埋め込み、ルールだけを別世界に追い出さずに済ませる。
* **コアとなる言語仕様とパラダイム**: パラダイムは「関数型コア + 制約付き論理型 + データフロー型」である。実行モデルはAOTコンパイルを基本としつつ、`rules` ブロックは組み込みのインクリメンタル推論エンジンで評価する。言語としての工夫は、ルール側を有限再帰または層化ルールに限定し、一般のPrologのような無制限なバックトラッキングを避けることにある。通常の関数やI/Oは命令型に近い自然な書き味を維持しつつ、複雑な判定だけを宣言的に切り出せるため、業務変更時の差分が読みやすい。
* **特徴的なシンタックスの例**:

```mosaic
type Order {
    id: OrderId
    country: Text
    total: Money
    tags: Set<Text>
}

rules PricingPolicy(order: Order) {
    vip if "vip" in order.tags
    tax_rate = 0.08 if order.country == "JP"
    discount = 0.15 if vip and order.total >= 10000yen
    review_required if order.total >= 300000yen and not vip
}

fn quote(order: Order) -> Quote {
    let facts = infer PricingPolicy(order)

    Quote {
        subtotal: order.total,
        tax: order.total * facts.tax_rate.or(0.10),
        discount: order.total * facts.discount.or(0.0),
        needs_review: facts.review_required
    }
}
```

この例では、業務ルールを `rules` に隔離し、通常コード側は `infer` した結果を読むだけでよい。重要なのは、MosaicがProlog的な表現力をそのまま持ち込むのではなく、「停止しやすく、差分更新しやすい範囲」に限定している点である。これにより、ルール変更時は `PricingPolicy` だけを更新すればよく、アプリケーション本体の制御フローを崩さずに済む。SQL、設定ファイル、アプリコードに分散しがちな判定ロジックを一か所に集約できるのが最大の価値である。

---

## 1. Mosaicとは何か

Mosaic は、業務システムにおける最も厄介な複雑性が「アルゴリズム」ではなく「ルールの散逸」にある、という認識から設計された言語である。

現場の多くのコードベースでは、次のような現象が起きる。

* 税率判定がアプリケーションコード、SQL、設定ファイルに分散する。
* 審査条件が if 文の入れ子、feature flag、管理画面設定に分かれて存在する。
* 例外条件だけが後付けで増え、最終的に誰も全体像を説明できなくなる。
* ある判定を変えたいだけなのに、Web API、バッチ、DBクエリ、UI表示ロジックを横断して修正が必要になる。

これは言い換えると、「ルールがコード構造に従属してしまっている」状態である。業務ルールは本来、手続きの枝分かれではなく、宣言的な知識として独立しているほうが読みやすい。しかし Prolog や Datalog のような論理型言語をそのまま本体言語として採用すると、今度は I/O、状態管理、UI、Web API、既存DB連携の書き味が重くなる。

Mosaic はこのギャップを埋める。通常のアプリケーションコードはあくまで自然な関数型または命令型スタイルで書きつつ、複雑な判定だけを `rules` ブロックへ引き上げる。つまり、論理型プログラミングを全面採用するのではなく、「ルールエンジンを言語へ内蔵する」方向の設計である。

## 2. Mosaicが狙う設計上の突破口

Mosaic の目的は、単に if 文を別記法へ置き換えることではない。狙っているのは次の5点である。

### 2.1 ルールを第一級にする

業務ルールを文字列や設定ファイルではなく、型付きのコンパイル対象として扱う。これにより、変更の影響範囲を追跡しやすくする。

### 2.2 停止しやすい論理系に限定する

Prolog のような無制限なバックトラッキングや任意再帰は強力だが、業務現場では予測不能性のほうが問題になる。Mosaic は層化ルール、有限再帰、単調推論を基本とし、止まりやすさと差分更新のしやすさを優先する。

### 2.3 通常コードと分断しない

ルールが別製品のルールエンジンにあると、アプリ側との行き来で理解コストが増える。Mosaic では `infer` の一行でルール結果を取得でき、通常コードと一つのモジュール内で共存できる。

### 2.4 変更差分を狭くする

業務変更はロジック全体の再設計ではなく、ルールの追記や条件修正で済むことが多い。Mosaic はその差分を `rules` ブロック内部に閉じ込めやすい構造を目指す。

### 2.5 観測可能な推論にする

結果だけ出ても、なぜその結論になったかが説明できなければ実務では弱い。Mosaic の推論エンジンは、どの規則が発火したか、どの条件で棄却されたかを追跡できる必要がある。

## 3. パラダイムと実行モデル

Mosaic のパラダイムは次の3つの混成である。

* **関数型コア**: 通常の値変換、データ合成、テスト容易性のために関数型中心の意味論を持つ。
* **制約付き論理型**: `rules` ブロックだけは宣言的な関係記述を行う。
* **データフロー型**: ルール結果は入力データの差分に応じて再計算され、全件再評価を避ける。

実行モデルは二層構造である。

* 通常コード: AOT コンパイルされたネイティブコードまたは高速バイトコード。
* ルールコード: コンパイル時に関係グラフへ変換され、実行時はインクリメンタル推論エンジン上で評価。

この分離が重要である。Mosaic はすべてを論理評価にしない。UI、HTTP、I/O、データ変換、JSON 処理は通常コードでそのまま書き、複雑な条件判定だけを論理的に処理する。

## 4. 中核となる言語仕様

## 4.1 rules ブロック

`rules` は、ある入力コンテキストから事実を導く宣言ブロックである。

```mosaic
rules Eligibility(app: LoanApplication) {
    adult if app.age >= 18
    stable_income if app.monthly_income >= 250000yen
    debt_heavy if app.debt_ratio > 0.45
    auto_reject if not adult or debt_heavy
    precheck_ok if adult and stable_income and not debt_heavy
}
```

ここで重要なのは、`adult` や `stable_income` が単なる一時変数ではなく、ルールエンジンにおける導出事実である点である。通常の if 文では評価順序がコードの本質になるが、Mosaic では関係性そのものが本質になる。

## 4.2 infer

`infer` は rules を評価して結果セットを返す。

```mosaic
let facts = infer Eligibility(app)
if facts.auto_reject {
    return Rejected
}
```

これにより、アプリケーション側は「どう推論したか」を rules に委ね、通常コードでは「何が導かれたか」だけを利用できる。

## 4.3 型付き事実

Mosaic の事実は単なる真偽値に限らない。値付き事実を扱える。

```mosaic
rules Pricing(order: Order) {
    region = "domestic" if order.country == "JP"
    region = "global" if order.country != "JP"
    tax_rate = 0.08 if region == "domestic"
    tax_rate = 0.10 if region == "global"
}
```

このように、事実は `bool` だけでなく `Text` や `Decimal` などの型付き値を持てる。重要なのは、衝突時の解決戦略が規則化されることである。

## 4.4 stratified rules

Mosaic は停止性を高く保つため、否定や集約を含むルールに層化制約を課す。

```mosaic
rules Review(order: Order) stratified {
    vip if "vip" in order.tags
    suspicious if order.total > 500000yen
    review_required if suspicious and not vip
}
```

`not vip` のような否定を許す代わりに、評価順序が循環しないことをコンパイラが確認する。これにより Prolog 的な強力さを一部残しつつ、無限推論や意味不明な再帰を避けられる。

## 4.5 explanation

Mosaic では、推論結果に理由を付けられることが重要である。

```mosaic
let result = infer Review(order) with explain
print(result.review_required.why())
```

説明可能性は、監査、法規制、サポート対応で極めて重要になる。Mosaic はここを外部ロギングで済ませず、言語仕様に近い位置で持つべきである。

## 5. ルールと通常コードの統合方法

Mosaic の実用性は、ルールが本体コードから孤立しないことにある。典型的な流れは次のようになる。

```mosaic
fn evaluate_order(order: Order) -> Decision {
    let policy = infer PricingPolicy(order) with explain

    if policy.review_required {
        enqueue_manual_review(order.id, policy.trace())
        return PendingReview
    }

    return Approved {
        tax: order.total * policy.tax_rate.or(0.10),
        discount: order.total * policy.discount.or(0.0)
    }
}
```

ここでは I/O やキュー投入は通常コード側で行い、推論結果だけを rules が提供している。この役割分担が重要である。Mosaic はアプリケーション全体を論理プログラムへ変えるのではなく、判定ロジックだけを論理的に持ち上げる。

## 6. 型システムの考え方

Mosaic の型システムは通常の静的型付けに加え、ルールの一貫性を保証する仕組みを持つ。

### 6.1 入力コンテキスト型

各 `rules` ブロックは明示的な入力型を持つ。これにより、ルールがどのデータに依存するかが明確になる。

### 6.2 事実スキーマ

コンパイラは rules から導出される事実集合のスキーマを生成する。`facts.tax_rate` が `Decimal?` なのか、`facts.review_required` が `Bool` なのかを静的に把握できる。

### 6.3 競合検出

同じ事実に矛盾する値が導出される場合、Mosaic は暗黙に決めない。優先順位、集約規則、または明示的な conflict policy を要求する。

```mosaic
rules Pricing(order: Order) conflict tax_rate use max {
    tax_rate = 0.08 if order.country == "JP"
    tax_rate = 0.10 if "special-tax" in order.tags
}
```

### 6.4 効果分離

`rules` ブロック内部では、原則として外部副作用を禁止する。DB 更新やネットワーク送信をルール内部に書けないようにすることで、推論の再実行可能性と説明可能性を保つ。

## 7. 推論エンジンの設計

Mosaic の価値は、ルール記法だけではなく、推論エンジンの現実性にある。推論エンジンは次の特徴を持つべきである。

### 7.1 インクリメンタル評価

入力の一部だけが変わった場合、すべてを再計算せず、影響のある事実だけを更新する。これにより、料金計算やアクセス制御評価のようなホットパスにも適用しやすくなる。

### 7.2 差分トレース

前回と今回でどのルールが新たに発火したか、どの条件が失われたかを追跡できる。監査やデバッグに有効である。

### 7.3 層化評価

否定や集約を含むルールは層ごとに評価される。これにより、意味が安定した結果を得やすくする。

### 7.4 キャッシュ可能性

`infer Policy(x)` の結果は入力値と依存関係に基づきキャッシュできる。ルールが副作用を持たないため、メモ化と再利用が容易である。

## 8. 典型的な構文パターン

## 8.1 アクセス制御

```mosaic
rules Access(user: User, resource: Resource) {
    admin if "admin" in user.roles
    owner if resource.owner_id == user.id
    same_org if resource.org_id == user.org_id
    can_read if admin or owner or same_org
    can_write if admin or owner
}
```

RBAC と ABAC が混ざる典型的な場面で、if 文より読みやすい。

## 8.2 料金計算

```mosaic
rules Shipping(cart: Cart) {
    oversized if cart.weight > 30kg
    remote_area if cart.prefecture in ["沖縄", "北海道"]
    fee = 1500yen if oversized
    fee = 800yen if remote_area
    fee = 0yen if cart.total >= 15000yen and not oversized
}
```

このように、条件の積み重ねが多い料金計算に向く。

## 8.3 データ品質検査

```mosaic
rules CustomerQuality(c: Customer) {
    missing_email if c.email == null
    invalid_age if c.age < 0 or c.age > 120
    risky if missing_email or invalid_age
}
```

データクレンジングや ETL の事前判定にも相性がよい。

## 8.4 例外と優先順位

```mosaic
rules Discount(order: Order) priority {
    discount = 0.20 if "campaign" in order.tags priority 10
    discount = 0.05 if "member" in order.tags priority 5
    discount = 0.00 default
}
```

現場では「原則」と「特例」が混在するため、優先順位付けを無視できない。Mosaic はこの現実も言語仕様に取り込む。

## 9. Mosaicが既存技術より優れる点

## 9.1 PythonやTypeScriptより優れる点

通常コードの書きやすさを大きく損なわずに、複雑な判定を if 文の入れ子から分離できる。変更差分が小さくなりやすい。

## 9.2 Prologより優れる点

論理記述の強みを借りつつ、停止性、説明可能性、アプリケーション統合を重視している。汎用コードとの境界が明快である。

## 9.3 Datalog単体より優れる点

推論だけでなく、その結果をどう使うかまで一つの言語で書ける。SQL 変換や外部ルールエンジン連携のオーバーヘッドを減らせる。

## 10. Mosaicの弱点とトレードオフ

Mosaic にも明確な制約がある。

* 任意再帰や探索を強く制限するため、純粋な論理型言語より表現力は落ちる。
* ルールが増えすぎると、今度は rules ブロック自体が巨大化しうる。
* 優先順位、競合解決、層化制約の設計を誤ると、かえって理解しにくくなる。
* 何でも rules へ押し込むと、アルゴリズムまで宣言的にしようとして破綻する。
* 推論トレースや差分更新を実装するコンパイラとランタイムの難易度は高い。

したがって Mosaic は、計算そのものより「判定」が複雑な領域に適用するのが正しい。

## 11. 典型的なユースケース

Mosaic が特に向くのは次のような領域である。

* 料金計算、税計算、割引適用
* ローン審査、保険引受、信用スコア前処理
* アクセス制御、ポリシー判定、構成検証
* データ品質チェック、入力検証、監査前処理
* ワークフローの分岐条件管理
* 規制対応ロジックや社内ルールの明文化

逆に、数値最適化、低レベルシステム開発、GUI 中心のアプリ、重い状態遷移制御そのものには向かない。

## 12. 実装するとしたらどう始めるか

Mosaic を実装するなら、最初から完全な論理型言語を目指さないほうがよい。段階的には次の順が現実的である。

1. `rules` と `infer` の最小構文を作る。
2. bool と値付き事実だけを持つ単純な推論エンジンを実装する。
3. 競合検出と explanation を追加する。
4. stratified negation と単純な集約を導入する。
5. 差分更新とキャッシュを入れる。
6. 最後に SQL やデータソース連携を整える。

この順なら、ルールの可読性と現実の処理系コストを段階的に検証できる。

## 13. まとめ

Mosaic の核は、「業務ルールをコードの枝分かれではなく、宣言的な知識として扱い直す」ことにある。Prolog のように世界全体を論理で覆うのではなく、通常コードと共存できる範囲へ論理性を限定することで、実務向けの言語として成立させる。

その価値は次の3点に要約できる。

* ルール変更の差分を狭くできる。
* 推論結果に説明可能性を持たせやすい。
* 通常コードとルールコードを一つの言語で保守できる。

新規性は、論理型プログラミングをそのまま復活させるのではなく、停止性、差分更新、説明可能性、アプリ統合という現代実務の制約に合わせて切り出し直す点にある。Mosaic は、Python や TypeScript の書きやすさを捨てずに、Prolog や Datalog が得意だった「複雑な判定の明示化」を取り戻すための現実的な次世代言語案である。
