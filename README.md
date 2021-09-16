<div aligin="center">
  <img src="./assets/logo.svg">
</div>
<h1  align="center">Core</h1>

## Overview

HelloSlide APIでは[Dapr](https://dapr.io/)を使用してマイクロサービスとして構築されています。\
これらのAppはそれぞれリポジトリごとに分かれており、Daprを使用してgRPCで通信します。

このアプリケーションはGKE上にデプロイされています。

## Applications

- [account-manager](https://github.com/hello-slide/account-manager)
  - アカウント作成、削除などの操作を行います。
- [token-manager](https://github.com/hello-slide/token-manager)
  - PASETOを使用したトークンの作成、検証を行うDaprアプリケーションです。
- [slide-manager](https://github.com/hello-slide/slide-manager)
  - スライドの作成、編集、削除を担当するDaprアプリケーションです。
- [synchronous-controller](https://github.com/hello-slide/synchronous-controller)
  - リアルタイムなスライドを実現するためのめちゃくちゃすごい（自分調べ）Dapr appです。

## How To

[デプロイ方法](./documents/howto.md)

## LICENSE

[MIT](./LICENSE)
