# NewDOM

> **NewDOM は、アプリケーション UI のための命令型・保持型・GPU ネイティブな描画オブジェクトモデルである。**

NewDOM は UI フレームワークではない。状態管理でもない。Reconciler でもない。

NewDOM が提供するのは、安定した NodeId を持つ描画オブジェクト群であり、上位層はそれらを直接 **create / update / move / destroy** する。React・Signal・ECS・Layout Engine は、NewDOM に mutation を流し込む *producer* であって、NewDOM コアそのものではない。

```
┌────────────────────────────────────────────┐
│  TypeScript  Python  Swift  Kotlin  Rust   │  ← 上位言語（何でもよい）
├────────────────────────────────────────────┤
│   React   Svelte   SolidJS   ECS   custom  │  ← UI フレームワーク（何でもよい）
├────────────────────────────────────────────┤
│                                            │
│               N E W D O M                 │  ← ここ
│       (C ABI / WASM / FFI で接続)          │
│                                            │
├────────────────────────────────────────────┤
│  WebGPU   Vulkan   Metal   DX12   OpenGL  │  ← GPU バックエンド（wgpu が抽象化）
└────────────────────────────────────────────┘
```

## ステータス

🚧 **Phase 0 — 実装中**（ブラウザ canvas に矩形が描けるところから始める）

## 技術スタック

| 役割 | 技術 |
|---|---|
| コア実装言語 | Rust |
| GPU バックエンド | [wgpu](https://wgpu.rs)（WebGPU / Vulkan / Metal / DX12 を統一抽象） |
| 2D レンダリング | [Vello](https://github.com/linebender/vello)（GPU compute shader、vendored） |
| レイアウト | [Taffy](https://github.com/DioxusLabs/taffy)（optional module、vendored） |
| テキスト | [parley](https://github.com/linebender/parley) + [fontique](https://github.com/linebender/fontique) + [skrifa](https://github.com/linebender/skrifa)（vendored） |
| NodeId 管理 | [slotmap](https://github.com/orlp/slotmap)（generational arena） |
| C ABI 生成 | cbindgen |
| WASM ビルド | wasm-pack |

## ドキュメント

- [設計仕様書 v0.1](docs/newdom-spec.md) — 原初の設計思想（※技術選定は ADR に上書きされています）
- [ドメイン用語集](CONTEXT.md) — NewDOM で使う言葉の定義
- [アーキテクチャ決定記録](docs/adr/) — 技術選定の理由と経緯
- [PRD](. scratch/newdom-substrate/PRD.md) — 要件定義

> **注意**: `docs/newdom-spec.md` は v0.1 の歴史的文書です。実装言語（C/Zig → Rust）・GPU バックエンド（独自抽象 → wgpu）・Layout の位置づけ等は `docs/adr/` の ADR によって上書きされています。最新の技術判断は ADR を参照してください。

## ライセンス

MIT — 商用利用・プロプライエタリフレームワークによる上乗せを制限しません。

依存ライブラリのライセンス（Vello 等 Apache-2.0 を含む）は `LICENSES/` ディレクトリで管理します。
