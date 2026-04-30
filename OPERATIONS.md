# OPERATIONS.md — 台本フィードバックサイト運用ルール

## ファイル構成

| ファイル | 役割 |
|----------|------|
| `index.html` | ハブ（台本一覧・共通項） |
| `<slug>.html` | クライアント別フィードバック（例: `dankogyo.html`） |
| `data.json` | 全クライアント・全数値の正本 |
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

## 操作レシピ

### 数値を更新する（一番頻繁）

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
