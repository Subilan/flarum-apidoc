# Flarum RESTful API

这是一个非官方的 Flarum RESTful API 文档。

## 前言

由于 Flarum 官方对其 RESTful API 有关的介绍停留在了 2017 年的这篇 [api.md](https://github.com/flarum/flarum.github.io/blob/20322c0e6011e4f304ae7e95f41594a0b086bc27/_docs/api.md)，信息严重不足，使该 API 的使用有很大的困难。最近我正在制作 Minecraft 插件 [FlarumReader](https://github.com/sotapmc/FlarumReader)，非常需要对 API 进行全面了解。考虑到这方面的信息匮乏，于是想要把自己做插件途中摸索出的该 API 的各种细节写下来。

该文档的内容建立在实际应用上，即经过我测试可以使用的方法和接口会就被分类、规范地在这里排列出来，因此你能看到的内容都是可以直接实践的。

#### 关于 RESTful API

本 API 文档所指的 API 是 Flarum 的 RESTful API，通过 `POST`、`GET` 等 HTTP 请求进行交互。在阅读之前，有必要了解关于 *RESTful* 的一些信息：

- RESTful 强调**状态转化（state transfer）**，这意味着每个行为的动词都应该直接体现在 HTTP 请求方式里。这一点，Flarum 做到了，而且做得很好。简而言之，那就是如果你想要「获取」信息，那么就用 `GET` 方法，返回值是你的目标信息；如果你想要「发送」信息，那么就用 `POST` 方法，返回值是其结果。除此之外，还有 `PATCH` 和 `DELETE` 两个方法可以投入使用。
- RESTful 这个单词是由 **REST** 加上形容词后缀 *-ful* 所构成的。REST 实际上是指 *Representational State Transfer* 表现层状态转换，实际上就是一种 API 的**设计风格**。

## 适用版本

这方面我无从探索。但是根据我的猜测，实际上这个 API 应该自那篇 2017 年的文档出现以后就没有太大的改变。

## 协议

没有协议。但是复制引用请注明出处，谢谢。
