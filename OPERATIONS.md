# OPERATIONS.md — 台本フィードバックサイト運用ルール

## ファイル構成

| ファイル | 役割 |
|----------|------|
| `index.html` | ハブ（台本一覧・共通項） |
| `<slug>.html` | クライアント別フィードバック（例: `dankogyo.html`） |
| `data.json` | 全クライアント・全数値の正本 |
| `admin.html` | 管理画面（隠しURL、リンク非掲載） |
| `OPERATIONS.md` | このファイル |

## データの流れ

```
data.json を編集
  → git commit & push
  → GitHub Pages が配信
  → 各HTMLがJSで data.json を fetch して描画
```

**HTMLは原則直接編集しない**。動的データは全部 `data.json` 経由。HTMLを触るのは「新規クライアントのテンプレ作成」「構成FB本文の追記」など、構造を変える時だけ。

## data.json の構造

### `clients[]` — クライアント別エントリ

| field | 説明 | 例 |
|-------|------|----|
| `slug` | URLスラッグ（HTMLファイル名と一致させる） | `"dankogyo"` |
| `page` | ハブから飛ぶHTMLファイル名 | `"dankogyo.html"` |
| `number` | 通し番号（zero-pad文字列） | `"01"` |
| `date` | 着手月（カードに表示） | `"2026.04"` |
| `name` | クライアント名 | `"暖工業"` |
| `subtitle` | カードのサブテキスト | `"船岡綾氏 / 採用ファン化動画 / 1日密着"` |
| `tag` | カードフッタのタグ | `"FANTSスタイル / 全15項目"` |
| `fb_points` | 構成FBの指摘項目数（HEROのFB POINTS集計に加算） | `15` |
| `stage` | 進行段階（後述） | `"shoot"` |
| `youtube_id` | YouTube動画ID。設定すると<br>サムネ自動表示＋詳細ページのサムネ→YouTubeリンク化 | `"abc123XYZ"` or `null` |
| `published_at` | 公開日 | `"2026.05.15"` or `null` |
| `metrics.views` | 再生数（数値、自動でカンマ区切り表示） | `12345` or `null` |
| `metrics.ctr` | CTR（数値、自動で%付き表示） | `5.2` or `null` |
| `metrics.retention` | 視聴維持率（数値、自動で%付き表示） | `48` or `null` |

### `stage` の値

| 値 | 状態 | ハブのフロー表示 | 詳細ページのPHASE表示 |
|----|------|------------------|------------------------|
| `shoot` | 構成FB済、撮影前/中 | 構成FB ✓ / 撮影 ◯ | PHASE 01 対応中 |
| `edit` | 撮影済、編集中 | 撮影 ✓ / 編集 ◯ | PHASE 01 完了 / PHASE 02 対応中 |
| `overall` | 編集済、最終全体FB中 | 編集 ◯（編集phase扱い） | PHASE 01-02 完了 / PHASE 03 対応中 |
| `done` | 公開済 | 全部 ✓ | 全PHASE 完了 |

### `patterns[]` — 共通項エントリ

| field | 説明 |
|-------|------|
| `num` | パターン番号（"01" 形式） |
| `title` | 見出し |
| `desc` | 本文（HTML可、`<strong>` 使える） |
| `source` | 出典（"暖工業 POINT 01" など） |

## 管理画面（admin.html）

URL: `https://akidori.github.io/dankogyo-feedback/admin.html`

リンクはハブから貼っていない（直URL アクセス）。`<meta name="robots" content="noindex,nofollow">` でクロール除外。

### 初回セットアップ（PAT登録）

1. [GitHub Settings → Personal Access Tokens (fine-grained)](https://github.com/settings/personal-access-tokens/new) で新規発行
2. 設定:
   - Resource owner: `akidori`
   - Repository access: **Only select repositories** → `dankogyo-feedback`
   - Permissions → Repository permissions → **Contents: Read and write**
   - 有効期限: 任意（90日推奨）
3. 生成されたPAT (`github_pat_...`) をコピー
4. admin.html 開く → ⚙PAT設定 → 貼って保存

PATは**このブラウザのlocalStorageのみ**に保存。コミットには含まれない。漏洩したらGitHub Settingsから即無効化。

### admin.html の使い方

- **CLIENTS**: クライアント一覧。クリックで展開、フォームで編集。
- **PATTERNS**: 共通項一覧。
- 各エントリの「削除」ボタンで削除。
- 末尾の「+ 新規クライアントを追加 / + 共通項を追加」で追加。
- 編集後、画面下部の **GitHubに保存** で `data.json` をリポジトリに直接コミット → Pages自動再ビルド（30-60秒で反映）。
- フォールバック:
  - **コピー**: JSON文字列をクリップボードへ
  - **ダウンロード**: `data.json` をローカル保存（手動push用）

### 注意

- admin.html はファイル単位の編集のみ。**新規クライアントのHTMLページ自体は別途作る必要がある**（`dankogyo.html` をコピー → `<新slug>.html` にリネーム → `data-slug` 書き換え → 構成FB本文を埋める）。

## 操作レシピ

### 数値を更新する（一番頻繁）

**admin.html を使う方法（推奨）:**

1. admin.html を開く
2. 対象クライアントを展開 → METRICS の views/ctr/retention を更新
3. 「GitHubに保存」

**手で `data.json` を編集する方法:**

`data.json` の対象クライアントの `metrics.{views,ctr,retention}` を YouTube Studio の値に書き換えて push。

```json
"metrics": {
  "views": 12345,
  "ctr": 5.2,
  "retention": 48
}
```

### 進行を進める

`data.json` の対象クライアントの `stage` を次の値に変更:

- `shoot` → `edit` （撮影完了時）
- `edit` → `overall` （編集完了時）
- `overall` → `done` （公開時）

### 動画公開時にやること

1. `youtube_id` をセット（YouTube URLの `?v=` 以降の文字列）
2. `published_at` をセット（`"2026.05.15"` 形式）
3. `stage` を `"done"` に
4. push

→ ハブカード・詳細ページのサムネが自動で表示される。

### 新規クライアントを追加する

1. `dankogyo.html` をコピーして `<新slug>.html` を作る
2. 新しいHTML内の `data-slug="dankogyo"` を `data-slug="<新slug>"` に変更
3. 構成FB本文（POINT 01〜）を新規クライアントの内容に書き換え
4. `data.json` の `clients[]` 末尾にエントリを追加

```json
{
  "slug": "<新slug>",
  "page": "<新slug>.html",
  "number": "02",
  "date": "2026.XX",
  "name": "クライアント名",
  ...
}
```

5. push

### 共通項を追加する

`data.json` の `patterns[]` に追記。`source` には抽出元のFB箇所を記載。

```json
{
  "num": "03",
  "title": "新パターン名",
  "desc": "本文。<strong>強調</strong>が使える。",
  "source": "クライアント名 POINT XX"
}
```

## 注意

- `data.json` はfetchで読むため `file://` では動かない。確認は GitHub Pages 上で。
- ブラウザキャッシュ対策に `?t=タイムスタンプ` を付けてfetchしているので、push後は基本リロードで反映される（GitHub PagesのCDN遅延で数十秒かかることはある）。
- JSON書き間違い（カンマ抜け・括弧抜け）でカード一覧が「読み込み中…」のまま止まる。push前に `python3 -m json.tool data.json` で構文チェックすると安全。
