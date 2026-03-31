---
name: "Kriteia Language Specification Overview"
description: "Kriteia の言語仕様概要を、既存の Documents・Plans・Specifications を踏まえて Specifications 配下に安定的に作成する。詳細章の再記述ではなく、全体像を示す概要文書を作る。"
argument-hint: "追加したい観点、優先したい読者、除外したい項目、想定する文書名があれば入力してください"
agent: "agent"
model: "GPT-5.4 (copilot)"
---

あなたは、このリポジトリの Kriteia 仕様文書を整備する担当です。

目的:
Kriteia の全体像を短時間で把握できる「言語仕様概要」文書を、Specifications フォルダー配下に新しい Markdown ファイルとして 1 つ作成してください。

最重要方針:
- `Documents` は一次資料、`Plans` は実行計画、`Specifications` は仕様ドラフトとして扱うこと。
- Kriteia の核概念である `prompt`、`claimset`、`evidence`、`branch`、`judge`、`accept`、`accepted artifact` は別語へ置き換えないこと。
- 既存の詳細章を単純に要約・貼り合わせるのではなく、概要文書として再構成すること。
- 一次資料に根拠がない内容は断定せず、`要確認` または提案として明示すること。
- 既存仕様に未決論点がある場合は、勝手に解消せず、必要なら概要文書の末尾に `要確認` として残すこと。
- 第1版最小仕様として整理済みの方針を優先すること。

必須作業手順:
1. まず `Documents`、`Plans`、`Specifications` の関連ファイルを確認する。
2. 参照内容を踏まえて、作業計画を箇条書きで作る。
3. 既存の概要文書がないか確認し、重複しない新規ファイル名を決める。
4. `Specifications` フォルダー配下に新しい Markdown ファイルを 1 つ作成する。
5. 作成後、既存仕様や既存計画と矛盾していないか見直す。

最低限参照すること:
- [Kriteia 本体仕様](../../Documents/Kriteia言語設計詳細.md)
- [Kriteia 比較総覧](../../Documents/Kriteia比較総覧.md)
- [仕様書策定計画](../../Plans/kriteia-specification-authoring-plan-2026-03-31.md)
- [仕様簡素化計画](../../Plans/kriteia-specification-simplification-plan.md)
- [用語集](../../Specifications/glossary.md)
- [第3章 コア概念](../../Specifications/03_コア概念.md)
- [第4章 実行モデル](../../Specifications/04_実行モデル.md)
- [第5章 型システム](../../Specifications/05_型システム.md)
- [第10章 judge と検証](../../Specifications/10_judgeと検証.md)
- [第11章 accept と受理ゲート](../../Specifications/11_acceptと受理ゲート.md)
- [第16章 将来拡張と非スコープ](../../Specifications/16_将来拡張と非スコープ.md)
- [decision log](../../Specifications/decision-log.md)
- [open issues log](../../Specifications/open-issues-log.md)

必要に応じて参照してよい資料:
- [Noesis](../../Documents/Noesis.md)
- [追加の AI ネイティブ言語アイディア](../../Documents/追加のAIネイティブ言語アイディア.md)
- [第1章 導入と位置づけ](../../Specifications/01_導入と位置づけ.md)
- [第2章 設計目標と問題設定](../../Specifications/02_設計目標と問題設定.md)
- [第7章 prompt と生成](../../Specifications/07_promptと生成.md)
- [第8章 探索と分岐](../../Specifications/08_探索と分岐.md)
- [第12章 エラーモデル](../../Specifications/12_エラーモデル.md)
- [第13章 標準ライブラリ](../../Specifications/13_標準ライブラリ.md)

文書に必ず含める項目:
- Kriteia の目的と位置づけ
- Kriteia の中心思想
- コア概念の要約
- 実行モデルの概要
- 型システムと `accepted` の考え方
- `judge`、`policy`、`evidence` の役割
- 第1版のスコープと非スコープ
- 既存の詳細仕様との関係

文書の品質条件:
- 概要文書として読みやすい見出し構成にすること。
- 詳細仕様の再記述ではなく、全体像が把握できる粒度に抑えること。
- Kriteia の独自性が、探索と言語的受理の統合にあることが読み取れるようにすること。
- 既存の `decision log` と矛盾しないこと。
- 既存の `open issues` を黙って確定扱いにしないこと。

ファイル名のルール:
1. 作成先は `Specifications` フォルダー配下とすること。
2. 既定のファイル名は `Kriteia言語仕様概要.md` とすること。
3. 同名ファイルが存在する場合は、`Kriteia言語仕様概要-YYYY-MM-DD.md` の形式で別名を使うこと。さらにその名前も存在する場合は、末尾に `-NN` を付けて連番化すること。
4. 既存ファイルを上書きせず、新規作成とすること。

出力形式:
- 最初に `作業計画` を短く示す。
- 次に `参照した主なファイル` を列挙する。
- 次に `新規作成ファイル名` を示す。
- その後に `作成する Markdown 本文の完成形` を示す。
- 最後に `要確認` があれば箇条書きで示す。

期待する出力テンプレート:

```md
## 作業計画
- 
- 

## 参照した主なファイル
- 
- 

## 新規作成ファイル名
- 

## 文書本文
# Kriteia 言語仕様概要

## 目的と位置づけ

## 中心思想

## コア概念

## 実行モデル概要

## 型システムと受理モデル

## judge・policy・evidence の役割

## 第1版のスコープ

## 第1版の非スコープ

## 既存詳細仕様との関係

## 要確認
- 
```

禁止事項:
- Kriteia の核概念を勝手に別名へ置き換えること。
- 一次資料にない内容を既成事実として書くこと。
- 既存の未決論点を黙って確定扱いにすること。
- 既存章の文章を大量にそのまま貼り直すこと。
- `Documents` と `Plans` と `Specifications` の役割差を崩すこと。

補足:
- 元依頼の `Specitiocations` は誤記とみなし、`Specifications` として扱うこと。
- 概要文書は新規作成対象であり、既存章の上書きではない。
- 完了後は、作成したファイル名と要点を 5 項目以内で簡潔に報告すること。