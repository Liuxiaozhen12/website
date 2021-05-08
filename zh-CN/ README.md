<!-- markdownlint-disable-file MD033 -->
<!-- markdownlint-disable-file MD041 -->

<p align="center">
  <img src="logo.svg" width="256" alt="Chaos Mesh Logo" />
</p>
<h1 align="center">网址</h1>
<p align="center">
  使用 <a href="https://v2.docusaurus.io/" target="_blank">Docusaurus 2</a>构建，一个现代静态网站生成器。
</p>

[![Gitpod 已准备对代码使用](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/chaos-mesh/website)

## 如何开发

```sh
yarn # 安装代码
yarn 开始
```

此命令启动本地开发服务器并打开浏览器窗口。 大多数更改都在实时反射，无需重新启动服务器。

## 构建...

```sh
yarn building
```

此命令生成静态内容到 `构建` 目录，并且可以使用任何静态内容托管服务。

## 新版本

```sh
yarn docusaurus docs:version x.x.x
```

所有文档的版本分为两部分， 一个是最新的 **( `docs/`)** 其他版本是 **版本( `versioned_docs/`)** 当版本发布后，最新的 `文档/` 将被复制到 `versioned_docs/` (通过运行上述命令)。

## 如何贡献

您主要只需要修改 `docs/`中的内容 但如果您想要某些版本作为最晚版本生效， 也请在 `versioned_docs/` dir中更新相同的文件。

## 许可协议

与 [Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh) 相同。
