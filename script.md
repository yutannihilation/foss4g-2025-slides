Hi, I'm Hiroaki Yutani. I'm a software engineer at MIERUNE, Inc. MIERUNE is a GIS company in the Northern part of Japan.

Okay, let me start with this question. Do you like GeoJSON?

GeoJSON has its goods and bads. One good part is that GeoJSON is a text format. But the bad part is also that GeoJSON is a text format. Yeah, I mean, talking about GeoJSON is almost like talking about the advantages and disadvantages of a text format.

The good thing about a text format is that many tools support it. Especially, JSON is very portable. Web browsers can parse JSON, so it's easy to load GeoJSON data on the web. Also, a text format is human-readable. I find it especially useful when I check the diffs on GitHub.

However, GeoJSON is bad in terms of efficiency. We have parsing overhead. We have writing overhead. I mean, since it's a text format, it needs to be parsed when we want to load it into some system. Also, it needs to be written back to text when we want to pass it to another system. Its file size is also a headache.

If we use GeoJSON, we hit these overheads several times. Let's think about a simple web application. We have a database, an application server, and a web browser. So, what happens if we want to pass some GIS data as GeoJSON?

First, while the database stores the data as binary, it needs to convert the data into GeoJSON. If you are using PostGIS or DuckDB, this is typically done by ST_AsGeoJSON.

Next, the app server receives the GeoJSON, and the server needs to parse the text into binary again. If this is a Node.js server, it's probably JSON.parse or something.

The same kind of conversion happens between the web server and the web browser.

Even in this simple example, we had many round-trip conversions. We converted a binary to a text, and the text to a binary, and the binary to a text, and the text to a binary. As you can imagine, this can be worse if the system is more complicated.

I know this isn't always a problem. This is fine as long as the data is small. However, the overhead does matter on larger data. Can we get rid of such overhead?

Yes! We can use GeoArrow! GeoArrow can travel from end to end without any unnecessary conversions.

So, what's GeoArrow? GeoArrow is a specification for storing geospatial data in Apache Arrow. What is Apache Arrow? Apache Arrow, or Arrow, is a columnar memory format for general purpose. I mean, not only for geospatial data.

Arrow is designed for zero-copy reads, which means it's ready to be used without any conversion. In comparison, GeoJSON is not zero-copy. It needs to be converted from text to binary. This conversion process means there are two copies: the original text and the newly parsed binary data.

Arrow aims for cross-language interoperability. It provides libraries and tools for many programming languages so that Arrow data can be universally read and written.

The idea is very simple. We have many tools and programming languages. They use different data representation in memory, so we need to convert data from one format to another if we want to pass the data between them. By using Arrow, we need no conversions!!! Because many tools and languages can read and write Apache Arrow. In reality, we still need some conversions, or copies, but it's much better.

That was the point of Arrow. So, what's the relationship between GeoArrow and Arrow? Arrow defines the actual memory layouts. GeoArrow defines what a geometry is represented as using Arrow. So, Arrow is a very low-level thing, but the GeoArrow specification is simple if you are familiar with GeoArrow.

Let's look at the specification a bit. A coordinate is represented in either of the two forms, separated or interleaved. It might not be very obvious how different they are.

In the separated representation, the data consists of two to four arrays for X, Y, Z, and M each. On the other hand, the interleaved representation uses one single array. As this is a bit too low-level a thing, I won't go into the detail. In short, the specification says the separated form is recommended for most use cases, but the interleaved form is still useful for some cases. For example, passing data to GPU buffers.

Now that we have covered the coordinate definition, the rest is simple. A point is just a coordinate. A line string is an array of coordinates. A polygon is an array of arrays of coordinates. ... and so on!

Okay, let's move on. Thanks to GeoArrow, we can move data from end to end without overhead. But, this is not all. If we use the data carefully, the GeoArrow data can reach your GPU.

You might wonder why the GPU matters here. This is because modern map rendering libraries rely on the GPU. Typically via WebGL or WebGPU.

Lonboard is a great example of how powerful GeoArrow is. Lonboard is a Python library to visualize geospatial data. It uses GeoArrow to pass data from Python to JavaScript, and then to the GPU via Deck.gl. This is possible because Deck.gl has a low-level interface that accepts binary arrays directly.

Thanks to these technologies, Lonboard handles GIS data very efficiently. You can smoothly visualize very large data that other tools cannot handle well.

Perfect! On Python, you have many data sources that can transfer data in the Arrow memory format. For example, PostGIS and DuckDB. You can use GeoPandas to read various formats of geospatial data. Then, you can pass the data to deck.gl. Perfect!

...if you are using Python.

What if you're not using Python? Especially, what if it's a web application?

Fortunately, there's a JavaScript library to render GeoArrow data on deck.gl: geoarrow/deck.gl.layers.

This provides the GeoArrow version of Deck.gl's layers. The API is almost the same as the original one. So, this is very easy to use if you are familiar with Deck.gl.

Have you ever used DuckDB WASM? DuckDB Wasm and this library are a perfect combo. DuckDB WASM can read remote GeoParquet files and output GeoArrow. So, this is awesome if you want to do everything on web browsers.

But..., what if you want to process data on servers instead of web browsers? Or, what if the output is not Deck.gl? For example, MapLibre, Leaflet, Cesium, or whatever.

We have challenges on the web.

I think there are two categories of challenges. The first one is about DB connectivity. The second one is about map rendering libraries.

Let's talk about the first one. In order to connect to a database, we need either a database connector or a SQL gateway.

Database connectors. If you are familiar with databases, you probably have heard of JDBC and ODBC. But, how about ADBC? ADBC stands for "Arrow DataBase Connector".

ADBC is really cool! It transfers data in the Arrow memory format, so this is a perfect fit for our use case. It supports many languages: C/C++, C#/.NET, Go, Java, Python, R, and Rust. It provides many database drivers: DuckDB, PostgreSQL, SQLite, Bigquery, Snowflake, and more. It's also easy to manage using dbc CLI. By using dbc, you can install and manage drivers via command line!

However..., ADBC is not available for JavaScript / TypeScript... So, there's no hope for Node.js servers. Note that Prisma is a popular solution for Node.js, but it only returns JSON, so it's not suitable for large geospatial data.

Another solution is SQL gateways. By SQL gateways, I mean services that provide a REST API for executing queries. For example, PostgREST is a service that exposes a REST API for a PostgreSQL database. There are a few other SQL gateways.

However..., as far as I know, their output is either JSON or CSV... The only exception is ROAPI. Anyway, it seems SQL gateways are not designed for large data.

Okay, next topic. Map-rendering libraries. There are many map-rendering JavaScript libraries. Does any of these support GeoArrow? What do you think? We already know about the status of Deck.gl. There is a great NPM package to support GeoArrow. So, in other words, GeoArrow is not supported natively. At least at the moment.

So, how about other libraries? In order to investigate, I searched the word "geoarrow" on the GitHub repositories of MapLibre, MapBox, Cesium, and Leaflet. Can you guess how many issues were hit?

The answer is... zero. There were no single issue mentioning GeoArrow.

Why???

Is it simply because it's technically impossible?

No. It might be difficult, but I believe it's possible. Here's my proof of concept to draw a GeoArrow polygon on MapLibre GL JS using a custom shader. This is a very simple example drawing only one polygon with a bit of fancy interaction.

So, again, why?

I don't know. I've been thinking about it for months, while preparing this presentation, but I haven't found an answer so far. Maybe it's just that GeoArrow is not so popular enough? If so, I hope my talk today contributed to spread the word a bit.

Okay, let me summarize.

First, I want to emphasize that GeoArrow is so cool in that it allows us to handle large geospatial data efficiently. There are already great tools powered by GeoArrow such as Lonboard and @geoarrow/deck.gl-layers. Actually, you might be already using GeoArrow without knowing the name. But, on the web, the support for GeoArrow is surprisingly small. I hope the situation will be better in future.