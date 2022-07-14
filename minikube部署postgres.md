# minikube 部署 postgres

參考文章：
[ref](https://severalnines.com/blog/using-kubernetes-deploy-postgresql/)
[ref](https://markgituma.medium.com/kubernetes-local-to-production-with-django-3-postgres-with-migrations-on-minikube-31f2baa8926e)

---

### Create Namespace

首先，不要污染其他的 k8s pod，我們創建一個 namespace，並且切到指定的 namespace

```bash
# [your-namespace] = my-space
# 創造一個namespace
$ kubectl create namespace [your-namespace]
# 取得所有namespace
$ kubectl get ns
# 目前的namespace為：
$ kubectl config view | grep namespace:

# 切到指定namespace
$ kubectl config set-context --current --namespace=[your-namespace]
```

### 依序創建服務

用 secret 取代 configmap

```bash
# Create config map
# kubectl create -f postgres-configmap.yaml
# Create secret
$ kubectl create -f postgres-secret.yaml
```

```bash
# Create PV, PVC
$ kubectl create -f postgres-storage.yaml
persistentvolume "postgres-pv-volume" created
persistentvolumeclaim "postgres-pv-claim" created
```

```bash
# Create deployment
$ kubectl create -f postgres-deployment.yaml
deployment "postgres" created
```

```bash
# Create Service
$ kubectl create -f postgres-service.yaml
service "postgres" created
```

### 確認服務狀態

```bash
# Get Service
$ kubectl get svc postgres
```

```bash
$ kubectl get all
```

#### [重要]取得 minikube 怎麼連到 postgres

```bash
minikube service postgres -n my-space
|-----------|----------|-------------|---------------------------|
| NAMESPACE |   NAME   | TARGET PORT |            URL            |
|-----------|----------|-------------|---------------------------|
| my-space   | postgres |        5432 | http://192.168.64.5:32394 |
|-----------|----------|-------------|---------------------------|
```

接著就可以用連線軟體，連進去 postgres db 了

```bash
$ psql -h 192.168.64.5 -U postgresadmin --password -p 32394 postgresdb
Password for user postgresadmin1:
psql (10.4)
Type "help" for help.

postgresdb=#
```

### delete all service

```
kubectl delete service postgres
kubectl delete deployment postgres-deployment
# kubectl delete configmap postgres-config
kubectl create -f postgres-secret.yaml
kubectl delete persistentvolumeclaim postgres-pv-claim
kubectl delete persistentvolume postgres-pv-volume
```
