<<<<<<< HEAD
##### Other languages: [简体中文](/CHN/CHN-15-Brotli压缩)
=======
##### Other languages: [简体中文](/CHN/CHN-15-Brotli压缩)
>>>>>>> 557192c59e37288b16a0d6dd1d9116101f5a08e1

## Brotli Info

Drogon supports out of the box brotli statically compressed files if it finds next to asset the corresponding brotli compressed asset.

So for instance, Drogon will search `/path/to/asset.js.br` for the request `/path/to/asset.js`.

It does so by setting `br_static` to `true` by default in `config.json` file.

if you want to dynamically compress with brotli you'll have to set `use_brotli` to `true` in `config.json`.

Users who don't intend to use brotli static, might want to get rid of brotli extra 'sibling check'
by setting `br_static` to `false` in `config.json`.

<<<<<<< HEAD
# 16 [Coroutines](/ENG/ENG-16-Coroutines)
=======
# 16 [Coroutines](/ENG/ENG-16-Coroutines)
>>>>>>> 557192c59e37288b16a0d6dd1d9116101f5a08e1
