# GKEのSSL化

[Google マネージド SSL 証明書の使用](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs)

公開したいPodがServiceになっている必要があります。

## 1. 静的IPアドレスの予約

[コンソール](https://console.cloud.google.com/networking/addresses/add)

## 2. マニフェスト適用

```bash
kubectl apply -f ./ssl/hello-slide-api.yaml
```

## 3. Ingress作成

```bash
kubectl apply -f ./ssl/hello-slide-api-ingress.yaml

# 確認
kubectl get ingress
```

## 4. 証明書のプロビジョニング

```bash
# 確認（プロビジョニングされるまで最大60分ほどかかる）
kubectl describe managedcertificate hello-slide-api
```

```yaml
Status:
  Certificate Name:    []
  Certificate Status:  Active # ここがActiveになったらOK
  Domain Status:
    Domain:     [domain]
    Status:     Active
```

## 5. 確認

```bash
curl https://api.hello-slide.jp/account
```
