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

    ```bash
    # Firestore用
    kubectl create secret generic firestore-secret --from-literal=private_key_id=*********
    kubectl create secret generic firestore-secret --from-literal=private_key=*********
    kubectl create secret generic firestore-secret --from-literal=email=*********
    kubectl create secret generic firestore-secret --from-literal=client_id=*********
    ```

2. firestore接続用dapr appをデプロイ

    ```bash
    kubectl apply -f ./state/user_data.yaml
    ```

### 2.3. Appのデプロイ

## 3. Dapr App一覧

- State Store
  - Firestore
    - [`state/user_data.yaml`](../state/user_data.yaml)

## 4. 参考文献等

- [GKEでkubernetesを理解する](https://qiita.com/ntoreg/items/74aa6de2f8f29b4a3b79)
- [Hello Kubernetes](https://github.com/dapr/quickstarts/tree/master/hello-kubernetes)
