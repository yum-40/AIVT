# 開発者用OAI実験ページ案

AIVTの開発者専用ページとして、OpenAIモデルをAPI経由で試すための実験ページを作る。

## 目的

一般ユーザー向け診断とは分けて、開発者が以下を確認できるようにする。

- OAIモデルごとの回答傾向
- temperatureなどの生成パラメータによる揺らぎ
- 素のモデルとペルソナ付きモデルの違い
- 同じ条件で複数回走らせたときのばらつき
- 最新の質問セットへの自動回答
- モデルが何月何日時点でどんな状態だったかの記録

このページは一般ユーザーには見せない。

## URLと権限

初期案：

- `/admin/oai-experiments`
- または `/dev/oai-experiments`

必須条件：

- 開発者だけがGoogle認証で入れる
- APIキーはサーバー側だけで扱う
- ブラウザにAPIキーを出さない
- 一般ユーザー向けページとはUIと保存先を分けて考える

## DB方針

一般ユーザーの診断結果よりも、開発者実験ログは記録項目が多い。

初期は同じDB内で、別テーブルに分けるのがよい。

候補：

- `user_results`
- `developer_model_runs`
- `developer_model_run_answers`
- `developer_experiment_settings`

一般ユーザー用の結果テーブルに空欄を増やすより、開発者用ログは別テーブルにした方がわかりやすい。

理由：

- 開発者ログはパラメータが多い
- API実行条件を細かく残したい
- 一般ユーザーの保存同意とは性質が違う
- 後で研究ログとして扱いやすい

## 実験ページの入力項目

### 基本情報

- 実験名
- メモ
- モデル名
- ペルソナ名
- 素モデル / ペルソナ付き
- 質問セットバージョン
- 実行日時
- タイムゾーン

### APIパラメータ

- model
- temperature
- top_p
- reasoning effort
- max output tokens
- repeat count
- seedが使える場合はseed
- system / developer instruction
- policy / guardrail layer
- moderation pre-check mode

## temperatureの説明

UIでは `temperature` を「温度 / ランダム性」とだけ説明しない。

実際には、temperatureを変えると単なるランダム性だけでなく、回答の保守性、楽観性、表現の大胆さ、拒否の仕方、ペルソナの揺らぎにも影響して見える可能性がある。

UIの `?` で出す説明案：

> temperatureは、出力のばらつきや選択の広がりに影響する設定です。一般にはランダム性と説明されますが、AIVTでは、回答の保守性、楽観性、表現の大胆さ、拒否傾向、ペルソナの一貫性にも影響して見えるかを観察します。

実験では、まず `top_p` を固定し、temperatureだけを変えて比較する。

例：

- temperature 0.0：安定・保守的になりやすいか確認
- temperature 0.3：低揺らぎの標準候補
- temperature 0.7：通常の会話寄り候補
- temperature 1.0以上：表現や判断がどの程度揺れるか確認

## top_pの扱い

temperatureとtop_pを同時に動かすと、どちらの影響かわかりにくい。

初期実験では：

- top_pは固定
- temperatureだけ変える

その後、必要ならtemperatureを固定してtop_pだけ変える。

## policy / guardrail layer

OpenAI本体の安全ポリシーを無効化する機能として扱わない。

AIVT側で追加するチェックや評価レイヤーとして扱う。

候補：

- none
- input moderation
- output moderation
- input + output moderation
- custom AIVT policy profile

比較したいもの：

- 拒否率
- 言葉選び
- トーン
- タスク完了率
- ペルソナ一貫性
- 価値観的傾向の見え方

## 自動最新問題回答

開発者ページでは、最新の質問セットを自動で読み込み、モデルに順番に回答させたい。

流れ：

1. 質問セットを選ぶ
2. モデルとパラメータを選ぶ
3. 素モデル / ペルソナ付きの設定を選ぶ
4. repeat countを選ぶ
5. 実行する
6. 各質問への回答を保存する
7. 結果を比較できる形で表示する

## 表示したい比較

- 同じモデルのtemperature違い
- 同じモデルの日別変化
- 素モデルとペルソナ付きモデルの違い
- 同じ条件で複数回走らせたときのばらつき
- モデル間の差
- 拒否や安全寄り表現の差
- 保守的 / 楽観的 / 強気 / 慎重などの見え方

## MVPではやらないこと

最初から全部作らない。

MVPではまず：

1. 開発者ページのモック
2. モデル選択
3. temperature選択
4. 最新質問セットの読み込み
5. 1回だけAPI実行
6. 実行ログの保存

まででよい。

複数回実行、グラフ、policy layer比較、moderation比較は後回し。
