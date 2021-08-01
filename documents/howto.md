# 使い方

**注意: このドキュメントは開発者が記憶喪失になってしまった場合用です。**

## 1. 事前に準備するもの

### 1.1. クラウド編

- GKE
- Google Cloud Firestore
- [GCPサービスアカウント](https://cloud.google.com/iam/docs/creating-managing-service-accounts?hl=ja)
  - 鍵を作成してjsonをダウンロードしておく
  - Cloud Firestoreの接続に使用します。

### 1.2. ローカル編

- Dapr
- gcloud
- kubectl
  - gcloudと連携済み
- helm

## 2. セットアップ

### 2.1. k8sクラスタ作成

```bash
gcloud container clusters create [name] --num-nodes=1
```

### 2.2. Dapr Appのデプロイ

```bash
dapr init --kubernetes --wait

# status
dapr status -k
```

### 2.2. StateStoreのデプロイ

1. k8s secretにkeyを追加する

    - google-secret.env

        ```env
        private_key_id="********"
        private_key="********"
        email="********"
        client_id="********"
        client_x509_cert_url="="********""
        ```

    ```bash
    # Firestore, Cloud Secret用
    kubectl create secret generic google-secret --from-env-file google-secret.env
    ```

2. Dapr appをデプロイ

    ```bash
    kubectl apply -f ./state/user-data.yaml
    kubectl apply -f ./state/user-email.yaml
    kubectl apply -f ./secret/secret-state.yaml

    # 確認
    dapr components -k
    ```

3. Redisのデプロイ

    ```bash
    # deploy redis
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm install redis bitnami/redis

    # 確認
    kubectl get pods

    # redisのパスワード取得
    kubectl get secret --namespace default redis -o jsonpath="{.data.redis-password}" | base64 --decode

    # redisパスワードをシークレットに追加
    kubectl create secret generic redis-secret --from-literal=password=*********

    # Dpar接続
    kubectl apply -f ./state/redis.yaml

    # もしシークレットが存在している場合
    kubectl delete secret redis-secret
    ```

    ```bash
    # redis 接続
    kubectl run --namespace default redis-client --restart='Never'  --env REDIS_PASSWORD=****  --image docker.io/bitnami/redis:6.2.5-debian-10-r0 --command -- sleep infinity

    # 接続する。（参考: https://qiita.com/MahoTakara/items/a14547166522d3e113a0#redis-cli%E3%81%8B%E3%82%89%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%97%E3%81%A6%E5%8B%95%E4%BD%9C%E7%A2%BA%E8%AA%8D）

    kubectl exec --tty -i redis-client --namespace default -- bash
    > redis-cli -h redis-master -a $REDIS_PASSWORD
    > (redis-cli -h redis-replicas -a $REDIS_PASSWORD)
    >> keys *
    ```

### 2.3. Appのデプロイ

```bash
kubectl apply -f ./components/token-manager.yaml
kubectl apply -f ./components/account-manager.yaml

# 確認
dapr list -k

# ログ確認
dapr logs -k -a [app-id]
```

### 2.4 外部公開とSSL化

[here](./ssl.md)

## 3. Dapr App一覧

- State Store
  - Firestore
    - [`state/user-data.yaml`](../state/user-data.yaml)
    - [`state/login-token.yalm`](../state/login-token.yaml)
- Secret Store
  - Google Secret Manager
    - [`state/secret-state.yaml`](../secret/secret-state.yaml)

## 4. 参考文献等

- [GKEでkubernetesを理解する](https://qiita.com/ntoreg/items/74aa6de2f8f29b4a3b79)
- [Hello Kubernetes](https://github.com/dapr/quickstarts/tree/master/hello-kubernetes)
