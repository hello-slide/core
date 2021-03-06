# 使い方（Dev）

**注意: このドキュメントは開発者が記憶喪失になってしまった場合用です。**

## 1. 事前に準備するもの

### 1.1. クラウド編

- GKE
- Google Cloud Firestore
- [GCPサービスアカウント](https://cloud.google.com/iam/docs/creating-managing-service-accounts?hl=ja)
  - 鍵を作成してjsonをダウンロードしておく
  - Cloud FirestoreとSecret Managerの接続に使用します。権限を追加しておいてください。

### 1.2. ローカル編

- Dapr
- gcloud
- kubectl
  - gcloudと連携済み
- helm

## 2. セットアップ

### 2.1. k8sクラスタ切り替え

```bash
# 一覧
gcloud container clusters list

# 切り替え
gcloud container clusters get-credentials [name] --zone=$ZONE

# 切り替え例
gcloud container clusters get-credentials hello-slide-api-dev --zone=asia-northeast1-a
```

[kubectl でのクラスタの切り替え設定](https://qiita.com/sonots/items/f82912367693d717ff06)

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

    - account-manager-secret.env

        ```env
        google_client=********
        google_secret=********
        seed=*****
        ```

    - token-manager-secret.env

        ```env
        public_key="****"
        ```

    - [google iam].json
      - google iamでダウンロードしたjson

    ```bash
    # Firestore, Cloud Secret用
    kubectl create secret generic google-secret --from-env-file google-secret.env

    # account-manager用
    kubectl create secret generic account-manager-secret --from-env-file account-manager-secret.env

    # token-manager用
    kubectl create secret generic token-manager-secret --from-env-file token-manager-secret.env

    # slide-manager用
    kubectl create secret generic slide-manager-secret --from-file=json=./[google iam].json
    ```

2. Dapr appをデプロイ

    ```bash
    kubectl apply -f ./state/user-data.yaml
    kubectl apply -f ./state/user-email.yaml
    kubectl apply -f ./state/slide-config.yaml

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

    # Dpar接続
    kubectl apply -f ./state/redis.yaml
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
kubectl apply -f ./dev/components/token-manager.yaml
kubectl apply -f ./dev/components/account-manager.yaml
kubectl apply -f ./dev/components/slide-manager.yaml

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
    - [`state/user-data.yaml`](../../state/user-data.yaml)
    - [`state/user-email.yalm`](../../state/user-email.yaml)
    - [`state/slide-config.yaml`](../../state/slide-config.yaml)
  - Redis on k8s
    - [`state/redis.yaml`](../../state/redis.yaml)

## 4. 参考文献等

- [GKEでkubernetesを理解する](https://qiita.com/ntoreg/items/74aa6de2f8f29b4a3b79)
- [Hello Kubernetes](https://github.com/dapr/quickstarts/tree/master/hello-kubernetes)
