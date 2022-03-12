# 使い方

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

### 2.1. k8sクラスタ作成

```bash
gcloud container clusters create [name] --num-nodes=1
```

or

GCPコンソールから

```bash
# クラスタ リスト
gcloud container clusters list

# GCPコンソールから追加したクラスタをkubectlと接続
gcloud container clusters get-credentials [name]
```

### 2.2. Dapr Appのデプロイ

```bash
dapr init -k --enable-ha=true --wait

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
        client_x509_cert_url="********"
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

    - sql-secret.env

        ```env
        user=****
        password=****
        name=****
        ```

    - [google iam].json
      - google iamでダウンロードしたjson
    - [google iam].json
      - cloud sql用 Google IAM

    ```bash
    # Firestore, Cloud Secret用
    kubectl create secret generic google-secret --from-env-file google-secret.env

    # account-manager用
    kubectl create secret generic account-manager-secret --from-env-file account-manager-secret.env

    # token-manager用
    kubectl create secret generic token-manager-secret --from-env-file token-manager-secret.env

    # slide-manager用
    kubectl create secret generic slide-manager-secret --from-file=json=./[google iam].json

    # synchronous-controller
    kubectl create secret generic synchronous-controller-iam --from-file=service_account.json=./[google iam].json
    kubectl create secret generic sql-secret --from-env-file sql-secret.env
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
kubectl apply -f ./components/token-manager.yaml
kubectl apply -f ./components/account-manager.yaml
kubectl apply -f ./components/slide-manager.yaml
kubectl apply -f ./components/synchronous-controller.yaml

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
    - [`state/user-email.yalm`](../state/user-email.yaml)
    - [`state/slide-config.yaml`](../state/slide-config.yaml)
  - Redis on k8s
    - [`state/redis.yaml`](../state/redis.yaml)

## 4. 参考文献等

- [GKEでkubernetesを理解する](https://qiita.com/ntoreg/items/74aa6de2f8f29b4a3b79)
- [Hello Kubernetes](https://github.com/dapr/quickstarts/tree/master/hello-kubernetes)
