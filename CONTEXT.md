# NewDOM

GPU-native UI rendering substrate。あらゆる言語の UI フレームワークと GPU バックエンドの間に位置する「描画のカーネル」。フレームワークでも状態管理でもなく、描画の共通語。

## Language

**NewDOM**:
GPU-native retained scene graph、言語非依存 C ABI、最小限のレイアウトエンジン、テキスト/ベクターレンダリングパイプライン、プラットフォームバックエンド抽象の総体。
_Avoid_: フレームワーク、ライブラリ、レンダラー単体

**Substrate**:
NewDOM そのものを指す別名。UI フレームワークの下層、GPU の上層に位置する基盤層。
_Avoid_: エンジン、フレームワーク、ランタイム

**Node**:
Scene Graph の最小構成単位。`ND_NODE_RECT` / `ND_NODE_TEXT` 等 GPU が直接理解できる型のみ存在する。HTML の div/span とは異なる。
_Avoid_: Element, Component, Widget

**Scene Graph**:
Node のツリー構造。Retained（保持型）であり、上層は差分のみを通知する。毎フレーム全体を再構築する Immediate mode とは対極。
_Avoid_: Virtual DOM, Element Tree

**Retained**:
Scene Graph が前フレームの状態を保持し、変更のあった Node のみを更新する方式。対義語は Immediate（毎フレーム全 Node を再構築）。

**Immediate**:
NewDOM が採用しない方式。毎フレーム全 Node を CPU で走査するコストと、テキスト shaping キャッシュが効かない問題から不採用。

**Backend**:
GPU API 抽象層。NewDOM は wgpu を唯一の Backend として使用し、wgpu が Vulkan / Metal / DX12 / WebGPU（ブラウザ）への変換を担う。NewDOM は独自の Backend 抽象を持たない。
_Avoid_: Renderer, Driver

**Binding**:
newdom.h（C ABI）を各言語の FFI 機構でラップしたもの。TypeScript / Python / Swift / Kotlin 等。Binding は薄いラップであり、ロジックを持たない。
_Avoid_: SDK, Wrapper, Port

**C ABI**:
newdom.h として公開される関数群。Rust の `extern "C"` + cbindgen で生成する。すべての Binding はこの C ABI を通じて NewDOM と通信する。
_Avoid_: API, Interface（文脈が曖昧な場合）

**Dirty Region**:
前フレームから変化のあった描画領域。NewDOM は Dirty Region のみを再描画し、変化のない領域の GPU Texture を再利用する。

**Glyph Atlas**:
レンダリング済みグリフを格納する GPU テクスチャ。LRU でエビクションし、UV 座標でアドレス指定する。

**NodeId**:
NewDOM が slotmap で払い出す不透明なハンドル。C ABI では `uint64_t` として公開する。上層は「どの entity がどの NodeId か」のマッピングを自身で管理する。削除済み Node への誤 update は generational check で検出される。
_Avoid_: Entity ID

**Phase 0**:
「ブラウザの canvas に wgpu + Vello で色付き矩形が 1 個描ける」状態。Qiita に投稿し、テキスト・C ABI・レイアウトを順次追加していく起点。

## Example Dialogue

> 「Node を追加したい」
> → 「Node の kind は何？ `ND_NODE_RECT` か `ND_NODE_TEXT` か？」
> 「テキストを表示したい」
> → 「まず `nd_text_shape()` で TextId を取得して、それを Node の props に渡す。Binding から呼ぶ場合は C ABI 越しになる」
> 「全部再描画したい」
> → 「NewDOM は Dirty Region しか再描画しない。全 Node を `nd_node_update()` で通知すれば全体が再描画されるが、それは上層の責務」
