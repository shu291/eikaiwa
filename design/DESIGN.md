# DESIGN.md — Talkie（じぶん英会話）の見た目のルール

実装中に迷ったらここを見る。ここに書いていないことは勝手に決めていい。
ここに書いてあることは**必ず守る**。

## 方向性

- 採用案: **A「向かいの席」**
- 雰囲気: 落ち着き / 深い青 / 余白多め / **相手のセリフが主役**
- なぜ: 夜に自室で1人で声を出す場面。眩しくなく、間違えても責められない静けさが要る
- 使う場面: 自室で1人、iPhoneを手に持って10〜15分。声を出すので人がいないとき

他の案は `design/design-brief.html` で見られる。切り替えるときは
`make_moodboard.py --emit-tokens B --tokens-out design/tokens.css` を実行して貼り直すだけ。

## このアプリの「主役」は何か

**相手が今言った英文、ただ1つ。** これが画面で一番大きく、一番明るい。

今までの弱点は、その英文のまわりに補助ボタンが6個・ミッション行・訳・新出表現が
同じ濃さで並んでいて、**どれを見ればいいか分からない**こと。順位を固定する。

| 順位 | もの | 見た目 |
|---|---|---|
| 1 | 相手の英文 | `--fs-xl`(22px) / `--text` / 行間ゆったり |
| 2 | 今どっちの番か | マイクの状態と「聞いています」の帯だけで示す |
| 3 | 自分の発言 | `--primary` の吹き出し |
| 4 | 訳・ヒント・フィードバック | `--fs-sm` / `--muted`。**開いたときだけ出す** |
| 5 | 補助ボタン | 既定で畳む。1個の「…」から開く |

## 色

`design/tokens.css` の変数だけを使う。**16進数のカラーコードをCSSに直接書かない。**

| 変数 | 使いどころ |
|---|---|
| `--bg` | ページ全体の背景 |
| `--surface` (`--card`) | カード・AIの吹き出し |
| `--border` (`--line`) | 罫線・カードの枠 |
| `--text` (`--ink`) | ふつうの文字。**相手の英文もこれ** |
| `--muted` (`--sub`) | 訳・注釈・日付など弱い文字 |
| `--primary` (`--acc`) | 主役のボタン・選択中・自分の吹き出し・マイク |
| `--on-primary` | `--primary` の上に乗る文字 |
| `--accent` | ミッション達成・連続日数など「おっ」と思わせる所 |
| `--danger` | 録音中・エラー・削除 |
| `--success` | 「自然な言い方です」 |

- **アクセント色は1画面に1〜2か所まで。** 全部に色を付けると主役が消える。
- **`--accent` はこのアプリでは「主役の色」ではない。** 主役は `--primary`。
  （元のコードは `--accent` を主役に使っていたので、実装時に `--primary` へ置き換えた）
- 段差は**影を使わず `1px solid var(--line)` で作る**（ダーク案なので `--shadow:none`）。

## 文字

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Zen+Kaku+Gothic+New:wght@400;500;700&family=JetBrains+Mono:wght@400;700&display=swap">
```

オフラインのときは読み込みに失敗するが、`--font-body` にシステムフォントの
フォールバックが並べてあるので**崩れない**。

- 見出し: `var(--font-heading)` / `700`
- 本文: `var(--font-body)` / `400`
- 数字・スコア・残り秒数: `var(--font-mono)` + `font-variant-numeric: tabular-nums`
  （残り秒数がカタカタ動かなくなる）

| 段階 | 変数 | px | 使う所 |
|---|---|---|---|
| 特大 | `--fs-2xl` | 28 | レポートの点数 |
| 大 | `--fs-xl` | 22 | **相手の英文**、画面タイトル |
| 中 | `--fs-lg` | 17 | カードの見出し |
| 標準 | `--fs` | 15 | 本文 |
| 小 | `--fs-sm` | 13 | 訳・ボタンの中・ラベル |
| 極小 | `--fs-xs` | 12 | 注釈。**これより小さくしない** |

### 禁止

- **11px以下を使わない。** 最小は12px。
- **太字だけで階層を作らない。** 「サイズ + 色 + 太さ」の3つで差をつける。
- `font-weight:800` 以上は1画面に1か所まで。

## 余白と形

- 余白は `--sp-1`〜`--sp-7` だけ。半端な `7px` `9px` `13px` を新しく書かない。
- **カードとカードの間は `--sp-5`(24px)。** 今まで14pxで全部詰まっていた。
- 角丸: カード `--r-lg`(20) / ボタン `--r`(16) / 小物 `--r-sm`(12) / チップ `--r-pill`

```css
.card{
  background:var(--card);
  border:1px solid var(--line);
  border-radius:var(--r-lg);
  padding:var(--sp-4);
}
```

## 背景（写真は使わない）

Openverse も Wikimedia も、このアプリに合う写真が取れなかった。
そもそも単一HTMLに写真を `data:` で埋めると 288KB が 1MB 近くになる。
**背景はCSSグラデーションで作る。**

```css
body{
  background:
    radial-gradient(1200px 600px at 80% -10%, color-mix(in srgb,var(--primary) 18%,transparent), transparent 60%),
    radial-gradient(900px 500px at 0% 110%, color-mix(in srgb,var(--accent) 10%,transparent), transparent 60%),
    var(--bg);
  background-attachment: fixed;
}
```

## アイコン

- 使うライブラリ: **lucide**（`stroke-width="2"`）。`design/icons/icons.js` の `icon()` を使う。
- **役割で使い分ける。混ぜているのではなく、意味が違う。**

| 種類 | 何を使うか | 例 |
|---|---|---|
| タブバー・主要操作・状態 | **SVG（lucide）** | マイク / 読み上げ / 会話を終える / 設定 |
| 中身の見分け | **絵文字** | ☕️カフェ ✈️入国審査 🧑‍🍳キャラの顔 |

理由: シーンやキャラの絵文字は**飾りではなく中身そのもの**（23シーンを一目で見分ける手段）。
一方マイクや設定は「機能」なので、色が変えられて大きさの揃うSVGにする。
**同じ列の中では必ずどちらかに統一する**（タブバーに絵文字を1個だけ混ぜる、はしない）。

- 意味のないアイコンには `aria-hidden="true"`、意味があるものには `aria-label`。

## 空っぽの画面（ここが一番の伸びしろ）

**データが0件の画面を、必ず「絵 + 一言 + ボタン1つ」にする。** 今は文字だけで寂しい。

```
   ┌─────────────┐
   │   （生成SVGの絵）    │
   │                     │
   │  まだフレーズがありません   │
   │  会話を1回すると、覚える   │
   │  言い方がここに貯まります   │
   │                     │
   │   [ 会話をはじめる ]    │
   └─────────────┘
```

対象: フレーズ帳0件 / 記録0件 / 復習の期限が来ていない / 弱点ノート0件。

## 動き

- 押した感: `transition:transform .12s ease` + `:active{transform:scale(.98)}`
- 出現: `fade` 0.18s。**150〜300msに収める**
- `@media (prefers-reduced-motion: reduce)` は tokens.css に入っている

## 最低限のアクセシビリティ

- タップできるものは **44×44px 以上**
- `color-scheme` を必ず書く（tokens.css に入っている）
- 色だけで情報を伝えない。録音中は**赤 + 「聞いています」の文字 + 波の動き**の3つで示す
- 装飾の画像・アイコンは `aria-hidden="true"`

## PWA

- `manifest.json` の `theme_color` と `<meta name="theme-color">` と `--bg` を
  **すべて `#0B1020`** に揃える。
- CSSを変えたら `sw.js` の `CACHE` の版を上げる。
