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
kubectl create namespace jhub
sms01-nmn:~/jsparks/manifests # kubectl apply --namespace jhub --recursive --filename ${PWD}/jupyterhub
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

Test's ...

```bash
kubectl --namespace=jhub get pods
```

Teardown

```bash
kubectl delete --namespace jhub --recursive --filename ${PWD}/jupyterhub
```
