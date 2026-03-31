# Chord言語設計詳細

## アイディア 2: Chord

* **キャッチコピー**: チャネルではなく会話そのものを型にする並行言語
* **ターゲット層と主なユースケース**: 分散サービス、ワークフローエンジン、決済や認証のように手順の正しさが重要なバックエンド、マイクロサービス間連携を扱う開発者向けである。Go の goroutine と channel は使っているが、リークやプロトコル逸脱の検出が難しいと感じているチームに特に有効である。
* **解決する既存言語の課題**: Goは並行処理を簡潔に書けるが、チャネルの送受信順序や終了条件が型に乗らず、goroutine リークやハンドシェイク不整合がレビュー頼みになりやすい。TypeScriptやJavaではAPI仕様が OpenAPI や protobuf で別管理されがちで、実装とのズレが起きる。Erlang系は堅牢だが、メッセージの構造と会話全体の順序を静的に縛るのは得意ではない。Chordは「メッセージ」ではなく「会話手順」を第一級にして、並行処理の失敗を構文上起こしにくくする。
* **コアとなる言語仕様とパラダイム**: パラダイムは「プロトコル指向 + アクターモデル + 構造化並行性」である。実行モデルは軽量VM上のバイトコード実行を基本とし、ホットパスのみJIT最適化する。中核機能は `protocol` と `session` で、開発者は「開始」「応答待ち」「終了」の遷移を有限状態機械として記述する。spawn されたタスクは必ず親スコープに束縛され、親の終了時にはキャンセルか join のいずれかが強制されるため、孤児タスクが残らない。通信相手はチャネルではなくセッション型を持つため、送受信順序や返答漏れはコンパイル時に検出される。
* **特徴的なシンタックスの例**:

```chord
protocol Checkout {
    start(cart: Cart) -> awaiting Payment
    Payment.ok(tx: TxId) -> done Receipt
    Payment.fail(reason: Text) -> done Retry
}

service checkout uses Checkout {
    session start(cart) {
        let quote = pricing.request(cart)
        let payment = pay.request(quote.total)

        await payment {
            ok(tx)   => emit Receipt { tx: tx, total: quote.total }
            fail(r)  => emit Retry { reason: r }
        }
    }
}
```

この例では、`Checkout` が会話全体の正しい遷移を定義している。`await payment` の分岐は `Payment.ok` と `Payment.fail` 以外を取り得ず、`Receipt` を返したあとに追加メッセージを送ることもできない。Goの channel では「値の型」は守れても「会話の順序」は守れないが、Chordでは順序自体が型になるため、分散フローの正しさをロジックではなく型検査に寄せられる。

---

## 1. Chordとは何か

Chord は、並行処理と分散処理の中心的な失敗原因が「スレッド」や「ソケット」そのものではなく、「会話手順の破綻」にある、という前提から設計された言語である。

現実のサービス障害の多くは、次のような形で起きる。

* 送るべき応答を送らずにタスクが終了する。
* まだ受け取っていないメッセージを受け取った前提で処理を進める。
* タイムアウト後も子タスクが裏で生き続け、二重送信や二重課金を起こす。
* API仕様書は存在するが、実装コードの分岐と同期していない。
* リトライ、キャンセル、補償処理が別々のレイヤーに分散し、全体の会話が読めない。

既存言語は、メッセージの中身や関数の引数には型を付けるが、「次に何を送ってよいか」「どの段階でしかキャンセルできないか」「この分岐のあとに必ず join すべきか」といった会話の順序そのものは、型の外へ追い出してきた。Chord はこの空白を埋める。

言い換えると、Chord は「並行性のある制御フローを、手続きではなくプロトコルとして記述する言語」である。開発者はキューやチャネルの実装詳細から離れ、会話の型と、その会話に従うサービスの実装に集中できる。

## 2. Chordが狙う設計上の突破口

Chord の設計目標は、単に actor 言語をもう一つ作ることではない。以下の4点を同時に満たすことを狙う。

### 2.1 順序を型にする

通常の静的型付けは「この値は `PaymentResult` である」とは言えるが、「この `PaymentResult` は `QuoteRequested` のあとにしか来ない」とは言えない。Chord はこの制約を session type としてモデル化し、順序違反をコンパイルエラーにする。

### 2.2 並行タスクを必ず回収する

Go の goroutine は軽量だが、軽量であるがゆえにリークしやすい。Chord では `spawn` は常に構造化されており、親スコープの終了時に `join` か `cancel` のどちらかが必須になる。これにより、孤児タスクとバックグラウンド副作用の暴走を防ぐ。

### 2.3 API仕様と実装を分離しない

OpenAPI や protobuf は必要だが、それだけでは不十分である。型定義が別ファイルにあり、ワークフローの実装が別言語にあり、運用時の再試行ポリシーが設定ファイルにあると、会話全体は人間の頭の中にしか存在しなくなる。Chord では `protocol` がそのままコンパイル対象であり、仕様と実装が同じ言語断面上に存在する。

### 2.4 高度すぎる理論言語にしない

セッション型やプロセス計算だけを純粋に追うと、言語はしばしば研究色が強くなり、現場の開発者が使いにくくなる。Chord は理論的には session type と actor model に支えられているが、構文は業務ワークフローを自然に書けるよう、サービス、要求、応答、待機、タイムアウト、補償といった現場語彙で設計する。

## 3. パラダイムと実行モデル

Chord のパラダイムは次の3つの合成である。

* **プロトコル指向**: 会話の状態遷移が設計の中心になる。
* **アクターモデル**: サービスはメッセージ駆動の独立実体として実行される。
* **構造化並行性**: 並行タスクの寿命は親スコープに従属し、放置できない。

実行モデルは、軽量VM上のバイトコード実行を基本とし、ホットパスだけを JIT 最適化する。AOT ではなく VM を前提にする理由は3つある。

* 分散環境ではデプロイ単位を小さくし、動的更新しやすいほうがよい。
* protocol を有限状態機械へ落とし込み、実行時検証や観測を自然に行える。
* メッセージスケジューラ、タイムアウト、トレース収集をランタイムで統合しやすい。

ただし、すべてをインタプリタで済ませると性能が不足するため、以下の二層構造を取る。

* コントロールプレーン: セッション遷移、タイムアウト、キャンセル伝播、監視。
* データプレーン: シリアライズ、暗号化、バッチ処理、ホットループを JIT 最適化。

この分離により、会話の正しさと実行性能を両立させる。

## 4. 中核となる言語仕様

## 4.1 protocol

`protocol` は Chord の中心概念であり、単なる型定義ではなく会話そのものを表す。各遷移には以下の情報が乗る。

* 誰が開始するか
* どの値が流れるか
* 次の状態は何か
* どの状態で終了するか
* タイムアウトやキャンセルが許されるか

たとえば、決済では「見積取得後に決済依頼」「決済成功なら領収書」「失敗なら再試行案内」といった流れがある。この順序は値オブジェクトではなく protocol に記述される。

```chord
protocol PaymentFlow {
    begin(order: OrderId, amount: Money) -> awaiting Gateway
    Gateway.authorized(tx: TxId) -> done Receipt
    Gateway.declined(code: DeclineCode) -> done Retry
    timeout 3s -> done Escalate
}
```

この定義の意味は、`begin` のあとに来るのは `Gateway.authorized` か `Gateway.declined` か `timeout` のいずれかであり、それ以外は不正である、ということである。

## 4.2 session

`session` は protocol の具体的な実行インスタンスである。値を送るだけではなく、今どの状態にいるかを型として持つ。

```chord
session start(order, amount) {
    let result = gateway.authorize(order, amount)
    await result {
        authorized(tx) => emit Receipt { tx: tx }
        declined(code) => emit Retry { reason: code }
    }
}
```

ここで `emit Receipt` を実行した時点でセッションは `done` になる。以後、追加で `emit` や `await` を書くことはできない。線形型に近い考え方で、セッションハンドルは一度消費されたら再利用できない。

## 4.3 service

`service` は actor 的な実行主体であり、メールボックスと実行コンテキストを持つ。HTTP サービス、キューワーカー、イベント購読者、内部コンポーネントはすべて service として表現できる。

```chord
service invoice uses PaymentFlow {
    session begin(order, amount) {
        let auth = payment_gateway.authorize(order, amount)

        await auth {
            authorized(tx) => emit Receipt { tx: tx }
            declined(code) => emit Retry { reason: code }
        }
    }
}
```

`uses PaymentFlow` により、この service が守るべき会話契約が明示される。service 実装が protocol と一致しない場合はコンパイルが通らない。

## 4.4 spawn, join, cancel

Chord の並行性は必ず構造化される。`spawn` は親スコープに紐づくタスクを作り、`join` か `cancel` を強制する。

```chord
session start(order) {
    let stock_task = spawn inventory.reserve(order)
    let pay_task   = spawn billing.authorize(order)

    let stock = await stock_task
    let pay   = await pay_task

    emit Completed { stock: stock, payment: pay }
}
```

もし `pay_task` を待たずに session を抜けようとするとコンパイラが拒否する。これにより、未回収タスクによる二重処理やリソースリークを防ぐ。

## 4.5 timeout と compensation

分散処理では「失敗しない」より「失敗したときの戻し方」が重要である。Chord は補償処理を後付けの慣習ではなく、言語要素として持つ。

```chord
protocol Booking {
    start(req: TripRequest) -> awaiting Reserve
    Reserve.ok(id: BookingId) -> done Confirmed
    Reserve.fail(msg: Text) -> done Rejected
    timeout 5s -> compensates cancel_hold -> done Rejected
}
```

この `compensates` は ACID 的なロールバックではなく、分散トランザクションの補償をモデル化する。つまり、すでに起きた外部副作用を打ち消すための業務操作を protocol に明示する。

## 5. 型システムの要点

Chord の新規性は、一般的な静的型に加えて「会話型」を持つ点にある。重要なのは次の4種類である。

### 5.1 メッセージ型

通常の構造体型や代数的データ型である。ここは TypeScript や Rust と近い。

### 5.2 セッション状態型

`Session<Checkout, AwaitingPayment>` のように、セッションがどの状態にいるかを型として持つ。これにより、ある状態で送ってよいメッセージ以外は送れない。

### 5.3 線形セッションハンドル

セッションハンドルは複製できない。2つの並行経路から同じ session に二重送信することを防ぐためである。これは Rust の所有権に似ているが、対象がメモリではなく会話である点が異なる。

### 5.4 効果型

Chord は完全純粋言語ではないが、危険な副作用には注釈を付ける。たとえば `network`, `disk`, `external_commit` のような効果が関数シグネチャに現れ、補償不能な副作用を timeout の前に実行していないかを静的解析できる。

## 6. 典型的な構文パターン

## 6.1 基本的なリクエスト・レスポンス

```chord
protocol Auth {
    login(creds: Credentials) -> awaiting Verify
    Verify.ok(user: User) -> done SessionToken
    Verify.ng(reason: Text) -> done AuthError
}
```

最小単位でも「何を送るか」だけでなく「どの順序か」が明示される。

## 6.2 ファンアウトと集約

```chord
protocol Search {
    query(text: Text) -> awaiting Fanout
    Fanout.completed(result: AggregatedResult) -> done SearchResponse
}

service search uses Search {
    session query(text) {
        let web  = spawn web_index.find(text)
        let docs = spawn docs_index.find(text)
        let faq  = spawn faq_index.find(text)

        let aggregated = merge(await web, await docs, await faq)
        emit SearchResponse { result: aggregated }
    }
}
```

この書き方では、3つの検索タスクが親 session に従属している。途中で timeout が起これば、未完了タスクへ自動的に cancel が伝播する。

## 6.3 リトライを型安全に扱う

```chord
protocol ReliablePayment {
    start(req: PaymentRequest) -> awaiting Attempt[3]
    Attempt.ok(tx: TxId) -> done Receipt
    Attempt.retryable(err: RetryableError) -> awaiting Attempt
    Attempt.fatal(err: FatalError) -> done Failed
}
```

ここでは、再試行可能な失敗と即時終了すべき失敗を protocol 上で分けている。業務上の扱いの違いが型に反映される。

## 6.4 バージョン互換性

実運用では protocol が進化する。Chord では後方互換な変更と破壊的変更を区別する。

```chord
protocol Checkout v2 extends Checkout v1 {
    Payment.pending(ticket: PendingId) -> awaiting Payment
}
```

このように protocol 自体に版管理を持たせることで、IDL と実装の分離によるズレを減らす。

## 7. コンパイラとランタイムの設計

Chord を実用化するには、理論だけでなく処理系設計が重要である。処理系は次の段階を持つ。

### 7.1 フロントエンド

ソースを parse し、通常の AST に加えて protocol graph を生成する。ここで各 protocol は有限状態機械へ正規化される。

### 7.2 型検査

通常の型検査に加え、次を検証する。

* protocol の遷移が閉じているか
* session 実装が protocol を完全に網羅しているか
* spawn したタスクが回収されているか
* timeout と補償処理が矛盾していないか
* 破壊的副作用が protocol の安全域を越えていないか

### 7.3 中間表現

中間表現は通常の SSA だけでは足りない。Chord では control-flow graph に加えて conversation-flow graph を持つ。これにより、会話の型と通常コード最適化を同時に扱える。

### 7.4 VM

VM は mailbox scheduler と timer wheel を内蔵する。重要なのは、タスクスケジューラと protocol state machine が統合されている点である。これにより、タイムアウト、キャンセル、再送、監視イベントを同じ実行基盤上で扱える。

### 7.5 観測性

すべての session は trace id を持ち、状態遷移が構造化ログとして自動記録される。つまり observability は外付けの tracing SDK ではなく、言語の副産物になる。

## 8. Chordが既存言語より優れる点

## 8.1 Goより優れる点

Go は実装が簡潔だが、チャネルの使い方が protocol を表現しない。Chord はチャネルより上位の抽象である「会話」を持つため、順序と終端条件を型にできる。

## 8.2 Erlang/Elixirより優れる点

Erlang 系は耐障害性に優れるが、メッセージ順序の仕様を型検査する方向ではない。Chord は actor 的な強さを保ちながら、会話制約を静的に持てる。

## 8.3 TypeScriptより優れる点

TypeScript は API ペイロード型の表現には強いが、非同期の会話全体は開発者の規律に依存しやすい。Chord は async/await の上位概念として、会話フローを直接書ける。

## 9. Chordの弱点と現実的なトレードオフ

Chord は万能ではない。設計上のコストも明確に存在する。

* 小さなスクリプトや単純な CRUD API には過剰である。
* 会話が明示されるぶん、最初の設計コストは Go より高い。
* session type の推論はコンパイラ実装が難しく、エラーメッセージ設計が重要になる。
* 柔軟なアドホック通信を多用するシステムでは、厳密さが煩わしく感じられる可能性がある。
* 動的プロトコル生成をどこまで許すかは、実用性と検証可能性のせめぎ合いになる。

したがって Chord は、何でもこれで書く言語というより、「会話破綻のコストが高い領域」に強い特化言語として位置づけるのが現実的である。

## 10. 典型的なユースケース

Chord が真価を発揮するのは、次のような領域である。

* 決済や請求のように、順序違反が金銭事故に直結するフロー
* 認証、認可、二段階承認のように状態遷移が厳密なシステム
* マイクロサービス間のオーケストレーション
* BFF や API Gateway のように複数バックエンドを集約する層
* 長時間ワークフローと補償処理を持つ業務プロセス
* エージェント間通信のように、メッセージは単純だが会話制約が複雑なシステム

逆に、数値計算中心のバッチ、単一プロセス完結の CLI ツール、状態遷移がほぼ存在しないデータ変換では、Chord の利点は小さい。

## 11. 実装するとしたらどう始めるか

Chord を本当に作るなら、最初から巨大言語にしないほうがよい。段階的には次の順が妥当である。

1. `protocol` と `session` の最小構文を作る。
2. protocol を有限状態機械へ落とすコンパイラを書く。
3. 単一プロセス内 actor runtime を実装する。
4. `spawn/join/cancel` と timeout を追加する。
5. 補償処理と observability を統合する。
6. 最後に分散配置、永続化、バージョニングへ進む。

この順にすれば、理論先行で破綻せず、各段階で言語価値を検証できる。

## 12. まとめ

Chord の本質は、「並行プログラムの正しさを、スレッド安全性ではなく会話安全性として捉え直す」ことにある。Rust がメモリ所有権を型に押し上げたように、Chord は会話順序とタスク寿命を型に押し上げる。Go のような簡潔さ、Erlang のような耐障害性、TypeScript のような現場適用性を維持しつつ、それらが曖昧なまま残してきた「会話破綻」をコンパイル時に減らすことが、この言語案の最大の価値である。

新規性の核は、チャネル、関数、HTTP エンドポイントを第一級にするのではなく、「会話」を第一級にする点にある。そして実用性の核は、その理論を session type の研究言語に閉じ込めず、ワークフロー、決済、認証、分散オーケストレーションという現場の問題へ直接つなげる点にある。Chord は、並行性が難しい理由を「共有メモリ」だけに求めず、「正しい順序を表現する言語手段が不足している」という問題設定へ切り替えることで、既存言語とは異なる進化方向を提示する。
