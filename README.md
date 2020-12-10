# laravel-mysql-kubernetes-demo

---

## Docker イメージを用意

```
.
├── README.md
├── docker
│   ├── app
│   │   └── Dockerfile
│   └── web
│       ├── Dockerfile
│       └── nginx.conf
└── docker-compose.yaml

3 directories, 5 files
```

```sh
docker-compose build --no-cache
docker images

docker push azum/laravel-mysql-app
docker push azum/laravel-mysql-web
```

## Minikube を用意

```sh
minikube start
minikube ssh
```

## persistentVolume 用のディレクトリを切る

```sh
cd /mnt
sudo mkdir mysqlvolume
ls
exit
```

## リソースを作成

```sh
# 起動
kubectl apply -f laravel-mysql-pv.yaml && \
  kubectl apply -f laravel-mysql-pvc.yaml && \
  kubectl apply -f laravel-mysql-deploy-db.yaml && \
  kubectl apply -f laravel-mysql-service-db.yaml && \
  kubectl apply -f laravel-mysql-configmap.yaml && \
  kubectl apply -f laravel-mysql-deploy-app-web.yaml && \
  kubectl apply -f laravel-mysql-service-app.yaml

# 起動確認
kubectl get service
kubectl get pods

minikube service appc

# ログ確認
kubectl describe pods appc-*********-*****
kubectl describe pods dbc-*********-*****

kubectl logs appc-**********-***** -c appc
kubectl logs dbc-**********-***** -c dbc

```

---

## おかたづけ

```sh
kubectl delete -f laravel-mysql-service-app.yaml
kubectl delete -f laravel-mysql-deploy-app-web.yaml
kubectl delete -f laravel-mysql-configmap.yaml
kubectl delete -f laravel-mysql-service-db.yaml
kubectl delete -f laravel-mysql-deploy-db.yaml
kubectl delete -f laravel-mysql-pvc.yaml
kubectl delete -f laravel-mysql-pv.yaml
```
