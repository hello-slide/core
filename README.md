<div aligin="center">
  <img src="./assets/logo.svg">
</div>
<h1  align="center">Core</h1>

## Overview

HelloSlide APIではDaprを使用した分散アプリケーションで構築されています。\
これらのAppはそれぞれリポジトリごとに分かれており、Daprを使用してHTTPで通信します。

このアプリケーションはGKE上にデプロイされています。

## Applications

- [account-manager](https://github.com/hello-slide/account-manager)
  - アカウント作成、削除などの操作を行います。
- [token-manager](https://github.com/hello-slide/token-manager)
  - PASETOを使用したトークンの作成、検証を行うDaprアプリケーションです。
- [slide-manager](https://github.com/hello-slide/slide-manager)
  - スライドの作成、編集、削除を担当するDaprアプリケーションです。

## How To

[here](./documents/howto.md)

## LICENSE

[MIT](./LICENSE)
