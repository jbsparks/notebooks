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

## Applying the changes on noname-sms

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

E.g. On my laptop dump the existing images
```bash
docker tag jsparkscraycom/hadrian-jupyter:latest sms.local:5000/cache/jupyterhub/hadrian-jupyter:latest
docker tag jupyterhub/k8s-hub:0.8.2 sms.local:5000/cache/jupyterhub/k8s-hub:0.8.2
docker tag jupyterhub/k8s-image-awaiter:0.8.2 sms.local:5000/cache/jupyterhub/k8s-image-awaiter:0.8.2
docker tag jupyterhub/k8s-network-tools:0.8.2 sms.local:5000/cache/jupyterhub/k8s-network-tools:0.8.2
docker tag jupyterhub/configurable-http-proxy:3.0.0 sms.local:5000/cache/jupyterhub/configurable-http-proxy:3.0.0
```

E.g. save the images to files
```bash
docker save sms.local:5000/cache/jupyterhub/hadrian-jupyter:latest | gzip > hadrian-jupyter.tgz
docker save sms.local:5000/cache/jupyterhub/k8s-hub:0.8.2 | gzip > k8s-hub_0.8.2.tgz
docker save sms.local:5000/cache/jupyterhub/k8s-image-awaiter:0.8.2 | gzip > k8s-image-awaiter_0.8.2.tgz
docker save sms.local:5000/cache/jupyterhub/k8s-network-tools:0.8.2 | gzip > k8s-network-tools_0.8.2.tgz
docker save sms.local:5000/cache/jupyterhub/configurable-http-proxy:3.0.0 | gzip > configurable-http-proxy_3.0.0.tgz
```

E.g. copy the artifacts to the remote sms node - in this case starlord-sms1

```
ssh root@starlord-sms1 mkdir jhub 
scp -r notebooks root@starlord-sms1:jhub
```

E.g. contents of the image directory. __Note:__ image sizes will change overtime.
```bash
cd jhub/notebooks/images 
ls -lh
total 2.7G
-rw-r--r-- 1 root root  19M Aug  6 06:22 configurable-http-proxy_3.0.0.tgz
-rw-r--r-- 1 root root 2.5G Aug  6 06:26 hadrian-jupyter.tgz
-rw-r--r-- 1 root root 218M Aug  6 06:27 k8s-hub_0.8.2.tgz
-rw-r--r-- 1 root root 1.6M Aug  6 06:27 k8s-image-awaiter_0.8.2.tgz
-rw-r--r-- 1 root root 2.3M Aug  6 06:27 k8s-network-tools_0.8.2.tgz
```
E.g. Load the images into the sms docker registry.
```bash
for f in `ls *.tgz`; do docker load < $f; done
 Loaded image: sms.local:5000/cache/jupyterhub/configurable-http-proxy:3.0.0
 Loaded image: sms.local:5000/cache/jupyterhub/k8s-hub:0.8.2
 Loaded image: sms.local:5000/cache/jupyterhub/k8s-image-awaiter:0.8.2
 Loaded image: sms.local:5000/cache/jupyterhub/k8s-network-tools:0.8.2
```

E.g. push images to the registry
```bash
docker push sms.local:5000/cache/jupyterhub/configurable-http-proxy:3.0.0
docker push sms.local:5000/cache/jupyterhub/k8s-hub:0.8.2
docker push sms.local:5000/cache/jupyterhub/k8s-image-awaiter:0.8.2
docker push sms.local:5000/cache/jupyterhub/k8s-network-tools:0.8.2
docker push sms.local:5000/cache/jupyterhub/hadrian-jupyter:latest
```

E.g. create the namespace
```bash
kubectl create namespace jhub
```
E.g. create the servies, assume we are in manifest directory....
```
kubectl create namespace jhub
kubectl apply --namespace jhub --recursive --filename ${PWD}/manifest
kubectl --namespace=jhub get pods
kubectl get service --namespace jhub
kubectl describe service proxy-public --namespace jhub
IP=$(ip a|grep em2|grep inet|awk '{print $2}'|cut -d"/" -f1)
PORT=$(kubectl get service --namespace jhub | grep "80:"|awk '{print $5}'|cut -d"/" -f1|cut -d":" -f2)
echo "http:${IP}:${PORT}"
```

Test's ...

```bash
kubectl --namespace=jhub get pods
NAME                      READY   STATUS    RESTARTS   AGE
hook-image-puller-g4cz7   1/1     Running   0          18m
hook-image-puller-h72r9   1/1     Running   0          18m
hook-image-puller-sqfch   1/1     Running   0          18m
hook-image-puller-xzw5m   1/1     Running   0          18m
hub-678b7d648-6nxrs       1/1     Running   1          18m
proxy-cd6dcdfb6-7j65s     1/1     Running   0          18m
```

```bash
kubectl get service --namespace jhub
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
hub            ClusterIP      10.23.72.229    <none>        8081/TCP                     19m
proxy-api      ClusterIP      10.23.134.123   <none>        8001/TCP                     19m
proxy-public   LoadBalancer   10.18.171.28    10.4.100.2    80:30590/TCP,443:31251/TCP   19m
```

__Note:__
If the IP for proxy-public is too long to fit into the window, you can find the longer version by calling:

```bash
kubectl describe service proxy-public --namespace jhub
Name:                     proxy-public
Namespace:                jhub
Labels:                   app=jupyterhub
                          chart=jupyterhub-0.8.2
                          component=proxy-public
                          heritage=Tiller
                          release=jub
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"jupyterhub","chart":"jupyterhub-0.8.2","component":"prox...
Selector:                 component=proxy,release=jub
Type:                     LoadBalancer
IP:                       10.18.171.28
LoadBalancer Ingress:     10.4.100.2
Port:                     http  80/TCP
TargetPort:               8000/TCP
NodePort:                 http  30590/TCP
Endpoints:                10.36.2.149:8000
Port:                     https  443/TCP
TargetPort:               443/TCP
NodePort:                 https  31251/TCP
Endpoints:                10.36.2.149:443
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason       Age   From                Message
  ----    ------       ----  ----                -------
  Normal  IPAllocated  20m   metallb-controller  Assigned IP "10.4.100.2"
  ```

To use JupyterHub, enter the external IP for the proxy-public service in to a browser. JupyterHub is running with a default dummy authenticator so entering any username and password combination will let you enter the hub.

## Session

On a __real__ Shasta system, we need to use ip addresses, cuz DNS isn't working and for github, turn off SSL verification.

```bash
export GIT_SSL_NO_VERIFY=true
git clone http://140.82.113.3/jbsparks/notebooks.git
```

# Teardown

```bash
kubectl delete --namespace jhub --recursive --filename ${PWD}/manifest
kubectl delete namespace jhub
```
