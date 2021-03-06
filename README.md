# laravel-mysql-kubernetes-demo

---

## Docker イメージを用意

```
.
├── README.md
├── docker
│   ├── app
│   │   ├── Dockerfile
│   │   └── php.ini
│   └── web
│       ├── Dockerfile
│       └── nginx.conf
├── docker-compose.yaml
├── laravel
├── laravel-mysql-configmap.yaml
├── laravel-mysql-deploy-app-web.yaml
├── laravel-mysql-deploy-db.yaml
├── laravel-mysql-ingress.yaml
├── laravel-mysql-pv.yaml
├── laravel-mysql-pvc.yaml
├── laravel-mysql-service-app.yaml
└── laravel-mysql-service-db.yaml

4 directories, 14 files
```

```sh
docker-compose build --no-cache
docker images

docker push azum/laravel-mysql-app
docker push azum/laravel-mysql-web
```

## （ Minikube を使用する場合）クラスターを作成

```sh
minikube start
```

## context を確認・設定

```sh
kubectl config get-contexts
# kubectl config use-context docker-desktop
# kubectl config use-context minikube
# kubectl config set-context $(kubectl config current-context) --namespace=laravel-mysql-namespace
```

## persistentVolume 用のディレクトリを切る

```sh
minikube ssh
```

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
kubectl get pods
kubectl get service # docker-desktop を使っている場合は `minikube ip` で得られる IP アドレスと、この結果の「 PORT(S) 」の : の右側のポート番号
```

```
ctl get podsNAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
appc         NodePort    10.103.74.169    <none>        8080:30529/TCP   17s
dbc          ClusterIP   10.103.143.244   <none>        3306/TCP         18s
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          3d1h
```

```sh
minikube service appc # Minikube を使っている場合はこの結果のホスト名＋ポート
```

```
|-----------|------|-------------|---------------------------|
| NAMESPACE | NAME | TARGET PORT |            URL            |
|-----------|------|-------------|---------------------------|
| default   | appc |        8080 | http://192.168.49.2:31793 |
|-----------|------|-------------|---------------------------|
🏃  Starting tunnel for service appc.
|-----------|------|-------------|------------------------|
| NAMESPACE | NAME | TARGET PORT |          URL           |
|-----------|------|-------------|------------------------|
| default   | appc |             | http://127.0.0.1:46597 |
|-----------|------|-------------|------------------------|
🎉  Opening service default/appc in default browser...
👉  http://127.0.0.1:46597
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```

```sh
# ログ確認
kubectl describe pods appc-*********-*****
kubectl describe pods dbc-*********-*****

kubectl logs appc-**********-***** -c appc
kubectl logs dbc-**********-***** -c dbc

```

## Ingressを作成

```sh
# NGINX Ingress Controller をインストール
# minikube addons enable ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml

# Ingress リソースを作成
kubectl apply -f laravel-mysql-ingress.yaml
```

### ADDRESS がセットされたことを確認

```sh
kubectl get ingress
```

```
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME                    CLASS    HOSTS       ADDRESS     PORTS   AGE
laravel-mysql-ingress   <none>   localhost   localhost   80      75s
```

### ログの確認方法

```sh
kubectl get pods -n kube-system
kubectl logs -n kube-system ingress-nginx-controller-**********-*****
```

## Laravel のマイグレーション

```sh
kubectl get pod
kubectl exec -it appc-cbf76f875-rptjb --container=appc bash
```

```sh
php artisan migrate
```

```
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (4.39ms)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (3.38ms)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (2.68ms)
```

```sh
composer require laravel/ui
```

```
Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0
Using version ^3.1 for laravel/ui
./composer.json has been updated
Running composer update laravel/ui
Loading composer repositories with package information
Updating dependencies
Lock file operations: 1 install, 0 updates, 0 removals
  - Locking laravel/ui (v3.1.0)
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
As there is no 'unzip' command installed zip files are being unpacked using the PHP zip extension.
This may cause invalid reports of corrupted archives. Besides, any UNIX permissions (e.g. executable) defined in the archives will be lost.
Installing 'unzip' may remediate them.
  - Downloading laravel/ui (v3.1.0)
  - Installing laravel/ui (v3.1.0): Extracting archive
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi

Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0
Discovered Package: facade/ignition
Discovered Package: fideloper/proxy
Discovered Package: fruitcake/laravel-cors
Discovered Package: laravel/sail
Discovered Package: laravel/tinker
Discovered Package: laravel/ui
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
73 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
```

```
php artisan ui vue --auth
```

```
Vue scaffolding installed successfully.
Please run "npm install && npm run dev" to compile your fresh scaffolding.
Authentication scaffolding generated successfully.
```

```sh
php artisan migrate
```

---

## おかたづけ

```sh
kubectl delete -f laravel-mysql-ingress.yaml
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml

kubectl delete -f laravel-mysql-service-app.yaml
kubectl delete -f laravel-mysql-deploy-app-web.yaml
kubectl delete -f laravel-mysql-configmap.yaml
kubectl delete -f laravel-mysql-service-db.yaml
kubectl delete -f laravel-mysql-deploy-db.yaml
kubectl delete -f laravel-mysql-pvc.yaml
kubectl delete -f laravel-mysql-pv.yaml
```
