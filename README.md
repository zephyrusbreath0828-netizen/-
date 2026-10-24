# 神旅帳（かみたびちょう）

全国47都道府県・100社の神社をめぐり、**参拝記録**と**御朱印**を残す神社巡礼アプリ（MVP）。

- 神社をさがす（現在地から近い順 / 都道府県 / 名前検索）
- GPSチェックインで参拝記録（神社から **300m以内** のみ登録可能）
- 御朱印の写真・参拝日・メモを保存（御朱印帳）

## 技術構成

| 領域 | 採用技術 |
| --- | --- |
| フレームワーク | Next.js 15（App Router）/ React 19 / TypeScript |
| スタイル | Tailwind CSS v4（和風モダンのデザイントークン） |
| 認証 | Firebase Authentication（Google ログイン） |
| データ | Cloud Firestore（`users` / `shrines` / `visit_history` / `goshuin`） |
| 画像 | Firebase Storage（`goshuin/{uid}/`、`avatars/{uid}/`） |
| 地図 | Google Maps JavaScript API（未設定時は OpenStreetMap の簡易地図） |
| PWA | Web App Manifest + Service Worker（オフラインページ対応） |

## すぐに動かす（デモモード）

Firebase の設定がなくても起動できます。データはブラウザの `localStorage` に保存されます。

```bash
npm install
npm run dev      # http://localhost:3000
```

`?demo=1` を URL に付けると、Firebase 設定済みでもデモモードで起動します（動作確認用）。

> チェックインの確認は、神社詳細の「動作確認用：この神社の位置を現在地にする」から
> 擬似現在地を設定すると、実際に参拝していなくても一連の流れを試せます。

## Firebase に接続する

```bash
cp .env.local.example .env.local   # NEXT_PUBLIC_FIREBASE_* を記入
npm run seed                       # 神社マスタ100件を Firestore へ投入
npm run build && npm start
```

詳細は [`docs/DEPLOY.md`](docs/DEPLOY.md) を参照。

## ディレクトリ構成

```
src/
  app/                 画面（App Router）
    page.tsx             ホーム（現在地・地図・周辺神社・訪問済件数）
    login/page.tsx       ログイン
    shrines/page.tsx     神社一覧（都道府県別・名前検索）
    shrines/[id]/page.tsx 神社詳細（チェックイン・御朱印登録）
    goshuin/page.tsx     御朱印帳
    goshuin/new/page.tsx 御朱印の登録
    mypage/page.tsx      マイページ（総参拝数・履歴・御朱印数）
    profile/page.tsx     プロフィール編集
  components/          UI 部品（AppShell / ShrineMap / CheckInPanel など）
  lib/                 ドメイン型・位置情報・Firebase 連携・状態管理
  data/shrines.ts      神社マスタ100件（自動生成）
firestore/             投入用 JSON・セキュリティルール・インデックス
scripts/               データ生成 / Firestore 投入スクリプト
docs/                  デプロイ手順書
```

## Firestore 構成（4コレクションのみ）

| コレクション | 主なフィールド |
| --- | --- |
| `users/{uid}` | nickname, photo_url, created_at, updated_at |
| `shrines/{shrineId}` | name, prefecture, address, latitude, longitude, main_deity, description, image_url, visit_count |
| `visit_history/{visitId}` | user_id, shrine_id, shrine_name, shrine_prefecture, visited_at, latitude, longitude, distance_m |
| `goshuin/{goshuinId}` | user_id, shrine_id, shrine_name, photo_url, visited_date, memo, created_at |

## データについての注意

`src/data/shrines.ts` の緯度経度は**境内代表点の概略値**です。本番運用の前に必ず公式情報で検証してください。
`visit_count` の初期値もサンプル値です（本番ではチェックイン時に加算されます）。

## 今回実装していない機能（将来バージョン）

祭神カード / 神話図鑑 / ランキング / SNS / デジタル巡拝印 / バッジ / 通知 / AIチャット / AI神話解説 / 管理者画面
