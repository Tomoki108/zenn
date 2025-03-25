---
title: "無職の本気ポートフォリオ（Go * Vue.js * Google Cloud）"
emoji: "❤️‍🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [go, vue, googlecloud, 個人開発, ポートフォリオ]
published: false
---

無職が**本気で**ポートフォリオ[^1]を作成したので、記事にまとめます。ポートフォリオを作成している同士の参考になれば幸いです。

- 運用も考慮し、CICD ワークフローの構築、Infrastructue as Code によるリソース定義、開発環境と本番環境の分離も実現しています。
- **ソースコードは全て公開しています。** (LICENSE: [AGPL 3.0](https://www.gnu.org/licenses/agpl-3.0.en.html))

https://github.com/Tomoki108/burny

# 作ったもの

スクラム方式の開発チームに向けた、Burny というバーンアップチャートを生成するアプリケーションを作成しました。

:::message
バーンアップチャートは、プロジェクトの総ストーリーポイント、期間、各スプリントの実際の消化ストーリーポイントを元に、進捗が理想と比べてビハインドしているか上回っているかを可視化するチャートです。
:::

https://burny.page/

## 機能

- ユーザー登録（メールアドレスの検証含む）、ログイン
- プロジェクトとスプリントスタッツの CRUD
- バーンアップチャート、ベロシティチャートによるプロジェクトの進捗の可視化
- API キーによる API アクセスの提供
- モバイルデバイス対応

## 画面例

![screen_example](/images/202503_burny/project_detail_page.png)

# アーキテクチャ

## Backend

[README.md](https://github.com/Tomoki108/burny/tree/dev/api)

- Go の[echo](https://github.com/labstack/echo) FW で作成した REST API サーバー。Cloud Run 上で動作します。
- 緩い Clean Architecture で実装しており、`外部インターフェース層->ユースケース層->ドメイン層`の一方向の依存関係を持ちます。
  - 依存性逆転は DB との通信等を行う `infrastrucure <-> usecase package`の間でのみ適用。`infrastrucure`は`domain`のインターフェースを実装し、`usecase`にそれを DI しています。
- [uber-go/dig](https://github.com/uber-go/dig)による DI や、[asaskevich/EventBus](https://github.com/asaskevich/EventBus)によるイベント駆動処理も特徴です。
- [README.md](https://github.com/Tomoki108/burny/tree/dev/api)

## Frontend

[README.md](https://github.com/Tomoki108/burny/tree/dev/web)

- Vue.js 3 の Composition API で実装した SPA。Cloud Storage の静的サイトホスティングでホストしています。
- UI には[Vuetify](https://github.com/vuetifyjs/vuetify)と[Chart.js](https://www.chartjs.org/)を活用しています。
- Vue の[composable](https://ja.vuejs.org/guide/reusability/composables)、データストアの[Pinia](https://github.com/vuejs/pinia)によって状態管理を伴うロジックをカプセル化しています。

## Infra

[README.md](https://github.com/Tomoki108/burny/tree/dev/infra)

- Terraform による Google Cloud のリソースの IaC。
- `dev`, `prod`, `dns`の３つのルートモジュールと、それぞれに対応する Google Cloud のプロジェクトが存在します。
  - `dns`を独立させたのは、ドメインの管理を`prod`に入れてしまうと`dev -> prod`の依存関係が発生してしまうからです。
- リソース群を`backend`, `frontend`, `github (Gitub Actionsでの認証認可のためのリソース)` の３つのモジュールに切り出してルートから利用しています。
- `terraform graph`で作成した`prod`環境のリソース図：
  ![infra_architecture](/images/202503_burny/graph.png)

## CI/CD

[.github/workflows](https://github.com/Tomoki108/burny/tree/dev/.github)

### テスト戦略

最もコストパフォーマンスの高いテストを考えた結果、Backend、Frontend ともに正常系のシナリオテストのみを実装することにしました。

ここでは「外部サービスのみをモックした状態で、ユーザーの実際の操作と同様のフローで行う結合テスト」をシナリオテストと呼んでいます。

- Backend: [senariop_test.go](https://github.com/Tomoki108/burny/blob/dev/api/scenario/scenario_test.go)
  - API のレスポンス JSON をファイルで保持し比較する Golden ファイルテスト。
- Frontend: [{pgae_name}.spec.ts](https://github.com/Tomoki108/burny/tree/dev/web/tests)
  - [Playwright](https://playwright.dev/)を使った実際にブラウザを操作するテスト。
  - API は Playwrite の機能でモック。

### フロー

- PR を作成した場合、`api`, `web`ディレクトリに差分があれば対応するシナリオテストが実行されます。
- `dev`, `main`ブランチにプッシュした場合、必ず両方のシナリオテストが実行され、パスした場合のみ ブランチ名に対応する環境の Cloud Run 及び GCS へのデプロイが行われます。

# 活用した AI

# 終わりに

[^1]: ここでは能力を証明するための自作アプリケーションの意
