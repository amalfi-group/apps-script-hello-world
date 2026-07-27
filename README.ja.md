# Apps Script Hello World

[![CI](https://github.com/amalfi-group/apps-script-hello-world/actions/workflows/ci.yml/badge.svg)](https://github.com/amalfi-group/apps-script-hello-world/actions/workflows/ci.yml)

[English](README.md)

[Apps Script Fleet](https://github.com/h13/apps-script-fleet) テンプレート
(「1 リポ = 1 機能」)で作られた最小の Google Apps Script 関数です。
hello-world の挨拶を返すだけ — 同時に、キーレス CI/CD 認証フローを
エンドツーエンドで最初に検証したフリートのカナリアリポでもあります。

## デプロイの仕組み

| 操作 | 結果 |
| --- | --- |
| PR を開く | CI 実行(lint + 型チェック + テスト、カバレッジ 80%) |
| `dev` に push | **development** GAS プロジェクトへ CD デプロイ |
| `main` に push | **production** GAS プロジェクトへ CD デプロイ |

`src/` の TypeScript を Rollup で `dist/` にバンドルし、clasp で push
します。Apps Script から呼べるのは `src/index.ts` のトップレベル関数のみです。

### GAS プロジェクト

| 環境 | スクリプト |
| --- | --- |
| development | [script.google.com/d/1pqOIAwMX8TelAY47GfcPZluTLFW3ioGBTnXVffr_BVzsQoHJhDYMxpLq](https://script.google.com/d/1pqOIAwMX8TelAY47GfcPZluTLFW3ioGBTnXVffr_BVzsQoHJhDYMxpLq/edit) |
| production | [script.google.com/d/1HQJM7HR13mtZMTyvv9PTcnT1P8SmqemInmSwv7LI4xwZgUP5sZ3OIVac](https://script.google.com/d/1HQJM7HR13mtZMTyvv9PTcnT1P8SmqemInmSwv7LI4xwZgUP5sZ3OIVac/edit) |

## 認証情報(キーレス CI)

CI は **clasp の長期シークレットを一切保持しません**。デプロイ時に
**Workload Identity Federation (OIDC)** で Google Cloud に認証し、中央 GCP
プロジェクトの **Secret Manager** からフリート共有の clasp 認証情報を取得
します。ローテーションと監査は中央で一元管理 — このリポ側の更新作業は
ありません。詳細:
[secret-manager.ja.md](https://github.com/h13/apps-script-fleet/blob/main/docs/secret-manager.ja.md)。

リポ固有の設定は GitHub の `development` / `production` environment
(`CLASP_JSON` secret + `DEPLOYMENT_ID` variable)にあり、`./scripts/init.sh`
が一度だけセットアップしています。

## 開発

```bash
pnpm install
pnpm run check      # lint + 型チェック + テスト
pnpm run build      # dist/ へバンドル
```

- `src/greeting.ts` — hello-world のロジック
- `src/index.ts` — GAS エントリポイント(`doGet`, `getMessage`)
- `test/` — Jest テスト

ツーリング(CI/CD・lint・ビルド設定)は upstream テンプレートが管理し、週次の
[template sync](.github/workflows/sync-template.yml) PR で最新に保たれます —
同期対象は `.templatesyncignore` を参照してください。
