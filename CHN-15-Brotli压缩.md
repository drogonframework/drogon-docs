[English](ENG-15-Brotli) | [简体中文](CHN-15-Brotli压缩)

## Brotli压缩

如果存在相应的brotli压缩文件，Drogon开箱即用地支持这些brotli静态压缩文件。

例如，请求“/path/to/asset.js”时，Drogon将搜索“/path/to/asset.js.br”。

它通过在“config.json”文件中默认将“br_static”设置为“true”来实现。

如果你想用brotli动态压缩，你必须在'config.json'中将“use_brotli”设置为“true”。

不打算使用brotli静态压缩的用户，可能希望摆脱brotli额外的“同名文件检查”，可通过在“config.json”中将“br_static”设置为“false”。

# 16 [协程]](CHN-16-协程)
