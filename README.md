# notebooks
Jupiter notebooks

## Convert helm/tiller to kubectl
__See__: https://blog.giantswarm.io/what-you-yaml-is-what-you-get for more details

```bash
$ mkdir charts manifests
$ helm fetch --untar --untardir ./charts jupyterhub/jupyterhub
$ cd charts
$ helm template \
    --name jub \
    --namespace jhub \
    --set proxy.secretToken="b326bae33a6793ab57c5ca3677c540c4d53a4be193c7551adcb0778018e05d97" \
    jupyterhub \
    --output-dir ~/manifests
$ cd ..
```

Applying the changes on noname-sms

```bash
# kubectl create namespace jhub
# kubectl apply --namespace jhub --recursive --filename ${PWD}/jupyterhub
configmap/hub-config created
deployment.apps/hub created
poddisruptionbudget.policy/hub created
persistentvolumeclaim/hub-db-dir created
serviceaccount/hub created
role.rbac.authorization.k8s.io/hub created
rolebinding.rbac.authorization.k8s.io/hub created
secret/hub-secret created
service/hub created
daemonset.apps/hook-image-puller created
job.batch/hook-image-awaiter created
serviceaccount/hook-image-awaiter created
role.rbac.authorization.k8s.io/hook-image-awaiter created
rolebinding.rbac.authorization.k8s.io/hook-image-awaiter created
deployment.apps/proxy created
poddisruptionbudget.policy/proxy created
service/proxy-api created
service/proxy-public created
poddisruptionbudget.policy/user-placeholder created
statefulset.apps/user-placeholder created
poddisruptionbudget.policy/user-scheduler created
```

__Note__: The opensource deployment model assumes pods can pull images from external locations. Not so for Shasta. We need to modify the image specification in the pod spec's to use the internal Shasta docker registery ... 

The mechanism is usually follows this recipe.

* docker pull myimage:latest
* docker tag myimage:latest sms.local:5000/cray/myimage:latest
* docker save sms.local:5000/cray/myimage:latest | gzip > shasta_myimage_latest.tar.gz

E.g. On my laptop
```bash
docker tag jsparkscraycom/hadrian-jupyter:latest sms.local:5000/cache/jupyterhub/hadrian-jupyter:latest
docker tag jupyterhub/k8s-hub:0.8.2 sms.local:5000/cache/jupyterhub/k8s-hub:0.8.2
docker tag jupyterhub/k8s-image-awaiter:0.8.2 sms.local:5000/cache/jupyterhub/k8s-image-awaiter:0.8.2
docker tag jupyterhub/k8s-network-tools:0.8.2 sms.local:5000/cache/jupyterhub/k8s-network-tools:0.8.2
docker tag jupyterhub/configurable-http-proxy:3.0.0 sms.local:5000/cache/jupyterhub/configurable-http-proxy:3.0.0
```

```bash
docker save sms.local:5000/cache/jupyterhub/hadrian-jupyter:latest | gzip > hadrian-jupyter.tgz
docker save sms.local:5000/cache/jupyterhub/k8s-hub:0.8.2 | gzip > k8s-hub_0.8.2.tgz
docker save sms.local:5000/cache/jupyterhub/k8s-image-awaiter:0.8.2 | gzip > k8s-image-awaiter_0.8.2.tgz
docker save sms.local:5000/cache/jupyterhub/k8s-network-tools:0.8.2 | gzip > k8s-network-tools_0.8.2.tgz
docker save sms.local:5000/cache/jupyterhub/configurable-http-proxy:3.0.0 | gzip > configurable-http-proxy_3.0.0.tgz
```

```bash
ls -lh
total 2.7G
-rw-r--r-- 1 root root  19M Aug  6 06:22 configurable-http-proxy_3.0.0.tgz
-rw-r--r-- 1 root root 2.5G Aug  6 06:26 hadrian-jupyter.tgz
-rw-r--r-- 1 root root 218M Aug  6 06:27 k8s-hub_0.8.2.tgz
-rw-r--r-- 1 root root 1.6M Aug  6 06:27 k8s-image-awaiter_0.8.2.tgz
-rw-r--r-- 1 root root 2.3M Aug  6 06:27 k8s-network-tools_0.8.2.tgz

for f in `ls *.tgz`; do docker load < $f; done
 Loaded image: sms.local:5000/cache/jupyterhub/configurable-http-proxy:3.0.0
 Loaded image: sms.local:5000/cache/jupyterhub/k8s-hub:0.8.2
 Loaded image: sms.local:5000/cache/jupyterhub/k8s-image-awaiter:0.8.2
 Loaded image: sms.local:5000/cache/jupyterhub/k8s-network-tools:0.8.2
 
docker push sms.local:5000/cache/jupyterhub/configurable-http-proxy:3.0.0
docker push sms.local:5000/cache/jupyterhub/k8s-hub:0.8.2
docker push sms.local:5000/cache/jupyterhub/k8s-image-awaiter:0.8.2
docker push sms.local:5000/cache/jupyterhub/k8s-network-tools:0.8.2
docker push sms.local:5000/cache/jupyterhub/hadrian-jupyter:latest
```

Move to the sms node

* docker load < shasta_myimage_latest.tar.gz


Test's ...

```bash
kubectl --namespace=jhub get pods
```

Teardown

```bash
kubectl delete --namespace jhub --recursive --filename ${PWD}/jupyterhub
```
