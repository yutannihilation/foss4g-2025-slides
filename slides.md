---
title: GeoArrow on Web; Can We Live Without GeoJSON?
theme: default
fonts:
  sans: Noto Sans JP
  mono: Fira Code
  weights: 600,900
class: text-center
drawings:
  persist: false
mdc: true
seoMeta:
  ogImage: /title.png
---

# <span color="blue">GeoArrow</span> on Web <br/> Can We Live Without <span color="orange">GeoJSON</span>?

Hiroaki Yutani
(MIERUNE Inc.)

---

# Who Am I?

- Software engineer at MIERUNE Inc.
- Occasionally contriubutes to various OSS
  - SedonaDB
  - DuckDB-spatial
  - ggplot2 \(R\)

<div class="absolute right-30px bottom-30px">
  <img src="/icon.jpg" class="h-[200px]" />
</div>


---
layout: section
---

# Do you like GeoJSON?

---

# GeoJSON

## Goods

<v-clicks>

- Text format

</v-clicks>


## Bads

<v-clicks>

- Text format

</v-clicks>

---

# GeoJSON = text format

## Goods

- Supported by many tools
  - Web browsers can parse JSON!
- Human-readable

## Bads

- Parsing / writing overhead
- Size bloat

---
layout: image
image: "/figure01-01.jpg"
---

# Typical web application

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

# Many round-trips between text and binary

- DB converts binary to text by `ST_AsGeoJSON()`
- Application server parses it (e.g. `JSON.parse()`) to JavaScript values
- Then, application server converts it to text
- Web browser parses it to JavaScript values

---

# Many round-trips between text and binary

- This is fine on small data
- However, the overhead matters on larger data

<v-clicks>

- Can we get rid of such overhead...?

</v-clicks>


---

# Yes!

---
layout: image
image: "/figure01-04.jpg"
---

# Yes!

---
layout: image
image: "/figure01-05-1.jpg"
---

# Yes!

---
layout: section
---

# GeoArrow

---

# What is GeoArrow?

- GeoArrow is a specification for storing geospatial data in **Apache Arrow**

<div class="absolute right-30px bottom-30px">
  <img src="https://geoarrow.org/geoarrow_logo.png" class="h-[200px]" />
</div>

---

# What is Apache Arrow?

- Columnar **memory format** for general purpose
- Designed for **zero-copy reads**
- Cross-language interoperability (C++, Rust, Java, Python, R...)

<div class="absolute right-30px bottom-30px">
  <img src="https://arrow.apache.org/img/arrow-logo_horizontal_black-txt_transparent-bg.png" class="h-[200px]" />
</div>

---
layout: image
image: "/figure02-01.jpg"
---

---
layout: image
image: "/figure02-02.jpg"
---

---

# GeoArrow & Arrow

- Arrow defines **the actual memory layouts**
- GeoArrow defines **what a geometry is represented as using Arrow**
- GeoArrow specification is simple if you are familiar with GeoJSON.

---

# GeoArrow specifications

- A coordinate is either

Separated representation:
```
Struct<
  x: double,
  y: double,
 [z: double,
 [m: double]]>
```

Interleaved representation:
```
[double, double, [double, [double]]]
```

---

# GeoArrow specifications

- A point is  
  `Coordinate`
- A linestring is  
  `[Coordinate, Coordinate, ...]`
- A polygon is  
  `[[Coordinate, Coordinate, ...], ...]`

...

---
layout: image
image: "/figure01-05-1.jpg"
---

---
layout: image
image: "/figure01-05-2.jpg"
---

# GeoArrow can reach your GPU!

---
layout: image
image: "/figure01-05-2.jpg"
---

<style>
div.slidev-page.slidev-page-27 p {
  color: black;
}

div.slidev-page.slidev-page-27 div {
    background-color: rgba(255,255,255,0.75);
    background-blend-mode: lighten;
}

</style>

# GeoArrow can reach your GPU!

(I'm sorry to say that a recent FOSS4G workshop made me rethink this part, but let's not talk about it for now...)

---

# Why does GPU matter?

- Because GPU renders maps
  - Deck.gl
  - MapLibre
  - Cesium
  - ...

---

# Lonboard

![](/lonboard.png)

---

# Lonboard

- Use GeoArrow between Python and JavaScript (and GPU via Deck.gl)
- Deck.gl accepts binary data!

---
layout: image
image: "/figure03-00-1.jpg"
---

# Perfect!

---
layout: image
image: "/figure03-00-1.jpg"
---

# Perfect! ...if you are using Python

---
layout: image
image: "/figure03-00-2.jpg"
---

# What if it's a web application?

---

# `@geoarrow/deck.gl-layers`

![](/deck-gl-layers.png)

---

# `@geoarrow/deck.gl-layers`

- Render GeoArrow data on Deck.gl
- Easy to use; the same API as Deck.gl itself
- DuckDB WASM + this is a perfect combo!
- This is awesome if you want to do everything on web browsers

---
layout: image
image: "/figure03-00-2.jpg"
---

# What if we want backend servers?

---
layout: image
image: "/figure03-00-3.jpg"
---

# What if the output is NOT Deck.gl?

---
layout: section
---

# Challenges on the web

---
layout: image
image: "/figure01-01.jpg"
---

---
layout: image
image: "/figure03-01.jpg"
---

# 1. Lack of **DB Connectivity** from the application server

---
layout: image
image: "/figure03-02.jpg"
---

# 2. Lack of support from **map rendering JavaScript libraries**


---

# Challenges on the web

- Lack of **DB Connectivity** from the application server
- Lack of support from **map rendering JavaScript libraries**

---

# DB Connectivity

To connect a database, we need either

- a database connector
- a SQL gateway

---

# Database connectors

- JDBC
- ODBC
- **ADBC** (Arrow DataBase Connector) 

---

# ADBC is cool!

<v-clicks>

- Transfers data in Arrow memery format
- Supports many languages: C/C++, C#/.NET, Go, Java, Python, R, Rust
- Provides many drivers: DuckDB, PostgreSQL, SQLite, Bigquery, Snowflake, ...
- Easy to manage using `dbc` CLI

</v-clicks>

---
layout: image
image: "/figure04-01.jpg"
---

---


# But...

- ADBC is NOT available for JavaScript / TypeScript...

<v-clicks>

- (To be fair, we can use other languages. But, it's not handy)

</v-clicks>

---

# SQL Gateways

- Provides HTTP API for executing SQLs in a database

---
layout: image
image: "/figure04-02.jpg"
---

---

# SQL Gateways

- The output format is either JSON or CSV...
- No one outputs Arrow except for ROAPI

---

# Map rendering JavaScript libraries

- MapLibre GL JS
- Mapbox GL JS
- CesiumJS
- Leaflet
- Deck.gl
- ...

---

# Map rendering JavaScript libraries

- I searched "geoarrow" on the GitHub repo of MapLibre GL JS, Mapbox GL JS, CesiumJS, and Leaflet.
- Can you guess how many issues were hit?

<v-clicks>

- **No single issue** mentions GeoArrow!😭

</v-clicks>

---

# Why?

<v-clicks>

- Is it technically impossible?

</v-clicks>

---

# No

<div class="z-10">

- This is my proof-of-concept to draw GeoArrow polygon on MapLibre using custom shader.  
    
  <https://yutannihilation.github.io/maplibre-geoarrow-test/>

</div>

<div class="absolute right-0 bottom-40 h-full scale-80 z--1">
<SlidevVideo autoplay controls>
  <source src="/maplibre-shader.mp4" type="video/mp4" />
</SlidevVideo>
</div>

---

# Why?

- ~~Is it technically impossible?~~

<v-clicks>

- I don't know. Maybe GeoArrow is not popular enough yet...?

</v-clicks>

---
layout: section
---

# Summary

---

# Summary

- **GeoArrow** allows us to handle large geospatial data efficiently
- There are already great tools powered by GeoArrow such as **Lonboard** and **@geoarrow/deck.gl-layers**
- Yet, on the web, GeoArrow is not supported

---
layout: section
---

# A few more words...

---
layout: image
image: "/figure01-05-2.jpg"
---

# This isn't really accurate because...

---
layout: image
image: "/figure05-01.jpg"
---

# GPU cannot render polygons!

---
layout: image
image: "/figure05-02.jpg"
---

# GPU cannot render polygons!

---
layout: image
image: "/figure05-03.jpg"
---

# We need to split it into triangles (This is called **tessellation**)

---
layout: section
---

# Tessellation is horrifying, as evidenced by the fact that its most famous algorithm is named...

---
layout: image
image: "/earcut01.jpg"
---

---
layout: image
image: "/earcut02.jpg"
---

---

# Tessellation

- No, I'm kidding
- It's not horrifying
- I just wanted to say tessellation can be horrifyingly costful!

<v-clicks>

- Not all geometry require tessellation
  - Points are fine
  - Lines...?

</v-clicks>


---

# Remaining question about GeoArrow

- GeoArrow itself doesn't take care of tessellation
- So, the data needs to be tessallated before passing to GPU anyway
- **Where should tessellation happen? On backend servers? Or, on web browsers?**

---

# MLT (MapLibre Tiles)

- "Next generation vector tiles format," which supersedes MVT
- MLT can contain **pre-tessellated** polygons!
- If the tile server (Martin) generates MLT on the fly, this might be the most performant option!

---

# Updated summary

- **GeoArrow** allows us to handle large geospatial data efficiently
- There are already great tools powered by GeoArrow such as **Lonboard** and **@geoarrow/deck.gl-layers**
- Yet, on the web, GeoArrow is not supported
- **Some pre-tessellated format might be better than GeoArrow, and MLT can be the one**


---

# References

- GeoArrow: https://geoarrow.org/
- Lonboard: https://developmentseed.org/lonboard/
- `@geoarrow/deck.gl-layer`: https://geoarrow.org/deck.gl-layers/
- PoC: https://yutannihilation.github.io/maplibre-geoarrow-test/