---
title: GeoArrow on Web; Can We Live Without GeoJSON?
format:
  gfm:
    variant: +yaml_metadata_block
theme: default
# background: /background.png
fonts:
  sans: Noto Sans JP
  mono: Fira Code
  weights: 600,900
class: text-center
drawings:
  persist: false
mdc: true
---

# <span color="blue">GeoArrow</span> on Web <br/> Can We Live Without <span color="orange">GeoJSON</span>?

Hiroaki Yutani
(MIERUNE, Inc.)

---
layout: image-right
image: "/icon.jpg"
backgroundSize: 70%
---

# ドーモ！

## 名前:

湯谷啓明 (@yutannihilation)

## 自己紹介:

- 株式会社 MIERUNE 所属
- 好きな言語は R と忍殺語

---

# 今日話すこと

<v-clicks>

- お詫び：シェーダーの話はほぼ出てきません
  - そこまで行きつかなかった...
- Apache Arrow・GeoArrow エコシステムについて
- GeoJSON なしで生きていきたいけど先は長い

</v-clicks>


---
layout: section
---

# GeoJSON、便利ですよね

---
layout: image
image: "/figure01-01.jpg"
---


---
layout: image
image: "/figure01-02.jpg"
---


---
layout: image
image: "/figure01-03-1.jpg"
---

---
layout: image
image: "/figure01-03-2.jpg"
---

---
layout: image
image: "/figure01-03-3.jpg"
---


---
layout: image
image: "/figure01-04.jpg"
---

---

# GeoJSON の問題点

- テキストとの変換のオーバーヘッドがある
  - 一度テキストに変換して、そのテキストをパースする、というムダ
- 小さいデータの場合は気にならないが、大きいデータだと問題になってくる

---

# Apache Arrow

- 無駄なデータコピーを発生させないためのデータフォーマット
  - 様々なプログラミング言語向けのライブラリや、関連ツールが用意されている
- 保存のためのフォーマットではなく、メモリ上のデータ処理に最適化されている

---
layout: image
image: "/figure02-01.jpg"
---

---
layout: image
image: "/figure02-02.jpg"
---


---

# GeoArrow

- Apache Arrow 形式でGISデータを扱う際の仕様
- C、Rust、Python、R、Wasm の実装がある

---
layout: image
image: "/figure01-04.jpg"
---


---
layout: image
image: "/figure01-05-1.jpg"
---

---
layout: image
image: "/figure01-05-2.jpg"
---

---

# 理想

- データソースから GPU まで変換なしでデータを届けたい！

---

# 実例：Lonboard

- GIS データを可視化する Python ライブラリ
- Python から deck.gl に GeoArrow 形式でデータを渡すので爆速

<Transform :scale="0.55">

![](/lonboard.png)

</Transform>

---

# しかし、現実は...

<v-clicks>

- GeoArrow 形式でデータを渡してくれるバックエンドやミドルウェアがあまりない
- GeoArrow をレンダリングしてくれるフロントエンドがあまりない

</v-clicks>

---
layout: image
image: "/figure01-01.jpg"
---

---
layout: image
image: "/figure03-01.jpg"
---

---

# バックエンド側

- ADBC（+ PostGIS、DuckDB、Snowflake など）
- Arrow SQL Flight
- Apache DataFusion

---

# ADBC (Arrow DataBase Connector)

- データベースを操作するための規約
- Python や R など、さまざまな言語・ツールから利用できる
- 列指向
- 要は Java の要らない JDBC みたいなもの

---

# Apache DataFusion

- クエリエンジン（≒ DB）
- InfluxDB など様々なプロジェクトで使われていて、意外と実績がある
- GeoArrow がこれ向けに `ST_*` の関数を実装していこうとしているので、今後に期待

---

# いちおう MIME タイプは決まっている


- ファイルフォーマット：`application/vnd.apache.arrow.file`
- ストリーミングフォーマット：`application/vnd.apache.arrow.stream`

<v-clicks>

- でもこれを返してくれる SQL ゲートウェイみたいなやつをほぼ見たことがない...

</v-clicks>

---

# 現状の選択肢

- SQL ゲートウェイみたいなやつを自作する
  - 簡単なやつならそんなに難しくないはず
  - 例：[Rust で ADBC 経由で DuckDB にクエリを投げる - Zenn](https://zenn.dev/yutannihilation/articles/48fec15ddc565d)
- DuckDB Wasm を使ってフロントエンドで完結
  - むしろこのユースケースが本命かも

---
layout: image
image: "/figure03-02.jpg"
---

---

# フロントエンド側がやること

<v-clicks>

- GeoArrow 形式のデータを受け取る（**簡単**）
  - Apache Arrow
- それをレンダリングする（**めっちゃむずい**）
  - MapLibre GL JS
  - deck.gl
  - ...

</v-clicks>

---

# GeoArrow 形式のデータを受け取る

- Apache Arrow の JavaScript 版を使う
- GeoArrow だからといって特殊なことはない

```js
import { tableFromIPC } from 'apache-arrow';

const buf = await fetch('https://...');
const dataFrame = await tableFromIPC(buf);

// 地物のカラムを取り出す（「geometry」という名前の場合）
const geoms = dataFrame.getChild("geometry");
```

---

# それをレンダリングする

- ネイティブで GeoArrow をサポートしてくれている地図描画ライブラリはない

<v-clicks>

- deck.gl
  - `@geoarrow/deck.gl-layers` を使う
- MapLibre GL JS
  - `@geoarrow/deck.gl-layers` を deck.gl レイヤーで使う
  - 自分でシェーダーを書く（タイトル回収！）

</v-clicks>

---

# `@geoarrow/deck.gl-layers`

- GeoArrow 公式が提供しているライブラリ
- deck.gl にはバイナリデータを直接流し込む API があり、それをいい感じにラップしてくれている
- 使い方は通常の deck.gl とほぼ同じ

---

# deck.gl を MapLibre に重ねるには

- いくつか方法がある
  - 参考：[Using with MapLibre | deck.gl](https://deck.gl/docs/developer-guide/base-maps/using-with-maplibre)
- MapLibre のカスタムレイヤーのひとつとして扱う（interleaved 方式）のが一番軽そう

---


```ts
// deck.gl を Mapbox や MapLibre にレイヤーとして
// 重ねるためのライブラリ
import { MapboxOverlay } from '@deck.gl/mapbox';

// `POINT` の地物を描画する deck.gl のレイヤー
import { ScatterplotLayer } from '@deck.gl/layers';
```

---

```ts
const deckOverlay = new MapboxOverlay({
  interleaved: true,
  layers: [
    new ScatterplotLayer({
      id: 'deckgl-circle',
      data: ...,
      getPosition: d => d.position,
      getFillColor: [255, 0, 0, 100],
      getRadius: 1000,
      beforeId: 'watername_ocean'
    })]});

map.addControl(deckOverlay);
```

---

# `@geoarrow/deck.gl-layers`に変更

- `GeoArrow` が先頭についたクラスに変更

```diff
- import {ScatterplotLayer} from '@deck.gl/layers';
+ import {GeoArrowScatterplotLayer} from '@geoarrow/deck.gl-layers';
```

```diff
  const deckOverlay = new MapboxOverlay({
    interleaved: true,
    layers: [
-     new ScatterplotLayer({
+     new GeoArrowScatterplotLayer({
        id: 'deckgl-circle',
```

---

# `@geoarrow/deck.gl-layers`に変更

- `getPosition` にはジオメトリのカラムを直接指定する（なぜこういうデザインなのかは謎...）

```diff
- data: ...,
- getPosition: d => d.position,
+ data: data_points,
+ getPosition: data_points.getChild('geom')!,
```

---

# デモ

- <https://yutannihilation.github.io/maplibre-geoarrow-deckgl-layers/>
  - 10万個の点、1万個の線、1万個の三角形

---

![](/demo1.png)

---

# `@geoarrow/deck.gl-layers` の不満点

- deck.gl を覚えるのが面倒なので MapLibre で完結したい
- 実装を見ると、間にちょっとした変換を挟んでいるので（separate → interleaved）、もしかして改善の余地がある...？

<v-clicks>

→ GeoArrow を MapLibre にレンダリングさせるコードを自分で書こう！

</v-clicks>

---

# MapLibre のカスタムレイヤー

- 参考：[Add a simple custom layer on a globe](https://maplibre.org/maplibre-gl-js/docs/examples/add-a-simple-custom-layer-on-a-globe/)

<Transform :scale="0.55">

![](/maplibre-custom-layer.png)

</Transform>

---

# `CustomLayerInterface`

- 主に実装するのは以下の2つ

```ts
onAdd(
  map: maplibregl.Map,
  gl: WebGLRenderingContext | WebGL2RenderingContext
)

render(
  gl: WebGLRenderingContext | WebGL2RenderingContext,
  options: CustomRenderMethodInput
)
```

---

# `onAdd()`

- GPU のバッファにデータを読み込む

```ts
onAdd(map, gl) {
  ...
  this.buffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, this.buffer);
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array([...]),
      gl.STATIC_DRAW
  );
  ...
}
```

---

# `render()`

- 実際の描画処理
- 公式サイトの例では、シェーダーのコードは `getShader()` という別のメソッドで生成するようにしている

```ts
render(gl, args) {
  const program = this.getShader(gl, args.shaderData);
  gl.useProgram(program);
  ...
```

---

# `render()`

- `onAdd()` で読み込んだバッファなど、必要なデータを参照して、描画する

```ts
  ...
  gl.bindBuffer(gl.ARRAY_BUFFER, this.buffer);
  ...
  gl.drawArrays(gl.TRIANGLE_STRIP, 0, 3);
}
```

---

# 理屈は分かったので、あとは GPU にデータをぶち込むだけ！

<v-clicks>

- ...と思うじゃないですか？

</v-clicks>

---
layout: image
image: "/figure04-01.jpg"
---

---
layout: image
image: "/figure04-02.jpg"
---

---

# GPU

- GPU はポリゴンをそのまま描いたりできない
- もっと細かい要素、具体的には三角形に分解する必要がある（**triangulation**）

---
layout: image
image: "/figure04-03.jpg"
---

---
layout: section
---

# こんな時の定番<br/>その名も...

---
layout: image
image: "/earcut01.jpg"
---

---
layout: image
image: "/earcut02.jpg"
---

---

# Earcut

- Mapbox が開発した三角形分割用のライブラリ
- 耳なし芳一を連想させるインパクトある名前
- 軽量（3kB）
- 色んな言語やライブラリにポーティングされている定番アルゴリズム

---

# Earcut

- 実は `@geoarrow/deck.gl-layers` も earcut を実装して三角形分割をやっている
- ウェブワーカーで処理することでメインスレッドをブロックしない、というテクい実装
  - 真似してみようとしたけど、自分の技術力ではよくわからず...

---

# あんまりわからなかったところ

- ポリゴン：earcut で切り刻む
- 点：SDF で描く
- 線：？

---

# シェーダー芸感を出すためだけのデモ

- <https://yutannihilation.github.io/maplibre-geoarrow-test/>
- 注：重いです
- ポリゴンをクリックすると波紋みたいなエフェクトが出ます

---

# シェーダー芸感を出すためだけのデモ

<Transform :scale="0.55">
<SlidevVideo v-click autoplay controls>
  <source src="/maplibre-shader.mp4" type="video/mp4" />
</SlidevVideo>
</Transform>

---

# まとめ

- GeoArrow というデータフォーマットがある
- ウェブ地図のエコシステムは、まだまだ GeoArrow を使うには早すぎるかも
  - `@geoarrow/deck.gl-layers` はまあまあ使えそう
- その気になれば MapLibre でシェーダー芸もできる！
