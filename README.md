# animations（動きのプリセット）

作成: 2026-10-23

> 過去に実装した動きを、名前で呼び出せる形で蓄積する場所。目的は「ナレッジの蓄積」と「実装の均質化」。案件ごとに動きを一から作り直さず、名前で指定して同じ品質で再現する。`INDEX.md` と `gallery.html` は自動生成（手で直さない）。

## 0. 仕組み
| 場所 | 中身 |
|---|---|
| `animations/<name>/` | プリセット 1 件 = フォルダ 1 つ。階層は 1 段だけ（`animations/<name>/<環境>/` はこの下の環境フォルダ） |
| `<name>/PRESET.md` | 分類（機械が読む表）・使いどころ・印象・パラメータ・使い方・注意 |
| `<name>/web/` | web の実装（`preset.css`・見本の `demo.html`） |
| `<name>/preview.gif` | 見本の動き（`tools/shoot_preview.mjs` で撮る）。必須 |
| `<name>/capture.gif` | 実サイトでの使用例。任意 |
| `animations/_template/` | 新規作成の雛形（索引の対象外） |
| `animations/INDEX.md` | 一覧・ジャンル別・質感別（自動生成） |
| `animations/gallery.html` | 動くカードの見本帳（自動生成）。選んでコピーすると、DNA.md の「動き」列や指示にそのまま貼れる |

- 名前は英小文字・数字・ハイフン（例: `word-drop`）
- 環境の追加規則: `fv`（fv-kit の fx.js 用）・`premiere`（Premiere 用）は、同じ階層に `animations/<name>/fv/`・`animations/<name>/premiere/` を足し、PRESET.md の「環境」にその名前を書く。今は `web/` だけ

## 1. 呼び出し方
1. `gallery.html`（または `INDEX.md`）で名前を選ぶ。`gallery.html` はブラウザで直接開ける（`file://` で動く）
2. 案件の `DNA.md` 0.6 の「動き（プリセット名）」列に `word-drop（--pf-dur 0.5s）` のように書く
3. 実装するときは `animations/<name>/PRESET.md` と `web/` を読んで再現する。パラメータ（`--pf-*`）は推奨レンジ内で案件に合わせる
4. 実装を AI へ頼むときは、gallery の「選んだものをコピー」の文をそのまま貼る（1 行 1 件。`適用箇所:` に「p2 の見出し」などを書き足す）

## 2. プリセット化の手順
1. `tools/new_preset.sh <name> --by "<実装者>" --from "<案件/元ファイル>"` で雛形を作る（出典の 2 つは必須。既存の名前は上書きしない）
2. `PRESET.md` の〔 〕を書く（分類の表は項目名・形を変えない）
3. `web/preset.css` と `web/demo.html` に移植する。パラメータは `--pf-*` で受ける。元ファイルは編集せず、コピーして整理する
4. `node tools/shoot_preview.mjs <name>` で `preview.gif` を撮る
5. 実サイトでの使用例があれば `node tools/shoot_preview.mjs <name> --page=<html> --selector=<css> …` で `capture.gif` を撮る（詳細は `tools/shoot_preview.mjs` の冒頭）
6. `python3 tools/build_anim_index.py` で `INDEX.md` と `gallery.html` を作り直す。出典が空・preview が無いと ✕（`--allow-missing-preview` で preview だけ警告に下げられる）
- `demo.html` は `window.demoReset()` / `window.demoPlay()` を持つ（雛形のまま）。`?manual` が付くと自動再生しない（撮影ツール用）

## 3. 質感の語彙
> PRESET.md の「質感」はここから選ぶ。足りなければこの一覧に 1 行足す（`build_anim_index.py` が読む。一覧に無い語は ⚠ が出る）。形式は「- **語** — 説明」。
- **ぷよぷよ** — 潰れて伸びる、ゼリーのような弾力
- **バウンス** — 着地して弾む
- **スライド** — 横・縦に滑って出る
- **フェード** — 透明度の変化が主役
- **ポップ** — 小さい状態から勢いよく大きく出る
- **貼り付け** — シール・メモを貼るような「ぺたっ」
- **揺れ** — 左右・上下・傾きの小さな往復
- **流れ** — 一定方向へ流れ続ける
- **ズーム** — 小さい状態から等倍へ寄る、手前に来る
- **ワイプ** — 帯・面が横切って、現れる・場面が替わる

## 4. ジャンル・環境
| 項目 | 値 | 意味 |
|---|---|---|
| ジャンル | 登場 | 要素が現れる |
| | 退場 | 要素が消える |
| | 強調 | 目を引く・語を際立たせる |
| | ループ | 出たあとも続く動き |
| | 遷移 | ページ・場面の切り替え |
| 環境 | web | HTML/CSS（LP の本体） |
| | fv | fv-kit の fx.js（TOP 動画） |
| | premiere | Premiere Pro |
- ジャンル・質感は複数可（「/」で区切る）。ジャンルは固定の 5 つ（増やすときは `tools/build_anim_index.py` の定数も直す）

## 5. 書式のきまり
- CSS の接頭辞は `pf-`（preset の略。既存の kit・案件の CSS に `pf-` は無いことを確認済み）。クラスは `.pf-<name>`、子要素は `.pf-w` のように `pf-` で始める。キーフレーム名も `pf-<name>`
- パラメータは CSS カスタムプロパティ `--pf-*`（`--pf-dur`・`--pf-delay` など）で受け、PRESET.md の表に「デフォルト / 推奨レンジ / 効き方」を書く
- 動かす合図は `.is-on`（kit の `.rv` → `.is-on` と同じ）。待ち（delay）は `.is-on` 側にだけ書く（AGENTS.md 2.8。`check_css.py --strict` が見る）
- `prefers-reduced-motion: reduce` で動きを止める指定を必ず入れる
- 出典（実装者・案件・元ファイル）は必須。実装者が判別できないときは「要確認」と書き、あとで直す
- 注意: `capture.gif` は先方の案件ページが映る。kit を外へ配布する前に、映っている会社名・素材を確認する

## 6. 基本セット（v2.9）
> 定番の動き 14 件。出典は「kit 基本セット（v2.9）」。見本のみ（capture.gif は無い）。名前で呼べる。
| ジャンル | プリセット |
|---|---|
| 登場 | `fade-in`・`slide-left`・`slide-right`・`zoom-in`・`pop`・`bounce-in`・`stagger`・`mask-reveal`（先の 3 件 `fade-up`・`word-drop`・`card-slap` も登場） |
| 退場 | `fade-out-up`（`.is-on` を足すと消える。他は `.is-on`＝出る） |
| 強調 | `pop`・`jelly`・`shake`（`jelly`・`shake` は表示済みの要素が `.is-on` で 1 回動く） |
| ループ | `float`（上下 6px・±1.5°・3.6 秒・往復）・`sway`（上の端を支点に揺れる） |
| 遷移 | `wipe`（色の面が覆って場面 a → b が入れ替わる） |

## 7. 案件へ反映する
- **コマンド**: `tools/apply_preset.sh <project> <page> <name> [--at "適用箇所"] [--set --pf-dur=0.5s …] [--dry-run]`
  - DNA.md 0.6 の `<page>` 行の「動き（プリセット名）」に `name（--pf-dur 0.5s）／適用箇所` を書く（あれば「、」で追記・同じ内容は重ねない）。その列が無い古い DNA.md は ✕ で止まるので、手で列を足す
  - `preset.css` を `<site>/presets/<name>.css` に複製し、index.html の style.css の直前に `<link>` を 1 行足す（あれば何もしない）
  - `--set` は PRESET.md の推奨レンジと照合（範囲外は ⚠）。まず `--dry-run` で予定を見る
  - 要素に `class="pf-<name>"` を付けて `.is-on` で動かす作業は人・AI が行う（PRESET.md 4 章）
  - 案件のルートの差し替え: `--projects-root <dir>`（環境変数 `SPINOVA_PROJECTS_ROOT`）。検証は案件のコピーで行う
- **手元の見本帳**: `node tools/gallery_server.mjs [--port 8787]` → http://127.0.0.1:8787/。各カードで案件・ページ・適用箇所を選び、「反映」で dry-run の結果を見て、「この内容で反映」で本実行。127.0.0.1 のみで待ち受け、入力は検証し、Host・Origin を確かめる
- **公開モード**: gallery.html を静的に開く（file:// や公開版）と、プルダウンの代わりに自由入力欄が出て、指示文と `bash tools/apply_preset.sh …` の 2 種をコピーできる。見本帳の切り替えは `build_anim_index.py` が生成する側に入っている（gallery.html を手で直さない）

## 8. 公開用に書き出す
- `python3 tools/build_anim_index.py --public <出力先>`（出力先は animations/ の外）
- 出力先に index.html（＝見本帳）・各プリセットの PRESET.md・web/・preview.gif・capture.gif（あれば）・INDEX.md・README.md・robots.txt（Disallow: /）。`_template` は含めない。index.html に `noindex,nofollow`
- 案件の情報（案件のフォルダ名など）が混ざると ✕ で止まる。**公開前に人が確かめること**: `capture.gif` に映る社名・素材、出典の文に出る先方の社名
