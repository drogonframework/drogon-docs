##### Other languages: [简体中文](CHN-15-Brotli压缩)

## Brotli Info

Drogon supports out of the box brotli statically compressed files if it finds next to asset the corresponding brotli compressed asset.

So for instance, Drogon will search `/path/to/asset.js.br` for the request `/path/to/asset.js`.

It does so by setting `br_static` to `true` by default in `config.json` file.

if you want to dynamically compress with brotli you'll have to set `use_brotli` to `true` in `config.json`.

Users who don't intend to use brotli static, might want to get rid of brotli extra 'sibling check'
by setting `br_static` to `false` in `config.json`.

# 16 [Coroutines](ENG-16-Coroutines)
