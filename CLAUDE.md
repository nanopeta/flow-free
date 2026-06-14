# CLAUDE.md

このリポジトリで作業する際のガイド。

## プロジェクト概要

`flowfree.html` 単体で完結する Flow Free パズルゲーム。
外部ライブラリ・ビルド工程・サーバーは一切なし。すべてのロジック・スタイル・
データが 1 ファイルに収まっている。モバイルブラウザ（特に Android）での動作が前提。

## ファイル構成

- `flowfree.html` — ゲーム本体（HTML + CSS + JS + ステージデータすべて）
- `docs/HANDOFF.md` — 開発引き継ぎ資料（仕様・アルゴリズム・改善候補）
- `README.md` — ユーザー向け説明

## 編集時の注意

- **単一 HTML を維持する**。外部 JS/CSS への分割やライブラリ導入は、依頼が
  ない限り行わない（`content://` スキームやオフライン配布で動くことが要件）。
- 全 JS は `<script>` 内の IIFE `(function(){ "use strict"; ... })()` に閉じている。
- ステージデータは `const ALL_STAGES=[...]`（`flowfree.html` 内の 1 行。
  サイズ別に 4/5/6/7/8 が各 5 問、9/10/12 が各 10 問の計 55 問）。
- localStorage が使えない環境向けに `memStore` フォールバックあり
  (`storageGet`/`storageSet`)。

## 動作確認

ビルド不要。ブラウザで `flowfree.html` を開く。ローカル配信は
`python3 -m http.server 8000` → `http://localhost:8000/flowfree.html`。

JS の構文チェック（手早い健全性確認）:

```sh
node -e "const fs=require('fs');const h=fs.readFileSync('flowfree.html','utf8');new Function(h.match(/<script>([\s\S]*)<\/script>/)[1]);console.log('OK')"
```

## 主要データ構造

```js
// ステージ
{ size: 9, pairs: [[r1,c1,r2,c2], ...], paths: [[[r,c],...], ...] }
//   pairs = 各色の端点座標 / paths = 各色の正解経路（ヒントに使用）

// レベル定義
{ name:'マスター', size:9, stages:[...], infinite:true }  // infinite:true が∞モード
```

## 主要な状態変数

```js
let curLevel, curStageInLevel   // 現在位置
let N, pairs, paths, grid       // 盤面状態
let infiniteCount               // ∞モードのクリア数
let infFixedSize, infFixedNc    // ∞モードの選択サイズ・色数（0=ランダム）
```

## 主要な関数

| 関数 | 役割 |
|------|------|
| `buildLevelList()` | タイトル画面のレベル一覧を描画 |
| `showStageSheet(lvIdx)` | ステージ選択シートを表示 |
| `startLevel(lvIdx, stIdx)` | 指定レベル・ステージを開始 |
| `prepareStage()` | ステージを初期化・描画（∞モード対応） |
| `generateInfinite()` | ∞モード用問題をリアルタイム生成 |
| `regenerateAllStages()` | 全ステージを再生成 |
| `buildBoard()` / `renderCell(r,c)` | 盤面 DOM 構築 / 1 セルを SVG 描画 |
| `checkClear()` | クリア判定（∞モード分岐あり） |
| `showHintForColor(ci)` | ヒント表示（stage.paths を参照、なければ `solveForHint()`） |

## 問題生成アルゴリズム

バックトラック付きランダムウォークで全マスを埋め、端点を抽出して問題化。
生成時のパスをそのまま `paths` に保存するため、別途ソルバーなしで解が保証される。

## 今後の改善候補（未実装）

- 手数カウント表示
- 効果音・振動フィードバック
- ダークモード
- クリアアニメーション強化
- 問題生成の非同期化（Web Worker）で UI フリーズ解消
